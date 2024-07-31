## Method one: Singular archive file

We can use the following command to create a dump of your entire database, all collections included. This assumes your source database is (also) in a container, but the command should be roughly identical if you your source database is directly installed on a (virtual) machine, just leave out the Docker specific bits.

```bash
docker exec -i <container_name> /usr/bin/mongodump --uri "<your mongodb+srv link>" --archive > ~/Downloads/mongodb.dump
```

```bash
docker exec -i <container_name> /usr/bin/mongodump --username <username> --password <password> --authenticationDatabase admin --db <database_name> --archive > ~/Downloads/mongodb.dump
```

### Restoring

Now that we have the database dump files ready, let's go ahead and import (or, restore) them. Whether this is a fresh and new container or you're restoring a backup, the process is the same.

#### Ensure a database user exists

Before we continue, we should make sure we have a database user account ready with read and write access to the database you want to restore into. If you don't have one already, let's go ahead and create one now.

You can run the following in your mongodb (cli) client of choice to create a new user. A prompt will appear after running this command in which you can specify the user's password.

```mongodb
use admin

db.createUser({
    user: "username",
	pwd: passwordPrompt(),
	roles:[{role: "readWrite" , db:"<database_name>"}]})
```

#### Why "use admin"

One part that seems somewhat confusing and that threw me off previously is that by default when you try to connect to a MongoDB server it expects that `admin` is the database in which the user exists – even if said user has no access permissions for this database.

There are ways to change this behavior, but that is outside the scope of this guide. As you specify user roles that specify which database(s) the user has actual access to, even though the user might exist in the `admin` database, it doesn't actually have any permissions to do anything there, so it seems to not be that big of a deal.

Slightly confusing, but that seems to be the Mongo way. Anyway, let's continue.

```bash
docker exec -i <container_name> /usr/bin/mongorestore --uri "<your mongodb+srv link>" --archive < ~/Downloads/mongodb.dump
```

```bash
docker exec -i <container_name> /usr/bin/mongorestore --username <username> --password <password> --authenticationDatabase admin --nsInclude="<database_name>.*" --archive < ~/Downloads/mongodb.dump
```

## Method two: Individual JSON files

By default a `mongodump` export generates individual files, one for reach collection. If this is the method you'd like to use, we won't be able to rely on shell piping to get the files directly out of our database container. Instead we will use an intermediary step of storing the dumped database files in a directory within the container, which we can then copy out to our host using `docker cp`. If you have a host directory mounted as a volume to the container, you can use that and skip the `docker cp` step altogether, too.

Here's an example that dumps your entire database to the `/dump` directory within your container:

```bash
docker exec -i <container_name> /usr/bin/mongodump --uri "<your mongodb+srv link>" --out /dump
```

```bash
docker exec -i <container_name> /usr/bin/mongodump --username <username> --password <password> --authenticationDatabase admin --db <database_name> --out /dump
```

Now that we have the resulting files we can copy them out of the container and to the host machine. Here I am using the `~/Downloads` destination as an example, but it can be whatever you need it to be of course:

```bash
docker cp <container_name>:/dump ~/Downloads/dump
```

### Restoring

Now that we have the database dump files ready, let's go ahead and import (or, restore) them. Whether this is a fresh and new container or you're restoring a backup, the process is the same.

#### Ensure a database user exists

Before we continue, we should make sure we have a database user account ready with read and write access to the database you want to restore into. If you don't have one already, let's go ahead and create one now.

You can run the following in your mongodb (cli) client of choice to create a new user. A prompt will appear after running this command in which you can specify the user's password.

---

#### Copy files into the container

If you used this method to create a database dump and have multiple individual files, then we should first make the dumped database files available within the docker container by copying them into the container:

```bash
docker cp ~/Downloads/dump <container_name>:/dump
```

Alternatively you can use docker volumes to (temporarily) mount the directory where you have the dump files stored on your host, so you don't have to manually copy them over like this. Use whichever method you prefer, it'll work either way.

#### Restoring

Now that your database dump files are available within the container, we can run the following command to import everything:

```bash
docker exec -i <container_name> /usr/bin/mongorestore --uri "<your mongodb+srv link>" /dump/<database_name>
```

```bash
docker exec -i <container_name> /usr/bin/mongorestore --username <username> --password <password> --authenticationDatabase admin --nsInclude=<database_name>.\* /dump/<database_name>
```

#### Cleaning up afterwards

If you'd like to delete the dump files from within the container after importing, you can open a shell session within the docker container like so:

```bash
❯ docker exec -it <container_name> /bin/bash
```

Now you can delete these files by running something like `rm -rf /dump`.
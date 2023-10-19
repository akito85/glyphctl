Format Dump
```
docker exec -i <container_name> <shell_name> 
-c "PGPASSWORD=pg_password pg_dump --username pg_username database_name" > <path_on_your_machine>
```
Format Restore
```sh
docker exec -i <container_name> <shell_name> 
-c "PGPASSWORD=pg_password psql --username pg_username database_name" < <path_on_your_machine>
```
----
**Example**

Dump:
```
docker exec -i postgresql16 /bin/bash -c "PGPASSWORD=psql16 pg_dump --username postgres comcent -n api" > /home/akito/Repositories/cc/DATA/comcent-postgre.sql
```

Restore:
```
docker exec -i postgresql16 /bin/bash -c "PGPASSWORD=psql16 psql --username postgres comcent" < /home/akito/Repositories/cc/DATA/comcent-postgre.sql
```
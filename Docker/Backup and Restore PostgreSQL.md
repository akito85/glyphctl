Format Dump:
```bash
docker exec -i <container_name> <shell_name> 
-c "PGPASSWORD=pg_password pg_dump --username pg_username database_name" > <path_on_your_machine>
```

Format Restore:
```bash
docker exec -i <container_name> <shell_name> 
-c "PGPASSWORD=pg_password psql --username pg_username database_name" < <path_on_your_machine>
```

Format Dump and Restore at The Same Time:
```bash
docker exec -i pg_old_container_name /bin/bash -c "PGPASSWORD=pg_password pg_dump --username pg_username database_name" | docker exec -i pg_new_container_name /bin/bash -c "PGPASSWORD=pg_password psql --username pg_username database_name"
```

----
**Example**

Dump:
```bash
docker exec -i postgresql16 /bin/bash -c "PGPASSWORD=psql16 pg_dump --username postgres comcent -n api" > /home/akito/Repositories/cc/DATA/comcent-postgre.sql
```

Restore:
```bash
docker exec -i postgresql16 /bin/bash -c "PGPASSWORD=psql16 psql --username postgres comcent" < /home/akito/Repositories/cc/DATA/comcent-postgre.sql
```
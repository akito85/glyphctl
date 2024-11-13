
**1. Minio Client on Docker** 
```bash
docker run --name minio \
   -v {$HOME}/minio/data:/data \
   quay.io/minio/minio:latest server /data --console-address ":9001"
```

Or
```bash
docker run -p 9000:9000 -p 9001:9001 minio/minio server /data --console-address ":9001"
```

**2. Create an Alias for the S3-Compatible Service**

Base command
```
mc alias set ALIAS HOSTNAME ACCESS_KEY SECRET_KEY
```

Implemented
```bash
mc alias set lcminio http://2.2.0.1:9000 qGXsrXl8z8JTY8mX3qgo F6DavzkuGKRtK0Hr7rV1SDaGMorn64LjEt1KQkkE
```

**3. Test Connection**
```bash
mc admin info lcminio
```

Result
```bash
at 10:49:30 ❯ mc admin info lcminio
●  2.2.0.1:9000
   Uptime: 1 hour
   Version: 2024-10-02T17:50:41Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1

Pools:
   1st, Erasure sets: 1, Drives per erasure set: 1

82 KiB Used, 1 Bucket, 1 Object
1 drive online, 0 drives offline
```
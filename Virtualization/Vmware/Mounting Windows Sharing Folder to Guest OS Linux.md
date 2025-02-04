Guest OS
--
**Confirm that you already shared the folder**
```
vmware-hgfsclient
```
**Create directory or target directory to be mounted**
```
mkdir <target_directory>
e.g.:
- mkdir /mnt/hgfs/shared
- mkdir /home/akito/shared
- mkdir /temp/shared
```
**Mount the directory**
```
sudo vmhgfs-fuse .host:<host_directory> <guest_directory> <args>
sudo vmhgfs-fuse .host:/shared /mnt/hgfs/shared -o allow_other -o uid=1000
```
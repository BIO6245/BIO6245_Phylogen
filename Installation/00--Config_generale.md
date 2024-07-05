# Configuration générale du serveur

See this webpage for details on how to share data between users: https://docs.alliancecan.ca/wiki/Sharing_data

The problem is that, by default, group members do not have access to files produced by other group members. It is possible to change a folder so that it is accessible by everyone in my group with `chmod g+rwx fichier`, but when a user creates a file in this folder, the file is only accessible to them and not to other users. The user would then have to `chmod` each new file it produces to give access to other group members, which is far from ideal.

Instead, it is possible to create a folder that is accessible to everyone in the group, and inside which files are open to everyone by default (r/w/x):
```bash
## fix permissions and default permissions on /data
sudo chown -R root:etudiants /data
sudo chmod -R g+rwx /data
sudo setfacl -R -d -m g::rwx /data
getfacl /data

## fix permissions and default permissions on /scratch
## fix permissions and default permissions on /scratch
sudo chown -R root:etudiants /scratch
sudo chmod -R g+rwx /scratch
sudo setfacl -R -d -m g::rwx /scratch
getfacl /scratch

```


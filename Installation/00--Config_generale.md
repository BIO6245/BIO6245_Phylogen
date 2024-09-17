sudo res# Configuration générale du serveur

## Permissions pour dossiers partagés

See this webpage for details on how to share data between users: https://docs.alliancecan.ca/wiki/Sharing_data

The problem is that, by default, group members do not have access to files produced by other group members. It is possible to change a folder so that it is accessible by everyone in my group with `chmod g+rwx fichier`, but when a user creates a file in this folder, the file is only accessible to them and not to other users. The user would then have to `chmod` each new file it produces to give access to other group members, which is far from ideal.

Instead, it is possible to create a folder that is accessible to everyone in the group, and inside which files are open to everyone by default (r/w/x):
```bash
## fix permissions and default permissions on /data
sudo chown -R root:etudiants /data
sudo chmod -R g+rws /data
sudo setfacl -R -d -m g:etudiants:rwx /data
getfacl /data

## fix permissions and default permissions on /scratch
sudo chown -R root:etudiants /scratch
sudo chmod -R g+rwx /scratch
sudo setfacl -R -d -m g:etudiants:rwx /scratch
getfacl /scratch

## fix permissions and default permissions on /data/hybseqRefs
mkdir -p /data/hybseqRefs
sudo chown -R root:etudiants /data/hybseqRefs
sudo chmod -R g+rx /data/hybseqRefs
sudo setfacl -R -d -m g:etudiants:rx /data/hybseqRefs
getfacl /data/hybseqRefs

```

## Installation de paquets via la logithèque Ubuntu

Voici une liste des paquets installés
```bash
## Java
sudo apt-get install default-jre default-jdk

## CMake to help compile stuff
sudo apt-get install cmake

## Boost libraries
sudo apt-get install libboost-dev libboost-system-dev libboost-program-options-dev libboost-iostreams-dev \
  libboost-filesystem-dev

## Julia
sudo apt-get install snapd
sudo snap install julia --classic

## R
sudo apt-get install r-base-core

```


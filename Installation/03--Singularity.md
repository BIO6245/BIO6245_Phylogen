# Installer Singularity

## Mettre à jour les paquets et installer les dépendances

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install -y \
    autoconf \
    automake \
    build-essential \
    cryptsetup \
    fuse \
    fuse2fs \
    git \
    libfuse-dev \
    libglib2.0-dev \
    libgpgme11-dev \
    libseccomp-dev \
    libssl-dev \
    libtool \
    pkg-config \
    runc \
    squashfs-tools \
    squashfs-tools-ng \
    uidmap \
    uuid-dev \
    wget \
    zlib1g-dev

```

## Installer Go

Singularity nécessite Go (version 1.18 ou plus récente). L'installer avec ces commandes:  
```bash
VERSION=1.23.2
OS=linux
ARCH=amd64

wget https://dl.google.com/go/go$VERSION.$OS-$ARCH.tar.gz
sudo tar -C /usr/local -xzvf go$VERSION.$OS-$ARCH.tar.gz
rm go$VERSION.$OS-$ARCH.tar.gz

```

Ajouter cette ligne au `.bashrc` de tous les utilisateurs pour rendre Go disponible à tous.  
```bash
BASHRC_PATHS=$(sudo find /home -name ".bashrc")
for FILE in $BASHRC_PATHS
  do
    echo "=== $FILE avant le changement ==="
	sudo tail $FILE
    echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee -a $FILE > /dev/null
	echo "--- $FILE après le changement ---"
	sudo tail $FILE
  done

echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee -a /root/.bashrc > /dev/null

```

Télécharger le paquet "deb" de Go qui est prérequis pour Singularity 3.0.0:  
```bash
go install github.com/golang/dep/cmd/dep@latest

```

## Télécharger et installer Singularity

```bash
VERSION=4.2.0
cd /opt
sudo wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz
sudo tar -xzf singularity-ce-${VERSION}.tar.gz
cd singularity-ce-${VERSION}
sudo env PATH=$PATH:/usr/local/go/bin ./mconfig
sudo env PATH=$PATH:/usr/local/go/bin make -C builddir
sudo env PATH=$PATH:/usr/local/go/bin make -C builddir install

## tester l'installation
SRC_SING=/opt/singularity-ce-4.2.0/builddir
$SRC_SING/singularity --version

```

---

## Installer l'image Singularity de OMM_MACSE

```bash
cd /opt
sudo singularity pull library://vranwez/default/omm_macse:v12.01

## test installation
SRC_MACSE=/opt/omm_macse_v12.01.sif 
SRC_SING=/opt/singularity-ce-4.2.0/builddir
$SRC_SING/singularity exec $SRC_MACSE

```


# Installation de programmes d'analyse phylogénétique

## Parcimonie

PAUP\* v.169 (ne fonctionne pas car demande libgfortran.so.4 qui n'est plus disponible sous Ubuntu 
24.04):  
```bash
cd /opt
sudo wget http://phylosolutions.com/paup-test/paup4a169_ubuntu64.gz
sudo gunzip paup4a169_ubuntu64.gz
sudo chmod +x paup4a169_ubuntu64

## tester paup
/opt/paup4a169_ubuntu64 -h

```

PAUP\* v.168 (semble fonctionner sous Ubuntu 24.04):  ```bash
cd /opt
sudo wget http://phylosolutions.com/paup-test/paup4a168_centos64.gz
sudo gunzip paup4a168_centos64.gz
sudo chmod +x paup4a168_centos64

## tester paup
/opt/paup4a168_centos64 -h

```


## Maximum de vraisemblance

FastTree:
```bash
sudo apt-get update
sudo apt-get install fasttree

```

RAxML:
```bash
cd /opt
sudo git clone https://github.com/stamatak/standard-RAxML.git
sudo mv standard-RAxML RAxML-8.2.12
cd RAxML-8.2.12
sudo make -f Makefile.SSE3.PTHREADS.gcc

## test installation
SRC_RAXML=/opt/RAxML-8.2.12
$SRC_RAXML/raxmlHPC-PTHREADS-SSE3 -h

```

IQ-Tree:  
```bash
cd /opt
sudo wget https://github.com/iqtree/iqtree2/releases/download/v2.3.6/iqtree-2.3.6-Linux-intel.tar.gz
sudo tar -xzvf iqtree-2.3.6-Linux-intel.tar.gz
sudo rm iqtree-2.3.6-Linux-intel.tar.gz

## test installation
SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
$SRC_IQ/iqtree2 --help

```

## Analyse Bayesienne

Installer Beagle:  
```bash
sudo apt-get update
sudo apt-get install \
  cmake \
  build-essential \
  autoconf \
  automake \
  libtool \
  git \
  pkg-config \
  openjdk-11-jdk

cd /opt
sudo git clone --depth=1 https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib
sudo mkdir build
cd build
sudo cmake -DCMAKE_INSTALL_PREFIX:PATH=/opt/beagle-lib ..
sudo make install

```

Installer MrBayes avec Beagle. Note importante: j'ai tenté initialement d'installer MrBayes 
avec mpich, et ça ne fonctionnait pas (mpirun initialisait plusieurs instances à 1 processeur 
chacune, au lieu d'une seule instance avec plusieurs processeurs). Ensuite, j'ai tenté avec 
OpenMPI et ç'a fonctionné tout de suite.  
```bash
## installer prérequis
sudo apt install \
  automake \
  autoconf \
  pkg-config \
  autoconf-archive \
  openmpi-bin \
  libopenmpi-dev

## installer version parallèle
cd /opt
sudo git clone --depth=1 https://github.com/NBISweden/MrBayes.git
sudo mv MrBayes MrBayes-mpi
cd MrBayes-mpi
sudo ./configure --with-mpi --with-beagle=/opt/beagle-lib
sudo make && sudo make install
sudo mv /opt/MrBayes-mpi/src/mb /opt/MrBayes-mpi/src/mb-mpi

## installer version non-parallèle
cd /opt
sudo git clone --depth=1 https://github.com/NBISweden/MrBayes.git
cd MrBayes
sudo ./configure --without-mpi --with-beagle=/opt/beagle-lib
sudo make && sudo make install

## tester version non-parallèle
/opt/MrBayes/src/mb -v

## tester la version parallèle
mpirun.openmpi -np 2 /opt/MrBayes-mpi/src/mb-mpi

```

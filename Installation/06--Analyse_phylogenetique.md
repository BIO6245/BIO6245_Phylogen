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

## Détermination des partitions et modèles

PartitionFinder:
```bash
cd /opt
sudo wget https://github.com/brettc/partitionfinder/archive/refs/tags/v2.1.1.tar.gz
sudo tar -xzvf v2.1.1.tar.gz
sudo rm v2.1.1.tar.gz
cd /opt/partitionfinder-2.1.1
sudo chmod +x PartitionFinder.py

## test
## !!! Make sure to create a partitionFinder conda environment (see Conda installation page)
conda activate partitionFinder
mkdir -p /scratch/elbourret/test/partitionFinder
cd /scratch/elbourret/test/partitionFinder
cp -r /opt/partitionfinder-2.1.1/examples/nucleotide .
/opt/partitionfinder-2.1.1/PartitionFinder.py --processes=1 nucleotide/partition_finder.cfg

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

Installer BEAST 2:  
```bash
cd /opt
sudo wget https://github.com/CompEvol/beast2/releases/download/v2.7.7/BEAST.v2.7.7.Linux.x86.tgz
sudo tar -xzvf BEAST.v2.7.7.Linux.x86.tgz
sudo rm BEAST.v2.7.7.Linux.x86.tgz

## test Installation
/opt/beast/bin/beast -help

```

Installer des paquets de BEAST 2. La méthode standard avec le `packagemanager` ne fonctionne pas, 
car les TI ont bloqué l'accès Internet aux programmes sur les serveurs de l'UdeM. Il faut donc 
installer les paquets à la main:  
```bash
VERSION=2.7

## créer dossier partagé pour les paquets BEAST 2
sudo mkdir -p /usr/local/share/beast/$VERSION

## télécharger les paquets
cd ~
wget https://github.com/BEAST2-Dev/BEASTLabs/releases/download/v2.0.0/BEASTlabs.package.v2.0.2.zip
wget https://github.com/rbouckaert/starbeast3/releases/download/v1.1.9/starbeast3.addon.v1.1.9.zip
wget https://github.com/jordandouglas/ORC/releases/download/v1.2.0/ORC.addon.v1.2.0.zip
wget https://github.com/rbouckaert/AlmostDistributions/releases/download/v0.2.0/AlmostDistributions.addon.v0.2.0.zip
wget https://github.com/nicfel/CoupledMCMC/releases/download/v1.2.0/CoupledMCMC.v1.2.2.zip

## extraire les paquets dans le dossier partagé
sudo unzip BEASTlabs.package.v2.0.2.zip -d /usr/local/share/beast/$VERSION/BEASTlabs
sudo unzip starbeast3.addon.v1.1.9.zip -d /usr/local/share/beast/$VERSION/starbeast3
sudo unzip ORC.addon.v1.2.0.zip -d /usr/local/share/beast/$VERSION/ORC
sudo unzip AlmostDistributions.addon.v0.2.0.zip -d /usr/local/share/beast/$VERSION/AlmostDistributions
sudo unzip CoupledMCMC.v1.2.2.zip -d /usr/local/share/beast/$VERSION/CoupledMCMC

## supprimer beauti.properties de tous utilisateurs pour réinitialiser chemin vers paquets
for i in $(ls -d /home/*)
  do
    sudo rm $i/.beast/2.7/beauti.properties
  done

```

## Arbres d'espèces

Installer ASTER:  
```bash
cd /opt
sudo wget https://github.com/chaoszhang/ASTER/archive/refs/heads/Linux.zip
sudo mv Linux.zip ASTER.zip
sudo unzip ASTER.zip
sudo mv ASTER-Linux ASTER
cd ASTER
sudo make
cd /opt
sudo rm ASTER.zip

## tester l'Installation
/opt/ASTER/bin/wastral -h


```


## Réconciliation de famille de gènes sur arbre d'espèces

AleRax:  
```bash
cd /opt
sudo git clone --recursive https://github.com/BenoitMorel/AleRax
cd AleRax
sudo ./install.sh

## test AleRax
/opt/AleRax/build/bin/alerax -h

```


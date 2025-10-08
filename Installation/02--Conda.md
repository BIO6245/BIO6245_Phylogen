# Installer conda, ses environnements et programmes

## Installer conda

Installer Miniconda:
```bash
cd /opt
sudo wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

sudo chmod +x ./Miniconda3-latest-Linux-x86_64.sh
sudo ./Miniconda3-latest-Linux-x86_64.sh -b -p /opt/miniconda

/opt/miniconda/bin/conda init bash
/opt/miniconda/bin/conda config
/opt/miniconda/bin/conda update -y conda

sudo rm ./Miniconda3-latest-Linux-x86_64.sh

```

S'assurer que conda s'initialise au démarrage nécessite un fichier `~/.profile` qui sourcera 
`~/.bashrc` à démarrer. Pour une raison quelconque, il n'était pas initialement présent sur le serveur. 
Le créer comme ceci (en tant que root) :  
```bash
## for each user
for u in /home/*/;
  do
  
    ## get username from home path
    CURRENT_USER=$(basename $u)
	echo $CURRENT_USER
	## create a .profile file
    echo 'if [ -n "$BASH_VERSION" ]; then
      # include .bashrc if it exists
      if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
      fi
    fi' > $u/.profile
	
	## copy the .bashrc file from elbourret
	cp -f /home/elbourret/.bashrc $u/.bashrc
	
	## change ownership of the newly created files
    chown $CURRENT_USER: $u/.profile
	chown $CURRENT_USER: $u/.bashrc
	
  done
  
```

Ensuite, activer les canaux de base pour l'installation de paquets python:  
```bash
source $HOME/.bashrc
conda activate base
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge

```

## Créer des environnements conda

Creating environments and installing software inside those environments:  
```bash
## list all environments
conda info --envs

## newick_utils
sudo /opt/miniconda/bin/conda create -y -n newick_utils -c bioconda newick_utils -c bioconda catfasta2phyml

## hybpiper
sudo /opt/miniconda/bin/conda create -y -n hybpiper -c bioconda -c conda-forge -c chrisjackson-pellicle hybpiper

## paragone (for paralog removal following hybpiper assembly)
sudo /opt/miniconda/bin/conda create -y -n paragone -c bioconda -c conda-forge paragone python=3.11

## biopython
sudo /opt/miniconda/bin/conda create -y -n bio -c conda-forge biopython

## ipyrad
sudo /opt/miniconda/bin/conda create -y -n ipyrad -c conda-forge -c bioconda ipyrad
sudo /opt/miniconda/bin/conda install -n ipyrad -c bioconda -c ipyrad structure clumpp
sudo /opt/miniconda/bin/conda install -n ipyrad -c eaton-lab toyplot

## partitionFinder
sudo /opt/miniconda/bin/conda create -y -n partitionFinder -c conda-forge python=2.7 numpy pandas pytables pyparsing scipy scikit-learn

## dendropy
sudo /opt/miniconda/bin/conda create -y -n dendropy -c conda-forge -c bioconda biopython numpy dendropy

## RagTag
sudo /opt/miniconda/bin/conda create -y -n ragtag -c bioconda ragtag

```

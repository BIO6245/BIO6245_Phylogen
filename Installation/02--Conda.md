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
Le créer comme ceci :  
```bash
echo 'if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi' >> /home/$USER/.profile

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
## newick_utils
sudo /opt/miniconda/bin/conda create -y -n newick_utils -c bioconda newick_utils -c bioconda catfasta2phyml

## hybpiper
sudo /opt/miniconda/bin/conda create -y -n hybpiper -c bioconda -c conda-forge -c chrisjackson-pellicle \
  hybpiper

## biopython
sudo /opt/miniconda/bin/conda create -y -n bio -c conda-forge biopython

```

!!! code below is not run AND NEEDS TO BE ADJUSTED TO THIS SERVER !!!

```bash
## getOrganelle
$SRC/conda create -n getOrganelle -c bioconda getorganelle
conda activate getOrganelle
get_organelle_config.py --config-dir $WD/getOrganelle --add embplant_pt,embplant_mt,embplant_nr

## RagTag
$SRC/conda create -n ragtag -c bioconda ragtag

## ipyrad
conda create -y -n ipyrad -c conda-forge ipyrad

## list all environments
$SRC/conda info --envs

```



## Problems to solve

I can't seem to install NovoWrap correctly in a conda environment. Here is what I tried up to now:

## NovoWrap
## Following issue #13 on the github, I found out that I need to install using pip, but still not working
$SRC/conda create -n novowrap python=3.8 pip
conda activate novowrap
$SRC/../envs/novowrap/bin/pip install -U --force-reinstall novowrap==1.30

$SRC/conda create -n novowrap -c wpwupingwp biopython=1.77 novowrap
To remove:
conda install 'wheel>=0.32.3' 'biopython>=1.77' 'coloredlogs>=10.0' 'matplotlib>=3.2.0' 'numpy>=1.19.1' 'pip>=18.0' 'graphviz>=0.13'

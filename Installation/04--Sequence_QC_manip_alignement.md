# Installation de programmes de manipulation, contrôle qualité et alignement de séquences

## Manipulation de séquences

Installer SRA-toolkit, qui sert à télécharger des données depuis le serveur NSBI SRA où sont déposés 
les données Illumina de presque toutes les études phylogénomiques:  
```bash
cd /opt
sudo wget --output-document sratoolkit.tar.gz \
  https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
sudo tar -vxzf sratoolkit.tar.gz
sudo rm sratoolkit.tar.gz

```

Ensuite, il faut ajuster la configuration de SRA_toolkit en naviguant vers le dossier 
`cd /opt/sratoolkit*`, puis en roulant `bin/vdb-config -i`. Choisir "Prefer SRA Lite files" pour 
assurer des téléchargement plus rapides, au coût de données de qualité des lectures un peu moins précises. 
**Très important: désactiver "Local file-caching" dans l'onglet Cache.**


seqtk:
```bash
sudo apt-get update
sudo apt-get install seqtk

```

samtools:  
```bash
sudo apt-get update
sudo apt-get install samtools

```

## Contrôle qualité

FastQC:
```bash
cd /opt
sudo wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.12.1.zip
sudo unzip fastqc*.zip
sudo rm fastqc*.zip
sudo chmod +x FastQC/fastqc

```

Trimmomatic:
```bash
cd /opt
sudo wget https://github.com/usadellab/Trimmomatic/files/5854859/Trimmomatic-0.39.zip
sudo unzip Trimmomatic-*.zip
sudo rm Trimmomatic-0.39.zip

```

HTStream:  
```bash
cd /opt
sudo git clone https://github.com/s4hts/HTStream.git
cd HTStream
sudo mkdir build
cd build
sudo cmake ..
sudo make

```


catfasta2phyml:
```bash
cd /opt
git clone https://github.com/nylander/catfasta2phyml.git
cd catfasta2phyml
chmod +x catfasta2phyml.pl

```

## Alignement

mafft:
```bash
sudo apt-get update
sudo apt-get install mafft

```

bwa:  
```bash
sudo apt-get update
sudo apt-get install bwa

```

gatk:  
```bash
cd /opt
sudo wget https://github.com/broadinstitute/gatk/releases/download/4.6.0.0/gatk-4.6.0.0.zip
sudo unzip gatk*.zip
sudo rm gatk*.zip

```

picard:  
```bash
cd /opt
sudo wget https://github.com/broadinstitute/picard/releases/download/3.2.0/picard.jar

```

bcftools:
```bash
sudo apt-get update
sudo apt-get install bcftools

```

tabix:  
```bash
sudo apt-get update
sudo apt-get install tabix

```

!!! Code ci-dessous pas encore terminé !!!




OMM_Macse for alignment of genes and pseudogenes at the nucleotide level based on codon similarity:
```bash
WD=/project/def-bourret/shared/progs

cd $WD

module load StdEnv/2023
module load apptainer/1.2.4

apptainer remote add --no-login SylabsCloud cloud.sycloud.io
apptainer remote use SylabsCloud
apptainer pull --arch amd64 library://vranwez/default/omm_macse:v12.01

```


## Filtrage des alignements


TAPER:  
```bash
cd /opt
sudo wget https://github.com/chaoszhang/TAPER/archive/refs/tags/v1.0.0.tar.gz
sudo tar -vxzf v1.0.0.tar.gz
sudo rm v1.0.0.tar.gz

## test installation
julia /opt/TAPER-1.0.0/correction_multi.jl -h

```

TrimAL:  
```
cd /opt
sudo wget https://github.com/inab/trimal/releases/download/v1.5.0/trimAl_Linux_x86-64.zip
sudo unzip trimAl_Linux_x86-64.zip
sudo mv trimAl_Linux_x86-64 trimAl_1.5.0
sudo rm ../trimAl_Linux_x86-64.zip

## test installation
/opt/trimAl_1.5.0/trimal -h

```
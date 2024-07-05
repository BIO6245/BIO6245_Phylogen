# Installation de programmes de manipulation, QC et alignement de séquences

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




!!! code below is not run AND NEEDS TO BE ADJUSTED TO THIS SERVER !!!





Navigate to the bin folder of the SRA-toolkit, and run `bin/vdb-config -i`. Set "Prefer SRA Lite files" to on: this will result in faster downloading and processing of files, at the cost of vary slightly innacurate read qualities. **VERY IMPORTANT: Disable local file caching!!!**

FastQC:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip
unzip fastqc_v0.11.9.zip
rm fastqc_v0.11.9.zip
chmod +x FastQC/fastqc

```

Trimmomatic:
```bash
WD=/project/def-bourret/shared/progs
mkdir -p $WD
cd $WD
wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.39.zip
unzip Trimmomatic-0.39.zip
rm Trimmomatic-0.39.zip
chmod +x FastQC/fastqc

```

catfasta2phyml:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
git clone https://github.com/nylander/catfasta2phyml.git
chmod +x catfasta2phyml.pl

```

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

seqtk:
```bash
WD=/project/def-bourret/shared/progs
cd $WD
git clone https://github.com/lh3/seqtk.git;
cd seqtk
make

```
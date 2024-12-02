# Télécharger des données publiques RADseq

## Trouver des données sur le SRA

Le [Short Read Archive (SRA)](https://www.ncbi.nlm.nih.gov/sra) est le plus important dépôt de 
séquences Illumina. Il est utilisé pour archiver les données brutes produites dans presques toutes 
les études phylogénomiques, incluant celles que nous allons utiliser pour tester les méthodes 
apprises dans ce cours. Voici un petit guide pour obtenir les données de 
[Marinček et al. (2024)]( https://doi.org/10.1002/ajb2.16361) depuis le SRA:  

1. Feuilleter Marinček et al. (2024) en cherchant un numéro d'étude SRA. À la fin de l'article, 
on peut trouver un tableau contenant l'information détaillée de l'échantillonnage "Appendix 1". 
Télécharger ce tableau et trouver le "numéro de projet" SRA.  
2. Naviguer vers [https://www.ncbi.nlm.nih.gov/sra/](https://www.ncbi.nlm.nih.gov/sra/)  
3. Rechercher le terme: `PRJNA433286[study]`, qui est le numéro d'étude SRA  
4. Cliquer sur *Send to -> File -> Accession list*   
5. Cliquer sur *Send to -> File -> RunInfo*  

Il faut ensuite transférer les fichiers `SraRunInfo.csv` et `SraAccList.csv` sur le serveur. Pour 
ce faire, vous pouvez utiliser plusieurs approches. L'approche que je préconise est d'envoyer les 
fichiers avec la commande `rsync`, qui doit être **executée à partir de votre machine locale,**
**non pas du serveur**. En d'autres mots, il faut ouvrir un terminal Unix à partir de votre 
ordinateur, ne pas vous connecter au serveur. À partir de votre ordinateur, tapez les lignes 
ci-dessous, en modifiant de façon appropriée les variables qui indiquent le chemin vers les 
fichiers enregistrés sur votre ordinateur et le chemin d'arrivée sur le serveur de calcul:  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
CLUSTER_USERNAME=testetudiant
LOCAL=/mnt/c/Users/Etienne/Downloads
REMOTE=/home/$CLUSTER_USERNAME


rsync --progress $LOCAL/SraAccList.csv $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/
rsync --progress $LOCAL/SraRunInfo.csv $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/

```

Notez que si vous êtes sur Windows, il faut remplacer les "backslash" (\\) dans votre chemin avec 
des "forward slash" (\/). De plus, le chemin "C:/" doit être remplacé par "/mnt/c/". Finalement, 
notez que les espaces dans les noms des dossiers et fichiers causent problème sur tous les 
terminal Linux/Mac. Il y a deux solutions pour les espaces: soit ne jamais utiliser d'espaces dans 
vos noms de fichier, ou soit entourer les espaces par des guillemets simples ('), ou précéder 
chaque espace par un "backslash" (\\) lorsque vous spécifiez un chemin vers un dossier ou fichier. 
Par exemple:  
- Le chemin Windows: `C:\Users\Moi\Documents\BIO 6245\ficher de travail.txt`  
- Doit être remplacé par: `/mnt/c/Users/Moi/Documents/BIO' '6245/fichier' 'de' 'travail.txt`  
- Ou bien par: `/mnt/c/Users/Moi/Documents/BIO\ 6245/fichier\ de\ travail.txt`

---

## Télécharger des données depuis le SRA

Le code ci-dessous va chercher les données Illumina pour 11 échantillons utilisés dans Marinček et 
al. (2024), incluant des diploïdes et quelques polyploïdes d'origine potentiellement hybride. Les 
commandes ci-dessous doivent être roulées sur le serveur de calcul, cette fois. Connectez-vous 
tout d'abord au serveur avant d'exécuter ces commandes:   
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_SRA=/opt/sratoolkit.3.1.1-ubuntu64/bin
WD=/data/$USER/RADseqTest
SCRATCH=/scratch/$USER
ACCLIST_PATH=/home/$USER
EMAIL=votre.courriel@umontreal.ca
TIME=6:00:00


mkdir -p $WD/reads
mkdir -p $SCRATCH
cd $WD/reads

cp $ACCLIST_PATH/SraRunInfo.csv .
cp $ACCLIST_PATH/SraAccList.csv .

## Sélectionner seulement 1 échantillon par espèce pour gérer moins de données à la fois:
## Si vous vouliez télécharger les données de tous les échantillons dans SraAccList.csv,
## alors il ne faut pas rouler les lignes de code ci-dessous (jusqu'au prochain commentaire)
SELECTED_ACCESSIONS=("SRR7448126" "SRR7448124" "SRR26678801" "SRR26678887" "SRR26678865" "SRR26678836" "SRR26678788" "SRR26678782" "SRR26678749" "SRR26678748" "SRR26678863")
rm SraAccList.csv
touch SraAccList.csv
for i in "${SELECTED_ACCESSIONS[@]}"
  do
     echo "Selecting accession $i"
     Acc_i=$(grep "$i" SraRunInfo.csv | 
       head -1 | 
       grep -o '^SRR[0-9]*' | 
       head -1)
     echo "$Acc_i"
     echo $Acc_i >> SraAccList.csv
  done




## Créer une batch file pour télécharger les données depuis SRA
echo '#!/bin/bash' > download_sra.sbatch
echo "#SBATCH --job-name=download_sra
#SBATCH --output=download_sra-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=8G
#SBATCH --time=$TIME

## select one accession to download
CURRENT_ACC=\$(head -n \$SLURM_ARRAY_TASK_ID SraAccList.csv | tail -1)

## prepare the download
$SRC_SRA/prefetch --verify no \$CURRENT_ACC

## download the data
$SRC_SRA/fasterq-dump -t $SCRATCH --split-files \$CURRENT_ACC

## compress the file to save space
gzip \${CURRENT_ACC}*.fastq

wait

## remove temporary folders for this accession
rm -r \$CURRENT_ACC/

## get the species name for the current accession number
SP=\$(grep \$CURRENT_ACC SraRunInfo.csv | cut -d',' -f29 | sed 's/ /_/g')

## change the name of the read files to include the species name
mv \${CURRENT_ACC}.fastq.gz \${SP}_\${CURRENT_ACC}_1.fastq.gz
mv \${CURRENT_ACC}_1.fastq.gz \${SP}_\${CURRENT_ACC}_1.fastq.gz
mv \${CURRENT_ACC}_2.fastq.gz \${SP}_\${CURRENT_ACC}_2.fastq.gz
" >> download_sra.sbatch

## Déterminer le nombre d'échantillons à analyser
NFILES=$(wc -l < SraAccList.csv)

## envoyer la tâche de téléchargement à SLURM
sbatch --mail-user=$EMAIL  --array=1-$NFILES download_sra.sbatch

```

Maintenant, il faut être patient car le serveur prendra plusieurs minutes (voir heures) pour 
télécharger les données. Le téléchargement se fait "en background" grâce à la commande 
`nohup [...] &`. Le progrès est inscrit dans un fichier ".log", nommé ici `dataFetch.log`. Il est 
possible de suivre le progrès en temps réel en tapant `tail -f dataFetch.log` dans le dossier où 
les données sont téléchargées. Pour quitter le suivi avec tail -f, il faut taper Ctrl+C.

Lorsque le téléchargement est terminé, vérifier combien de fichiers et quelle est leur taille à 
l'aide de la commande `ls -sh`.

- **Question:** Pourquoi y a-t-il plus de fichiers téléchargés que de nombre d'échantillons?

- **Questions:** Quels échantillons ont une meilleure couverture de séquençage? Quels ont une 
moins bonne couverture? Quelles sont les conséquences attendues d'une moins bonne couverture?

  Regarder aussi le contenu de quelques fichiers à l'aide de la commande `zcat *.fastq.gz | more`.

- **Question:** Que signifient chacune des 4 lignes associées à chaque lecture Illumina?

# Télécharger des données publiques HybSeq

## Trouver des données sur le SRA

Le [Short Read Archive (SRA)](https://www.ncbi.nlm.nih.gov/sra) est le plus important dépôt de séquences 
Illumina. Il est utilisé pour archiver les données brutes produites dans presques toutes les études 
phylogénomiques, incluant celles que nous allons utiliser pour tester les méthodes apprises dans ce cours. 
Voici un petit guide pour obtenir les données de 
[Tiley et al. (2024)](https://doi.org/10.1093/sysbio/syae024) depuis le SRA:  

1. Feuilleter Tiley et al. (2024) en cherchant un numéro d'étude SRA. À la fin de l'article, ils mentionnent 
que leur numéro d'étude SRA est dans le tableau S1 du matériel supplémentaire.  
2. Naviguer vers [https://www.ncbi.nlm.nih.gov/sra/](https://www.ncbi.nlm.nih.gov/sra/)  
3. Rechercher le terme: `SRP316248[study]`, qui est le numéro d'étude SRA  
4. Cliquer sur *Send to -> File -> Accession list*   
5. Cliquer sur *Send to -> File -> RunInfo*  

Il faut ensuite transférer les fichiers `SraRunInfo.csv` et `SraAccList.csv` sur le serveur. Pour ce faire, 
vous pouvez utiliser plusieurs approches. L'approche que je préconise est d'envoyer les fichiers avec la 
commande `rsync`, qui doit être **executée à partir de votre machine locale, non pas du serveur**. 
En d'autres mots, il faut ouvrir un terminal Unix à partir de votre ordinateur, ne pas vous connecter au 
serveur. À partir de votre ordinateur, tapez les lignes ci-dessous, en modifiant de façon appropriée les 
variables qui indiquent le chemin vers les fichiers enregistrés sur votre ordinateur et le chemin d'arrivée 
sur le serveur de calcul:  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
USERNAME=elbourret
LOCAL=/mnt/c/Users/p0948315/Downloads
REMOTE=/home/$USERNAME


rsync --progress $LOCAL/SraAccList.csv $USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/
rsync --progress $LOCAL/SraRunInfo.csv $USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/

```

## Télécharger des données depuis le SRA

Le code ci-dessous va chercher les données Illumina pour 7 échantillons utilisés dans Tiley et al. (2024), 
incluant deux allotétraploïdes et cinq diploïdes. Les commandes ci-dessous doivent être roulées sur le 
serveur de calcul, cette fois:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_SRA=/opt/sratoolkit.3.1.1-ubuntu64/bin
WD=/data/$USER/HybSeqTest
SCRATCH=/scratch/$USER
ACCLIST_PATH=/home/$USER


mkdir -p $WD/reads
mkdir -p $SCRATCH
cd $WD/reads

cp $ACCLIST_PATH/SraRunInfo.csv .
cp $ACCLIST_PATH/SraAccList.csv .

## Sélectionner seulement 1 échantillon par espèce pour gérer moins de données à la fois:
## Si vous vouliez télécharger les données de tous les échantillons dans SraAccList.csv,
## alors il ne faut pas rouler les lignes de code ci-dessous (jusqu'au prochain commentaire)
SELECTED_ACCESSIONS=("SRR14320989" "SRR14320984" "SRR14321005" "SRR14321002" \
  "SRR14320992" "SRR14320997" "SRR14320998")
rm SraAccList.csv
touch SraAccList.csv
for i in "${SELECTED_ACCESSIONS[@]}"
  do
     echo "Getting reads for $i"
     Acc_i=$(grep "$i" SraRunInfo.csv | 
       head -1 | 
       grep -o '^SRR[0-9]*' | 
       head -1)
     echo "$Acc_i"
     echo $Acc_i >> SraAccList.csv
  done





## Shell script pour télécharger les données depuis SRA
echo "$SRC_SRA/prefetch --verify no --option-file SraAccList.csv
$SRC_SRA/fasterq-dump -t $SCRATCH --split-files *RR*
gzip *.fastq
rm -r *RR*/
for i in *_1.fastq.gz
do
  NAME=\$(basename \$i _1.fastq.gz)
  SP=\$(grep "\$NAME" SraRunInfo.csv | cut -d',' -f29 | sed 's/ /_/g')
  mv \${NAME}_1.fastq.gz \${SP}_\${NAME}_1.fastq.gz
  mv \${NAME}_2.fastq.gz \${SP}_\${NAME}_2.fastq.gz
done" > dataFetch.sh

## Rouler le shell script
chmod +x ./dataFetch.sh
nohup ./dataFetch.sh > dataFetch.log &

```

Maintenant, il faut être patient car le serveur prendra plusieurs minutes (voir heures) pour télécharger 
les données. Le téléchargement se fait "en background" grâce à la commande `nohup [...] &`. Le progrès est 
inscrit dans un fichier ".log", nommé ici `dataFetch.log`. Il est possible de suivre le progrès en temps 
réel en tapant `tail -f dataFetch.log` dans le dossier où les données sont téléchargées. Pour quitter le 
suivi avec tail -f, il faut taper Ctrl+C.

Lorsque le téléchargement est terminé, vérifier combien de fichiers et quelle est leur taille à l'aide de 
la commande `ls -sh`.

- **Question:** Pourquoi y a-t-il plus de fichiers téléchargés que de nombre d'échantillons?

- **Questions:** Quels échantillons ont une meilleure couverture de séquençage? Quels ont une moins bonne 
couverture? Quelles sont les conséquences attendues d'une moins bonne couverture?


Regarder aussi le contenu de quelques fichiers à l'aide de la commande `zcat *.fastq.gz | more`.

- **Question:** Que signifient chacune des 4 lignes associées à chaque lecture Illumina?


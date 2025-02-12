# Assemblage avec ipyrad

**iPyRAD** est un logiciel utilisé en phylogénie pour analyser des données 
génomiques issues de méthodes comme le RADseq, ddRAD, 3RAD, GBS, DartSeq et 
toutes les autres méthodes basées sur la sélection de loci avec des 
enzymes de digestion. Il assemble des séquences brutes en loci homologues entre 
individus ou espèces, corrige les erreurs et produit des jeux de données prêts 
pour l'inférence phylogénétique avec des outils comme IQ-TREE ou RAxML.

---

## Assemblage des données RADseq venant du Short Read Archive

### Préparation du fichier de configuration (params)

Les paramètres contenus dans un fichier de configuration (params) influencent 
les actions réalisées à chaque étape d’un assemblage avec iPyRAD. Les valeurs 
par défaut proposées sont généralement adaptées à la plupart des assemblages. 
Cependant, il est toujours nécessaire de modifier au moins quelques paramètres 
(par exemple, pour spécifier l’emplacement de vos données), et souvent, de 
nombreux paramètres devront être ajustés. L’une des fonctionnalités principales 
d’iPyRAD est la possibilité d’assembler facilement votre jeu de données en 
testant différentes configurations de paramètres.

En premier, il faut générer un fichier de paramètres avec les valeurs par 
défaut:  
```bash
WD=/scratch/$USER/RADseqTest

## Aller au répertoire de travail
cd $WD

## Créer un fichier de paramètres pour ipyrad
conda activate ipyrad
ipyrad -n test

## visionner le contenu du fichier de paramètres qui vient d'être généré
more params-test.txt

```

Une fois cela fait, vous pouvez examiner le contenu de ce fichier de paramètres 
en exécutant la commande `more params-test.txt` ou `cat params-test.txt`. 
Essayez de deviner ce que chaque paramètre influence dans l'assemblage des 
données.  

Ensuite, lisez les instructions détaillées sur le site d'iPyrad pour mieux 
comprendre comment choisir les valeurs de tous les paramètres 
([https://ipyrad.readthedocs.io/en/master/6-params.html](https://ipyrad.readthedocs.io/en/master/6-params.html)). 
Vous pouvez aussi en discuter avec le professeur de votre cours.  

Pour modifier les paramètres, vous pouvez soit modifier ce fichier de 
paramètres manuellement, par exemple en utilisant l'outil `nano` 
(`nano params-test.txt`).  

Alternativement, vous pouvez modifier le script à l'aide de l'outil `sed`, qui 
a l'avantage d'être plus facile à répliquer sans erreur. Pour ce faire, roulez 
le code ci-dessous:  
```bash
WD=/scratch/$USER/RADseqTest
SORTEDREADS=/scratch/$USER/RADseqTest/reads/trim
OVERHANG="TGCAG"
RADTYPE=rad
NAME=test


## Aller au répertoire de travail
cd $WD

## Créer un fichier de paramètres pour ipyrad
conda activate ipyrad
ipyrad -n $NAME

## Ajouter le chemin vers les fichiers de lecture
sed -i "s,                               ## \[4\],$SORTEDREADS/* ## \[4\],g" params-$NAME.txt

## Changer le type de données pour le bon type
sed -i "s,rad                            ## \[7\],$RADTYPE                       ## \[7\],g" params-$NAME.txt

## Définir les bons overhangs des enzymes
sed -i "s/TGCAG,/$OVERHANG,/g" params-$NAME.txt

## Permettre de caller un SNP avec seulement 3 lectures
sed -i "s,6                              ## \[12\],3                              ## \[12\],g" params-$NAME.txt

## Permettre 1 erreur dans le code-barres
sed -i "s,0                              ## \[15\],1                              ## \[15\],g" params-$NAME.txt

## visionner le fichier de paramètres final
cat params-$NAME.txt

```

### Lancer l'assemblage iPyrad

Une fois que le fichier des paramètres d'iPyrad est préparé, il suffit de 
lancer l'analyse sur SLURM:  
```
WD=/scratch/$USER/RADseqTest
NAME=test
EMAIL=votre.courriel@umontreal.ca
TIME="0-3:00:00"
CORES=8

## Créer un fichier SLURM pour exécuter le démultiplexage + découpe/filtrage
echo '#!/bin/bash' > ipyrad_assemble.sbatch
echo "#SBATCH --job-name=ipyrad_assemble
#SBATCH --output=ipyrad_assemble.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

ipyrad -c $CORES -p params-$NAME.txt -s 1234567" >> ipyrad_assemble.sbatch

## Soumettre ce fichier batch à SLURM
sbatch --mail-user=$EMAIL --time=$TIME ipyrad_assemble.sbatch

```

Une fois la tâche envoyée, vous pouvez faire le suivi en exécutant 
`tail -f ipyrad_assemble.out`. Pendant que l'analyse roule, continuez à lire 
les explications sur le [site web d'iPyrad](https://ipyrad.readthedocs.io/en/master/4-data.html).  

Lorsque cette tâche sera terminée, le résultat pourra directement être analysé 
à l'aide d'algorithmes d'estimation phylogénétique.  

---

## Assemblage des données 3RAD non-démultiplexées

Les données 3RAD déposées dans les dépôts publics comme le SRA sont 
généralement déjà démultiplexées, c'est à dire que les lectures de chaque 
échantillon sont déjà séparées dans différents fichiers. Lorsqu'on séquence nos 
propres bibliothèques RADseq, les données ne sont pas démultiplexées et ça 
vient avec un lot d'autres considérations.

Ici, je vais montrer comment assembler un jeu de données 3RAD d'aubépines qui 
a été séquencé en deux bibliothèques 3RAD différentes, avec des fichiers 
formattés différemment, et contenant des échantillons différents (parfois avec 
le même index, mais alors dans des lignes de séquençage différentes). Aussi, 
une des deux bibliothèques 3RAD a été séquencé sur deux lignes différentes, 
donc les fichiers doivent être combinés avant le démultiplexage, et cette même 
bibliothèque contient certains échantillons qu'on ne veut pas inclure dans le 
jeu de données, car ils appartiennent à un projet différent.

### Choisir quelles données analyser

Le première étape est d'identifier la localisation des lectures Illumina 
associées à chaque bibliothèque. Dans notre cas, il y a deux bibliothèques 
3RAD:  
  - Bibliothèque 3rad_2024-06-04_plate1
	Entreposée: `/data/sequenceData/3rad/3rad_2024-06-04_plate1/with_8N_data`
  - Bibliothèque 3rad_20241129_plate1
	Entreposée: ` /data/sequenceData/3rad/3rad_20241129_plate1`

Ensuite, rouler ce code, en s'assurant de bien ajuster le contenu de la 
commande `echo` pour refléter le chemin vers les lectures associées à chacune 
des bibliothèques 3RAD qui sont à analyser.  
```bash
WD=/scratch/$USER/radCrat

mkdir -p $WD
cd $WD

## Créer un tableau nommé "libraries.txt" contenant une ligne par bibliothèque 3RAD, 
## avec les colonnes correspondant au (1) nom de la bibliothèque, (2) chemin vers 
## le fichier ipyrad barcodes, et (3) chemin vers les lectures, séparés par des 
## tabulations:
echo "3rad_2024-06-04_plate1	/data/sequenceData/3rad/3rad_2024-06-04_plate1/ipyrad_barcodes.txt	/data/sequenceData/3rad/3rad_2024-06-04_plate1/with_8N_data/3RAD_test*/*_R[1-2].fastq.gz
3rad_20241129_plate1	/data/sequenceData/3rad/3rad_20241129_plate1/ipyrad_barcodes.txt	/data/sequenceData/3rad/3rad_20241129_plate1/*.fastq.gz" > libraries.txt

```

Les bibliothèques à assembler contiennent des échantillons qui ne sont pas des 
*Crataegus*, le genre focal ici. Ainsi, il faut créer un fichier 
`TaxaToInclude.txt` 

### Assembler avec iPyrad

Une fois que ce fichier `libraries.txt` est créé, il est possible de rouler le 
code ci-dessous pour préparer le dossier et le fichier de paramètres pour 
iPyrad:  
```bash
## ajuster les valeurs de ces variables
WD=/scratch/$USER/radCrat
SAMPLES_TO_KEEP=Crataegus
METHOD=reference
REF_SEQ=/data/genomes/Crataegus_pinnatifida_var._major/Cpinnatifida_major_v1.0.fasta
DATATYPE=pair3rad
OVERHANG=CTAGA\\,CTAGC\\,AATTC
CLUST_THRESHOLD=0.95
BARCODE_MISMATCH=1
MAX_ALLELES=4
MAX_HS=0.15
MAX_SHARED_HS=1
EMAIL=votre.courriel@umontreal.ca
TIME="5-00:00:00"
CORES_DEMULTIPLEXING=8
CORES_ASSEMBLING=16

cd $WD
conda activate ipyrad

## faire une copie du génome de référence
## c'est nécessaire car iPyrad doit pouvoir créer des fichiers dans le dossier 
## dans lequel le génome est sauvegardé, et même avec un lien ça ne marche pas
mkdir -p $WD/genomeRef
cp $REF_SEQ $WD/genomeRef/refseq.fasta

## loop pour chaque bibliothèque (chaque ligne dans libraries.txt)
while IFS=$'\t' read -r -a line
  do
	
	  ## créer un dossier pour la bibliothèque
		mkdir -p "${line[0]}"
		
		## copier les lignes d'intérêt du fichier ipyrad_barcodes.txt
		grep "$SAMPLES_TO_KEEP" "${line[1]}" > ${line[0]}/ipyrad_barcodes.txt
		
		## 
		
		## liens des liens vers les fichiers de lectures Illumina
		for i in $(ls ${line[2]})
		  do
			  ln -s -f $i "${line[0]}/"
			done
			
		## s'assurer que la lecture gauche contient _R1_, et droite _R2_
		rename 's/_1.f/_R1_.f/g' ${line[0]}/*
		rename 's/_2.f/_R2_.f/g' ${line[0]}/*
		rename 's/_R1.f/_R1_.f/g' ${line[0]}/*
		rename 's/_R2.f/_R2_.f/g' ${line[0]}/*
		
		## créer un fichier de paramètres pour la bibliothèque
		ipyrad -n "${line[0]}"
		
		## donner le dossier dans lequel l'assemblage doit être sauvegardé
		sed -i "s,$WD,$WD/${line[0]},g" params-${line[0]}.txt
		
		## ajuster le chemin vers les fichiers de lectures Illumina
		sed -i "s,                               ## \[2\],$WD/${line[0]}/*.f* ##\[2\],g" params-${line[0]}.txt
		
		## ajuster le chemin vers le fichier ipyrad_barcodes.txt
		sed -i "s,                               ## \[3\],$WD/${line[0]}/ipyrad_barcodes.txt ## \[3\],g" params-${line[0]}.txt
		
		## ajuster méthode
		sed -i "s,denovo                         ## \[5\],$METHOD                         ## \[5\],g" params-${line[0]}.txt
		
		## indiquer génome de référence
		sed -i "s,                               ## \[6\],$WD/genomeRef/refseq.fasta ## \[6\],g" params-${line[0]}.txt
		
		## ajuster datatype
		sed -i "s,rad                            ## \[7\],$DATATYPE                            ## \[7\],g" params-${line[0]}.txt
		
		## ajuster overhang
		sed -i "s,TGCAG\,                         ## \[8\],$OVERHANG                         ## \[8\],g" params-${line[0]}.txt

		## ajuster clustering threshold
		sed -i "s,0.85                           ## \[14\],$CLUST_THRESHOLD                           ## \[15\],g" params-${line[0]}.txt

		## ajuster barcode mismatch
		sed -i "s,0                              ## \[15\],$BARCODE_MISMATCH                         ## \[15\],g" params-${line[0]}.txt
		
		## ajuster nb max d'allèles
		sed -i "s,2                              ## \[18\],$MAX_ALLELES                              ## \[18\],g" params-${line[0]}.txt
		
		## ajuster nb max d'hétérozygotes
		sed -i "s,0.05                           ## \[20\],$MAX_HS                           ## \[20\],g" params-${line[0]}.txt
		
		## ajuster proportion maximum d'hétérozygotée partagée
		sed -i "s,0.5                            ## \[24\],$MAX_SHARED_HS                            ## \[24\],g" params-${line[0]}.txt

	done < libraries.txt

## créer un fichier SLURM pour exécuter le démultiplexage + découpe/filtrage
echo '#!/bin/bash' > ipyrad_demult.sbatch
echo "#SBATCH --job-name=ipyrad_demult
#SBATCH --output=ipyrad_demult-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES_DEMULTIPLEXING
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

## aller chercher la bibliothèque Illumina numéro "SLURM_ARRAY_TASK_ID"
RAD_LIB=\$(ls params-*.txt | head -n \$SLURM_ARRAY_TASK_ID | tail -1)

## lancer le démultiplexage
ipyrad -c $CORES_DEMULTIPLEXING -p \$RAD_LIB -s 1" >> ipyrad_demult.sbatch

## Créer un autre fichier SLURM pour exécuter les autres étapes d'assemblage
echo '#!/bin/bash' > ipyrad_assembly.sbatch
echo "#SBATCH --job-name=ipyrad_assembly
#SBATCH --output=ipyrad_assembly.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES_ASSEMBLING
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

## Créer un fichier pour l'analyse combinée de toutes les bibliothèques
ipyrad -m merged params-*.txt

## lancer les autres étapes de l'analyse
ipyrad -c $CORES_ASSEMBLING -p params-merged.txt -s 234567" >> ipyrad_assembly.sbatch

## soumettre les jobs à SLURM
## on utilise la fonction --parsable pour aller chercher le # de job pour 
## les jobs de démultiplexage et on configure la job d'assemblage pour qu'elle 
## débute seulement lorsque les jobs de démultiplexage ont terminées sans 
## erreurs (afterok)
conda activate ipyrad
NFILES=$(ls params-*.txt | wc -l)
JOB_ID=$(sbatch --parsable --array=1-$NFILES --mail-user=$EMAIL --time=$TIME ipyrad_demult.sbatch) &&
  sbatch --dependency=afterok:$JOB_ID --mail-user=$EMAIL --time=$TIME ipyrad_assembly.sbatch

```

### Téléchargez les résultats

Une fois l'assemblage terminé, il peut être utile de télécharger les fichiers 
sur votre ordinateur personnel pour pouvoir les examiner plus facilement. À 
partir de votre ordinateur, tapez les lignes ci-dessous, en modifiant de façon 
appropriée les variables qui indiquent le chemin vers les fichiers enregistrés 
sur votre ordinateur et le chemin d'arrivée sur le serveur de calcul:  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
CLUSTER_USERNAME=elbourret
LOCAL=/mnt/c/Users/Etienne/Downloads
REMOTE=/scratch/$CLUSTER_USERNAME/radCrat/*/merged_outfiles


rsync \
  --progress \
  $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/merged.snps \
	$LOCAL/
rsync \
  --progress \
  $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/merged_stats.txt \
	$LOCAL/

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

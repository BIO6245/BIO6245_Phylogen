# Préparations des données test HybSeq de *Carex*

Ce jeu de données a été généré par Étienne Lacroix-Carignan et Maurane
 Bourgouin dans le cadre de leur doctorat. Il s'agit de données de séquençage
 HybSeq sur des *Carex* subsect. *Lupulinae* et hors-groupes basé sur les sondes
 spécifiques aux *Carex* de Tamara Villaverde et des sondes Angio353.

Le but est de préparer un jeu de données qui représente bien les données
 brutes de séquençage Illumina d'une bibliothèque HybSeq.

J'ai sélectionné 7 taxons à représenter dans ce jeu de données. Je veux
 ensuite m'assurer que chaque échantillon contient uniquement 0.25M lectures
 pairées pour réduire la complexité des analyses. Dans un fichier .fastq,
 chaque lecture est encodée comme 4 lignes, donc on veut conserver 1M de lignes
 .fastq = 0.25M de lectures Illumina.
 
Ensuite, j'ai créé un "hybride" in-silico en combinant 0.125M de lectures
 de *Carex lupulina* et 0.125M de lectures de *Carex gator*.

Voici le code pour conserver maximum 0.25M lectures pairées par échantillon:
```bash
WD=/data/testdata/HybSeq_Carex

sudo mkdir - $WD/refs
cd $WD/refs

## Copier les échantillons sélectionnés de la plaque de séquençage
PLAQUE=/data/sequenceData/hybseq/hybseq_20250314_Maurane_Cassandra_ELC
ln -s $PLAQUE/Carex_retrorsa* .
ln -s $PLAQUE/Carex_gator* .
ln -s $PLAQUE/Carex_gigantea* .
ln -s $PLAQUE/Carex_louisianica* .
ln -s $PLAQUE/Carex_lupulina* .

## créer les fichiers finaux pour les espèces
cd $WD
for i in $WD/refs/*.fastq.gz
  do
	  SAMPLE_NAME=$(basename $i .fastq.gz)
		zcat $i | head -n 1000000 | gzip > $WD/$SAMPLE_NAME.fastq.gz
	done

## créer un "hybride" in-silico
cd $WD
zcat Carex_gigantea*_R1.fastq.gz | head -n 500000 \
  > Carex_gigantea_x_louisianica_R1.fastq
zcat Carex_louisianica*_R1.fastq.gz | head -n 500000 \
  > Carex_gigantea_x_louisianica_R1.fastq
zcat Carex_gigantea*_R2.fastq.gz | head -n 500000 \
  > Carex_gigantea_x_louisianica_R2.fastq
zcat Carex_louisianica*_R2.fastq.gz | head -n 500000 \
  > Carex_gigantea_x_louisianica_R2.fastq

## gzipper les données de l'hybride
gzip Carex_gigantea_x_louisianica*.fastq

## supprimer les liens dans du fichier refs
cd $WD
rm -r ./refs
```


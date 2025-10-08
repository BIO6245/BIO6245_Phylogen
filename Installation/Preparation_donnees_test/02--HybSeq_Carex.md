# Préparations des données test HybSeq de *Carex*

Ce jeu de données a été généré par Étienne Lacroix-Carignan et Maurane 
Bourgouin dans le cadre de leur doctorat. Il s'agit de données de séquençage 
HybSeq sur des Carex subsect. Lupulinae et hors-groupes basé sur les sondes 
spécifiques au Carex de Tamara Villaverde et des sondes Angio353.

Le but est de préparer un jeu de données qui représente bien les données 
brutes de séquençage Illumina d'une bibliothèque HybSeq.

Tout d'abord, j'ai sélectionné 12 taxons à représenter dans ce jeu de données 
et je les ai placé dans le dossier `/data/testdata/HybSeq_Carex`. Je veux 
ensuite m'assurer que chaque échantillon contient uniquement 0.5M lectures 
pairées pour réduire la complexité des analyses. Dans un fichier .fastq, 
chaque lecture est encodée comme 4 lignes, donc on veut conserver 4M de lignes 
.fastq = 0.5M de lectures Illumina.

Voici le code pour conserver maximum 0.5M lectures pairées:
```bash
WD=/data/testdata/HybSeq_Carex

mkdir - $WD/downsampled
cd $WD

for i in *.fastq.gz
  do
	  SAMPLE_NAME=$(basename $i .fastq.gz)
		zcat $i | head -n 2000000 | gzip > $WD/downsampled/$SAMPLE_NAME.fastq.gz
	done

```
# Assemblage avec ipyrad

**iPyRAD** est un logiciel utilisé en phylogénie pour analyser des données génomiques issues de méthodes comme 
le RADseq, ddRAD, 3RAD, GBS, DartSeq et toutes les autres méthodes basées sur la sélection de loci avec des 
enzymes de digestion. Il assemble des séquences brutes en loci homologues entre individus ou espèces, corrige 
les erreurs et produit des jeux de données prêts pour l'inférence phylogénétique avec des outils comme IQ-TREE 
ou RAxML.

## Assemblage des données RADseq

Les paramètres contenus dans un fichier de configuration (params) influencent les actions réalisées à chaque 
étape d’un assemblage avec iPyRAD. Les valeurs par défaut proposées sont généralement adaptées à la plupart des 
assemblages. Cependant, il est toujours nécessaire de modifier au moins quelques paramètres (par exemple, pour 
spécifier l’emplacement de vos données), et souvent, de nombreux paramètres devront être ajustés. L’une des 
fonctionnalités principales d’iPyRAD est la possibilité d’assembler facilement votre jeu de données en testant 
différentes configurations de paramètres.

En premier, il faut générer un fichier de paramètres avec les valeurs par défaut:  
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

Une fois cela fait, vous pouvez examiner le contenu de ce fichier de paramètres en exécutant la commande 
`more params-test.txt` ou `cat params-test.txt`. Essayez de deviner ce que chaque paramètre influence dans 
l'assemblage des données.  

Ensuite, lisez les instructions détaillées sur le site d'iPyrad pour mieux comprendre comment choisir les 
valeurs de tous les paramètres ([https://ipyrad.readthedocs.io/en/master/6-params.html](https://ipyrad.readthedocs.io/en/master/6-params.html)). 
Vous pouvez aussi en discuter avec le professeur de votre cours.  

Pour modifier les paramètres, vous pouvez soit modifier ce fichier de paramètres manuellement, par exemple en 
utilisant l'outil `nano` (`nano params-test.txt`).  

Alternativement, vous pouvez modifier le script à l'aide de l'outil `sed`, qui a l'avantage d'être plus facile 
à répliquer sans erreur. Pour ce faire, roulez le code ci-dessous:  
```bash
WD=/scratch/$USER/RADseqTest
SORTEDREADS=/scratch/$USER/RADseqTest/reads/trim
OVERHANG="TGCAG"
RADTYPE=rad
NAME=test
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-3:00:00"
CORES=8

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

## Créer un fichier SLURM pour exécuter le démultiplexage + découpe/filtrage
echo '#!/bin/bash' > ipyrad_assemble.sbatch
echo "#SBATCH --job-name=ipyrad_assemble
#SBATCH --output=ipyrad_assemble.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CORES
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

ipyrad -c $CORES -p params-test.txt -s 1234567" >> ipyrad_assemble.sbatch

## Soumettre ce fichier batch à SLURM
sbatch --mail-user=$EMAIL --time=$TIME ipyrad_assemble.sbatch

```

Une fois la tâche envoyée, vous pouvez faire le suivi en exécutant `tail -f ipyrad_assemble.out`. Pendant que 
l'analyse roule, continuez à lire les explications sur le [site web d'iPyrad](https://ipyrad.readthedocs.io/en/master/4-data.html).  

Lorsque cette tâche sera terminée, le résultat pourra directement être analysé à l'aide d'algorithmes 
d'estimation phylogénétique.  


# Filtrage des alignements

Plusieurs sources d'erreur peuvent survenir lors du séquençage, de l'assemblage et de l'alignement 
qui peuvent résulter en des erreurs d'alignement, où les nucléotides d'une espèce ne sont pas du 
tout aligner avec les nucléotides homologues d'une autre espèce.

Quelques exemples d'erreur:  
- Si l'ADN d'une espèce est de plus faible qualité, il peut résulter une plus faible couverture de 
séquençage, menant à un plus grand nombre d'erreurs de séquençage. En conséquence, certaines 
"mutations" observées chez cette espèce peuvent être seulement dû à des erreurs de séquençage.  
- Dans une région particulièrement riche en mutations de type insertion et délétion, il peut être 
plus difficile pour les algorithmes d'alignement automatisés de créer un alignement fiable.  
- Les mutations de type inversion causent presque automatiquement des erreurs d'alignement, car la 
séquence ACTG inversée devient GTCA, interprétée comme 4 mutations différentes dans un alignement.
- Si des éléments transposables sont insérés au centre d'une séquence, ou si le locus séquencé a 
changé de position dans le génome, ça peut mener à des nucléotides présents dans la séquence d'une 
espèce qui n'ont pas d'équivalent dans la séquence des autres espèces.  
- etc.  

En conséquence, il est préférable d'examiner les alignements à l'oeil pour déceler les erreurs, 
leur fréquence et leur source probable.

Une fois la fréquence et la source des erreurs identifiée, on peut soit:  
1. laisser les erreurs s'ils sont peu fréquentes et ne risquent pas de confondre le résultat 
final;  
2. corriger à la main les erreurs (lorsque la quantité de données est assez faible);  
3. retirer les séquences problématiques ou les "masquer" en les transformant en "?", des "-" ou 
des "N";  
4. utiliser des programmes pour retirer et/ou masquer les zones de séquences problématiques.  

Nous allons ici tester quelques programmes qui filtrent les alignements en retirer les colonnes 
et les zones qui semblent être moins bien alignées.  

---

## Masquer les zones qui sont anormalement divergentes à l'aide de TAPER

[TAPER](https://github.com/chaoszhang/TAPER) est une approche qui identifie des zones de 
divergence anormalement élevée par rapport au reste de la séquence de la même espèce. Ces zones 
sont ensuite masquées en les transformant en "N" (ambiguïté). Voici du code pour exécuter TAPER sur 
les alignements:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-3:00:00"
THREADS=4
FILTERING_CUTOFF=1
MASKING_CHAR=N


###
## Filtrer les alignements d'exons
###

ALIGNS_PATH=$WD/seqs/exon/align

mkdir -p $ALIGNS_PATH/taper
cd $ALIGNS_PATH

## Créer un fichier contenant la liste des alignements à filtrer avec TAPER
echo "" > align.list
for a in $ALIGNS_PATH/*.fasta
  do
    BASENAME=$(basename $a .fasta)
	echo "$a" >> align.list
	echo "$ALIGNS_PATH/taper/$BASENAME.fasta" >> align.list
  done

## créer un batchfile pour exécuter TAPER sur cette liste
echo '#!/bin/bash' > TAPER.sbatch
echo "#SBATCH --job-name=TAPER
#SBATCH --output=TAPER.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$THREADS
#SBATCH --mem-per-cpu=1G
#SBATCH --time=$TIME

## lancer TAPER sur la liste des gènes dans align.list
julia -t $THREADS $SRC/TAPER-1.0.0/correction_multi.jl \\
  --mask $MASKING_CHAR \\
  --cutoff $FILTERING_CUTOFF \\
  --list align.list" >> TAPER.sbatch


## envoyer la tâche TAPER à SLURM
sbatch --mail-user=$EMAIL TAPER.sbatch

```

- **Question**: Est-ce que TAPER a masqué beaucoup de zones dans ces séquences? Examinez les 
alignements avec `more ./taper/*.fasta` pour le savoir. Vous pouvez aussi importer quelques 
alignements dans MEGA sur votre propre ordinateur. Notez que TAPER masque les caractères qui sont 
trop divergents avec des "N" dans ce script. 

- **Question**: Modifiez manuellement un des alignements en l'éditant avec `nano`, de façon à 
introduire des erreurs. Sauvegardez, puis supprimer le fichier avec les résultats de TAPER en 
exécutant `rm ./taper/*.fasta`, puis ré-exécuter tout le code ci-dessus. Est-ce que les régions où 
vous avez introduit des erreurs ont été filtrées?

- **Question**: La variable "cutoff" indique le niveau d'aggressivité à utiliser avec TAPER. Plus 
cutoff est petit, plus TAPER filtre aggressivement, c'est à dire qu'il va masquer des régions plus 
souvent. Essayez de modifier la variable cutoff (valeur entre 0.5 et 5) et ré-exécutez tout le code 
(après avoir effacé les résultats de la dernière exécution). Est-ce que ça fait une différence?

---

## Filtrer les régions des alignements qui contiennent trop de caractères indéterminés

**[TrimAl](https://vicfero.github.io/trimal/)** est un outil utilisé pour en supprimer les 
positions mal alignées ou contenant des indels. Il optimise les alignements en supprimant les 
colonnes peu informatives. Voici du code pour exécuter trimAL sur les alignements:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-3:00:00"

## Ajuster aussi les paramètres de trimAl si vous voulez
WINDOW_SIZE=3
MIN_NONGAP_PERCENT=0.1
MIN_SIMILARITY=0.1
MIN_OVERLAP=0.5


###
## Filtrer les alignements d'exons
###

ALIGNS_PATH=$WD/seqs/exon/align

mkdir -p $ALIGNS_PATH/trimal
cd $ALIGNS_PATH

## créer un batchfile pour filtrer les alignements
echo '#!/bin/bash' > trimal.sbatch
echo "#SBATCH --job-name=trimal
#SBATCH --output=trimal-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=$TIME

## sélectionner l'alignement à filtrer dans cette tâche
ALIGNIN=\$(ls $ALIGNS_PATH/taper/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)

## déterminer le nom du gène
GENENAME=\$(basename \$ALIGNIN .fasta)

## lancer trimAl
$SRC/trimAl_1.5.0/trimal \\
  -in \$ALIGNIN \\
  -out ./trimal/\$GENENAME.fasta \\
  -w $WINDOW_SIZE \\
  -gapthreshold $MIN_NONGAP_PERCENT \\
  -simthreshold $MIN_SIMILARITY \\
  -resoverlap $MIN_OVERLAP \\
  -seqoverlap $MIN_OVERLAP" >> trimal.sbatch

## exécuter trimAl sur tous les alignements filtrés par TAPER
NFILES=$(ls -1 $ALIGNS_PATH/taper/*.fasta | wc -l)
sbatch --mail-user=$EMAIL --array=1-$NFILES trimal.sbatch

```

### Paramètres de TrimAl

- **-w**: Utilise un alignement pondéré en ajustant les séquences en fonction de leur similarité 
évolutive, pour donner plus de poids aux séquences représentatives.  
- **-gapthreshold**: Définit un seuil de pourcentage pour les gaps dans les colonnes. Par exemple, 
`-gapthreshold 0.3` permet de conserver les colonnes avec moins de 30% de gaps.  
- **-simthreshold**: Contrôle le seuil de similarité entre les séquences. Par exemple, 
`-simthreshold 0.7` conserve uniquement les colonnes dont la similarité est d’au moins 70% entre 
les séquences.  
- **-resoverlap**: Définit le nombre minimum de résidus non-gaps requis dans une séquence pour que 
cette séquence soit conservée.  
- **-seqoverlap**: Définit le nombre minimum de séquences qui doivent contenir un résidu 
spécifique pour qu'il soit conservé dans l'alignement.  

Vous pouvez modifier ces paramètres, ou même avoir une liste complète de toutes les options en 
exécutant `/opt/trimAl_1.5.0/trimal -h`

---

## Conserver uniquement les alignements filtrés avec >4 séquences

Lorsque cela est fait, le résultat est des alignements plus petits qui peuvent contenir moins de 
séquences, car certains échantillons auront toutes leurs positions d'alignement filtrées par TrimAl 
pour certains loci. Nous devons donc à nouveau supprimer les loci qui contiennent moins de 
4 échantillons (après le filtrage par TAPER et TrimAl):  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest

## find loci with data for <4 samples, and remove them (they cannot be used to estimate a tree)
cd $WD/seqs/exon/align/trimal
for i in $(ls *.fasta)
  do
    NSAMPLES=$(grep '>' $i | wc -l)
    if [ $NSAMPLES -lt 4 ]
      then
        echo "$i contains $NSAMPLES samples, which is <4, so it was removed!"
        rm $i
    else
	  echo "$i contains $NSAMPLES samples."
    fi
  done >> filter_min4_after_trimal.log

```



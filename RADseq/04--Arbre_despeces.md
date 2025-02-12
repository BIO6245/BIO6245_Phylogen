# Estimation d'un arbre d'espèces

Ce tutoriel vous montrera comment estimer un arbre d'espèces à partir de 
données RADseq. Il existe trois grandes approches:   
  1. Co-estimation des arbres de gènes et de l'arbre d'espèce (p.ex., 
  **\*BEAST** (=**starBEAST**);
  2. Estimation des arbres de gènes individuels, puis estimation d'un arbre 
	d'espèce avec une méthode par abrégé (p.ex., **ASTRAL**);
  3. Estimation de l'arbre d'espèce directement à partir des patrons de chaque 
	position individuelle de l'alignement, sans passer par l'estimation d'arbres 
	de gènes (p.ex., **CASTER**, **svdQuartets**)

Malheureusement, la co-estimation des arbres de gènes et de l'arbre d'espèce 
avec \*BEAST demande beaucoup trop de ressources de calcul pour être faisable 
avec un jeu de données génomiques comme le RADseq. C'est efficace uniquement 
pour un jeu de données de 10 à 50 gènes maximum. Pour des centaines de gènes, 
il faut utiliser des méthodes par abrégé ou des méthodes qui estiment 
directement à partir des patrons de l'alignement.  

---

## Analyse des données 3RAD des aubépines

### Estimation de l'arbre d'espèces avec CASTER

Il existe plusieurs "variétés" d'algorithmes ASTRAL, chacun avec des avantages 
et inconvénients:  
  - ASTRAL: l'algorithme standard, qui prend en compte seulement la topologie 
  des arbres de gènes, sans évaluer les longueurs de branches et le support 
  statistique.
  - ASTRAL-Pro: permet d'utiliser des arbres de gènes qui contiennent des 
  duplications (arbres de   famille de gènes avec plusieurs copies dans chaque 
	espèce).
  - weighted ASTRAL ou wASTRAL: comme l'algorithme standard, mais pondère la 
	valeur de chaque quartet en fonctionne de la longueur des branches et/ou 
	support bootstrap associé, ce qui fait que les clades qui ont un faible 
	support dans les arbres de gènes auront moins d'influence sur le résultat 
	final de l'arbre d'espèce. Ça donne des résultats plus fiables.
  - CASTER: méthode qui part des alignements directement.
  - WASTER: méthode qui part des séquences non alignées.

Nous allons utiliser la méthode CASTER-site, qui est la plus fiable lorsqu'on a 
des loci qui sont de courte taille, avec peu de variation à l'intérieur de 
chaque locus, comme dans les données RADseq. CASTER-site est préférable à 
CASTER-pair ici, car on a de l'hétérozygosité non-phasée due à la présence de 
polyploïdes.  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_ASTER=/opt/ASTER/bin
WD=/scratch/$USER/radCrat/caster
ALIGNMENT=/scratch/$USER/radCrat/*/merged_outfiles/merged.snps
OUTPUT_NAME=CratQC_caster-site.tre
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
MEM_PER_CORE=4M
CORES=16

## créer le dossier où on sauvegardera les résultats
mkdir -p $WD
cd $WD

## effectuer l'analyse
sbatch \
  --job-name=caster-site \
	--output=caster-site.out \
  --mail-user=$EMAIL \
	--time=$TIME \
	--cpus-per-task=$CORES \
	--mem-per-cpu=$MEM_PER_CORE \
	--wrap="$SRC_ASTER/caster-site \
		--thread $CORES \
    --format phylip \
		--ambiguity 1 \
		--output $OUTPUT_NAME \
		--input $ALIGNMENT"

```

Téléchargez l'arbre d'espèces qui a été estimé et visualisez-le avec 
FigTree.


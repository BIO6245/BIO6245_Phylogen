# Sélection des modèles, partitions et arbres de distance

Ce tutoriel vous guidera dans l’utilisation de **PAUP\*** et **IQ-Tree** pour la sélection des 
modèles de substitution, le partitionnement des données, et l’estimation d’arbres de distance. 
Il vous fournira des exemples concrets de commandes et d’explications pour chaque étape.

---

## Sélection des modèles et du partitionnement dans IQ-Tree

**IQ-Tree** offre une approche rapide et flexible pour la sélection de modèles et le 
partitionnement. Il permet de tester plusieurs partitions et modèles pour chaque partition.

Le fichier [`test.nex`](fichiers/test.nex) contient les données provenant de deux gènes 
nucléaires, le premier gène étant représenté par 13 nucléotides (positions 1 à 13 dans 
l'alignement), et le deuxième par 6 nucléotides (positions 14 à 20). Les deux gènes encodent des 
protéines, donc leurs nucléotides sont organisés en codons de 3 nucléotides, et chaque codon peut 
évoluer à une vitesse différente, étant donné que les mutations ont plus de chance de changer 
l'acide aminé encode si elles surviennent dans les nucléotides en première position d'un codon, et 
moins d'influence si elles surviennent en 3e position.

Nous allons donc partitionner l'alignement par gène et par position dans les codons, et déterminer 
automatiquement quel est le ou les modèles et le ou les partitions qui sont les mieux supportées 
avec l'aide de **IQ-Tree**

### Étapes à suivre pour l'analyse IQ-Tree

**IQ-Tree** dispose d'une fonction intégrée pour tester plusieurs modèles de substitution de 
séquences.

1. Assurez-vous d'avoir déjà **transféré le fichier Nexus** [`test.nex`](fichiers/test.nex) depuis 
votre ordinateur local vers le serveur de calcul à l'aide de la commande `rsync`.

2. **Connectez-vous au serveur**.  

3. Créer un fichier de partitions nommé `partitions.txt` qui contient les informations sur le 
partitionnement naturel de vos données (par exemple, avec l'aide de la commande 
`nano partitions.txt`). Le fichier doit contenir ceci:  
   ```
   DNA, gene1_codon1 = 1-13\3
   DNA, gene1_codon2 = 2-13\3
   DNA, gene1_codon3 = 3-13\3
   DNA, gene2_codon1 = 14-20\3
   DNA, gene2_codon2 = 15-20\3
   DNA, gene2_codon3 = 16-20\3
   ```

4. Exécuter IQ-Tree avec partitionnement :
   ```bash
   ## Ajuster les variables ci-dessous de façon appropriée
   SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
   WD=/home/$USER
   ALIGNMENT=/home/$USER/test.nex
   PARTITIONS=/home/$USER/partitions.txt
   EMAIL=votre.courriel@umontreal.ca
   TIME="0-12:00:00"
   THREADS=1
      
   sbatch \
     --job-name=iqtree_modeltest \
     --output=iqtree_modeltest.log \
     --mail-user=$EMAIL \
     --nodes=1 \
     --time=$TIME \
     --cpus-per-task=$THREADS \
     --mem-per-cpu=2G \
     --wrap="$SRC_IQ/iqtree2 \
	   -s $ALIGNMENT \
	   -p $PARTITIONS \
	   -m TESTMERGEONLY \
	   -nt $THREADS"
   
   ```
   - la ligne débutant par `SRC_IQ=` initialise une variable qui indique où se trouve le programme 
   **IQ-Tree**.
   - `-p partitions.txt` : Spécifie dans quel fichier se trouve les partitions.
   - `-m TESTMERGEONLY` : Sélectionne automatiquement le meilleur modèle pour chaque partition et 
   fusionne les partitions similaires pour déterminer le meilleur modèle de partitionnement.

5. Examiner le rapport de l'analyse en exécutant la commande `more partitions.txt.iqtree`.
   
   - **Question**: Quel est le meilleur partitionnement des données?
   - **Question**: Quel est le meilleur modèle pour chacune des partitions? Inclut-il un courbe 
   gamma de variation des taux évolutifs?

---

## Estimation d'arbres de distance dans PAUP\*

**PAUP\*** demeure un des programmes les plus polyvalents pour l'estimation de matrices de 
distance variées et l'estimation d'arbres de distance avec des algorithmes standards comme le 
*Unweighted Pair Group Mean Averaging* (UPGMA) et le *Neighbor-Joining* (NJ). Toutefois, notez 
que PAUP\* ne permet pas de spécifier de partitions.

### Chargement du fichier d'alignement Nexus dans PAUP\*

1. Assurez-vous d'avoir déjà **transféré le fichier Nexus** [`test.nex`](fichiers/test.nex) depuis 
votre ordinateur local vers le serveur de calcul à l'aide de la commande `rsync`.

2. **Connectez-vous au serveur**.  

3. **Ouvrez le fichier Nexus** dans PAUP*:  
   ```paup
   /opt/paup4a168_centos64 test.nex
   
   ```

4. Vérifiez que les données sont bien chargées avec la commande:  
   ```paup
   showmatrix;
   
   ```
5. Regardez la liste des commandes en exécutant `?` dans PAUP*:
   ```paup
   ?;
   
   ```
   
6. Ajuster certains paramètres de base dans PAUP*:  
   ```paup
   set autoclose=yes;
   set taxlabels=full;
   set increase=auto autoinc=100 autoclose=yes;
   set warnreset=no warntree=no;
   
   ```

### Utilisation de PAUP* pour estimer des arbres de distance

1. Calculer une matrice de distance en utilisant le modèle "Felsenstein 1984" (`distance=F84`) et 
en assumant aucune variation de taux entre les sites (`rates=equal`):  
   ```paup
   dset distance=F84 rates=equal;
   showdist;
   
   ```
   - **Question**: Quelles sont les espèces les plus semblables? Quelle est leur distance?
   - **Question**: Quelles sont les espèces les moins semblables? Quelle est leur distance?

2. Essayez de recalculer les distance en utilisant un modèle différent, si possible le meilleur 
modèle sélectionné par **IQ-Tree**. Pour voir les options de modèle disponibles dans PAUP\*, 
exécutez la commande `dset ?` ou allez voir dans le manuel en ligne...

   - **Question**: Est-ce que la distance entre les espèces a changé par rapport au modèle 
   précédent?

3. Estimer l'arbre avec les algorithmes Neighbor-Joining (NJ) ou UPGMA:  
   ```paup
   nj;
   
   ```

   ou

   ```paup
   upgma;
   
   ```

   - **Question**: Quelles sont les différences entre ces arbres?
   - **Question**: Est-ce que les positions relatives entre les espèces a changé, ou seulement la 
   longueur des branches?

4. Changez le modèle et ré-estimez les arbres.  
   
   - **Question**: Remarquez-vous des changements?

5. Il est aussi possible de chercher le meilleur arbre avec des algorithmes exacts 
(commande `bandb`) ou heuristiques (commande `hsearch`) en utilisant le critère de distance pour 
sélectionner le meilleur arbre. C'est la méthode des "moindres carrés" (least-squares). Dans ce 
cas, PAUP\* sélectionne l'arbre qui représente le mieux les distances entre chaque paire d'espèce 
en comparant la matrice des distances originales aux distances "phénétiques" calculées le long 
des branches de l'arbre. Voici comment faire:  
   ```paup
   set criterion=distance;
   dset objective=lsfit;
   hs;
   descr 1;
   
   ```
   
   - `set criterion=distance` indique qu'on veut chercher les meilleurs arbres en fonction de la 
   distance, plutôt que la parcimonie.
   - `dset objective=lsfit` indique qu'on veut utiliser le *least-squares fit* (moindres carrés de 
   la différence entre les distances originales de la matrice et les distances calculées le long 
   des branches de l'arbre.
   
   - **Question**: Comment cet arbre se compare aux précédents?
   
6. Finalement, PAUP\* permet aussi de chercher le meilleur arbre en utilisant le critère 
"d'évolution minimale" (*Minimum Evolution* ou *ME*). Voici comment faire:  
   ```paup
   set criterion=distance;
   dset objective=me;
   hs;
   descr 1;
   
   ```
   
   - `dset objective=me` signifie qu'on veut utiliser le critère de *Minimum Evolution*.
   
   - **Question**: Comment cet arbre se compare aux précédents?
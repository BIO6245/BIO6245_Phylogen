# Estimation des états ancestraux

Ce tutoriel vous guidera dans l'utilisation de **Mesquite** et de **R** pour
 estimer l'évolution de caractères sur une phylogénie (estimation des
 états ancestraux, *ancestral state estimation*). On traitera deux types
 de caractères:

  - un caractère **discret** (des catégories, par ex. couleur de la fleur,
 habitat, présence/absence d'une structure);
  - un caractère **continu** (une mesure, par ex. taille, longueur d'une
 feuille, nombre de chromosomes).

Contrairement aux tutoriels précédents, **toutes les analyses se font sur**
 **votre ordinateur personnel**, pas sur le serveur de calcul. Ces analyses sont
 rapides (quelques secondes) parce qu'elles partent d'un arbre déjà estimé.

---

## Le jeu de données

On utilisera un jeu de données publié sur les anguilles (Anguilliformes;
 [Collar et al. 2014](https://doi.org/10.1038/ncomms6505)), fourni avec le
 paquet R  **phytools**:

  - un arbre daté (ultramétrique) de 61 espèces;
  - un caractère discret: le **mode d'alimentation**, soit par morsure
 (*bite*) ou par succion (*suction*);
  - un caractère continu: la **longueur maximale** du corps (cm).

La question biologique: **quel était le mode d'alimentation de l'ancêtre**
 **commun des anguilles**, et combien de fois ce mode a-t-il changé au cours de
 leur évolution?

Pour Mesquite, les mêmes données sont fournies dans le fichier
 `anguilles.nex` (le code R qui a servi à produire ce fichier est donné en
 annexe). Dans ce fichier, la longueur est déjà transformée en logarithme
 (voir la section R pour la raison).

---

## Installation des programmes

### Mesquite

  1. Télécharger la version la plus récente (4.x) pour votre système
  d'exploitation à partir de la page des versions sur GitHub:
  <https://github.com/MesquiteProject/MesquiteCore/releases>
  (site principal du projet: <https://www.mesquiteproject.org>).
  2. Décompresser l'archive et déplacer le dossier `Mesquite_Folder` (ou
  l'application sur Mac) à l'endroit de votre choix.
  3. Mesquite est écrit en **Java**. S'il ne démarre pas, installez Java
  puis réessayez.

### R et RStudio

  1. Installer **R** à partir du CRAN: <https://cran.r-project.org> (choisir
  votre système d'exploitation, puis la version de base, *base*).
  2. Sur Windows, installer la version la plus récente de **RTools**: 
  <https://cran.r-project.org/bin/windows/Rtools/>
  3. (Recommandé) Installer **RStudio Desktop**, une interface conviviale pour
  R: <https://posit.co/download/rstudio-desktop/>. RStudio a besoin que R soit
  déjà installé.
  4. Ouvrir R ou RStudio et installer le paquet **phytools** (qui installe
  aussi **ape** automatiquement) en tapant dans la console:
  ```r
  install.packages("phytools")
  ```

Il faut installer un paquet **une seule fois** par ordinateur, mais il faut le
 charger (avec p.ex., `library("phytools")`) **à chaque nouvelle session** de R.

---

## Estimation des états ancestraux dans Mesquite

**Mesquite** est un logiciel avec interface graphique, très pratique pour
 explorer visuellement l'évolution de caractères sur un arbre. Il permet
 l'estimation des états ancestraux par **parcimonie** (caractères discrets et
 continus) et par **maximum de vraisemblance** (caractères discrets seulement).

Prérequis:
  - Avoir installé Mesquite;
  - Avoir le fichier `anguilles.nex`.

Attention! Le nom exact de certains menus peut varier légèrement d'une version
 de Mesquite à l'autre. Si un menu n'est pas exactement où il est indiqué,
 cherchez une option au nom semblable dans les menus voisins.

### Ouvrir le fichier et afficher l'arbre

  1. Ouvrir le fichier `anguilles.nex` dans un éditeur texte et examinez la
  structure par blocs. Vous devriez remarquer qu'il y a un bloc avec le nom des
  échantillons, un bloc avec les données qualitatives sur le mode 
  d'alimentation, un bloc avec les données de taille moyenne par espèce,
  transformées par le logarithme naturel, et un bloc avec un arbre 
  phylogénétique ultramétrique qui a déjà été estimé auparavant. Assurez-vous
  que vous seriez en mesure de récréer vous même ce types de blocs si vous aviez
  à faire l'analyse de vos propres données (en combinant les résultats de vos
  analyses phylogénétiques avec des données sur les traits).
  2. Démarrer Mesquite, puis `File > Open File...` et choisir
  `anguilles.nex`.
  3. La fenêtre du projet montre les blocs lus: les taxons (61), deux
  matrices de caractères (`Alimentation` et `Taille`) et un bloc d'arbres.
  4. Ouvrir une fenêtre d'arbre: 
  `Taxa&Trees > New Tree Window > With Trees From Source` (choisir
    l'arbre stocké, *Stored Trees*).
  5. Pour que les longueurs de branches soient représentées ainsi que les temps
  de divergence, sélectionner `Tree > Chronogram Display`.
  6. Comparer l'enracinement de l'arbre avec celui de l'étude d'où il provient
  ([Collar et al. 2014](https://doi.org/10.1038/ncomms6505)). S'il n'est pas
  bien enraciné, le réenraciner avec l'outil `Reroot at branch` dans la barre
  d'outils (pictogrammes) à gauche de l'arbre.

### Caractères discrets par parcimonie

  1. Dans la fenêtre d'arbre: `Analysis:Tree > Reconstruct Ancestral States`.
  2. Choisir **Parsimony Ancestral States** dans la fenêtre, puis **Alimentation** dans la suivante.

Les branches sont colorées selon l'état estimé. Le nombre de pas (critère de 
 parcimonie) est aussi estimé pour ce caractère.

### Caractères discrets par maximum de vraisemblance

  1. Ouvrir une nouvelle fenêtre avec le même arbre phylogénétique avec
  `Taxa&Trees > New Tree Window > With Trees from Source`, et le
  représenter encore une fois comme un chronogramme. 
  2. Relancer `Analysis:Tree > Reconstruct Ancestral States`.
  3. Sélectionner **Stored Probability Model**.
  4. Sélectionner encore **Alimentation** comme caractère à estimer.
  5. Deux choix de modèles évolutifs s'offrent à vous: un modèle avec des taux
  de transitions asymmétriques (Asymm. 2 param.), où le taux d'évolution de
  l'état 0->1 est différent du taux de changement de 1->0, et un modèle Mk1 qui
  suppose que les taux sont symmétriques ( taux(0 -> 1) = taux(1 -> 0) ). Le
  modèle avec des taux asymmétriques est aussi nommé AsymmMk.

Les  branches sont maintenant bicolores lorsque le modèle n'est pas certain
 de quel état ancestral est le plus probable. C'est le cas pour la majorité des
 noeuds, et c'est aussi l'avantage d'une méthode probabiliste comme le maximum
 de vraisemblance: ça prend en compte l'incertitude. Si vous placez votre
 curseur sur une branche, la légende affichera la **vraisemblance**
 **proportionelle** de chaque état de caractère pour cette branche, qui
 correspond au ratio entre la vraisemblance de l'état, divisé par la somme des
 vraisemblances de tous les états de caractère possibles. Ce n'est pas une
 probabilité à proprement parlé, comme en analyse bayesienne, mais on peut
 l'interpréter un peu de la même façon: plus on se rapproche de 1, plus le
 modèle supporte fortement cette estimation.
 sont maintenant des **diagrammes en pointe de tarte**: la
 proportion de chaque couleur représente la **vraisemblance proportionnelle**
 de chaque état à ce noeud.
 
La légende présente aussi la vraisemblance globale du modèle en fonction de ce
 caractère (-log L).

  6. Refaire l'analyse avec l'autre modèle dans une nouvelle fenêtre.

- **Question**: Comparez les vraisemblances globales (-log L) des modèles Mk1
 et Asymm. 2 param. Quel est le meilleur modèle? Est-ce que la différence
 entre les deux modèles est significative? (Indice: un test du rapport
 de vraisemblance compare 2 × Δlog L à une distribution du χ² à 1 degré de
 liberté; la valeur critique à 5 % est 3,84.)
- **Question**: Est-ce que les estimations d'états ancestraux se ressemblent
 entre les modèles de vraisemblance et la parcimonie? Où sont les différences?

### Caractères continus par parcimonie

  1. Ouvrez une autre fenêtre avec le même arbre, et estimez les états
  ancestraux par parcimonie de la longueur des anguilles (caractère **Taille**).
  2. Sauvegardez une image de vos résultats pour pouvoir comparer plus tard avec
  les estimations dans **R** avec `File > Save Tree As PDF`.


---

## Estimation des états ancestraux dans R

**R** est un langage de programmation pour les statistiques. Il est moins
 intuitif qu'un logiciel avec menus, mais il a des avantages majeurs:

  - **reproductibilité**: le script garde la trace exacte de chaque étape;
  - **flexibilité**: des centaines de paquets offrent des méthodes
  comparatives récentes (modèles, tests, cartographie stochastique, etc.);
  - **figures** de qualité publication, entièrement personnalisables,
  contrairement aux figures pas très jolies de Mesquite...

### Premiers pas dans R

#### Console ou script?

On peut taper des commandes directement dans la **console** (la fenêtre où
 apparaît `>`), mais il vaut beaucoup mieux les écrire dans un **script**: un
 simple fichier texte avec l'extension `.R`.

  - Dans RStudio: `File > New File > R Script`. Pour exécuter une ligne (ou
  une sélection), placer le curseur dessus et faire **Ctrl+Entrée**
 (**Cmd+Entrée** sur Mac).
  - Sans RStudio: écrire le script dans un éditeur de texte et copier-coller
  les lignes dans R, ou exécuter tout le script depuis un terminal avec
  `Rscript mon_script.R`.
  - Les lignes de commande R ci-dessous peuvent être directement copier-coller
  dans la console R pour exécuter chaque étape, mais pour votre projet de
  session, il est impératif de conserver une copie de votre code dans votre
  journal de bord (ne pas travailler directement dans la console).

#### Les éléments de base du langage

Voici quelques lignes qui illustrent le fonctionnement de base de **R**, pour
 ceux qui ne connaissent pas déjà le langage. Exécuter les lignes suivantes une
 à une, et observer le résultat de chacune:
```r
# Tout ce qui suit un dièse est un commentaire: R l'ignore.

# Créer un objet avec la flèche d'assignation "<-"
x <- 5

# Taper le nom d'un objet affiche son contenu
x

# Exécuter une opération sur un objet affiche le résultat
x * 2

# Pour sauvegarder le résultat d'une opération sur un objet, il faut l'envoyer
# vers un autre objet.
y <- x * 2
y

# Un vecteur: une suite de valeurs du même type, créée avec c() pour
# "concaténation"
longueurs <- c(12.5, 30.1, 8.4)

# Une fonction fait des opérations sur des objets, par exemple:
mean(longueurs)
length(longueurs)

# Un vecteur de texte (chaînes de caractères, entre guillemets)
especes <- c("sp_A", "sp_B", "sp_C")

# Donner des noms aux éléments d'un vecteur: TRÈS utilisé en
# phylogénétique, car c'est ainsi qu'on associe une valeur à une feuille
names(longueurs) <- especes
longueurs

# Extraire un élément par son nom
longueurs["sp_B"]      

# Extraire un élément par sa position
longueurs[2]           

# Obtenir l'aide d'une fonction
?mean

```

Quelques règles à retenir:

  - R est **sensible à la casse**: `Arbre` et `arbre` sont deux objets 
  différents.
  - Les arguments d'une fonction sont séparés par des virgules et peuvent être
  nommés: `round(3.14159, digits = 2)`.
  - Un **tableau de données** (*data.frame*) est un tableau où chaque colonne
  est une variable. On extrait une colonne avec `$`: `tableau$colonne`.
  - Un **facteur** (*factor*) est un vecteur de catégories (par ex. les états
  d'un caractère discret).
  - Si une commande donne `Error in ...: could not find function`, c'est
  souvent qu'on a oublié de charger le paquet avec `library()`.

#### Le répertoire de travail

R lit et écrit les fichiers dans un **répertoire de travail**:
```r
# Afficher le répertoire de travail
getwd()

# Le changer (par exemple)
setwd("C:/Users/moi/TP_ancetres") 

```

Dans RStudio, on peut aussi changer le "working directory" avec 
 `Session > Set Working Directory > Choose Directory...`. Attention: R utilise
 des barres obliques penchées à droite `/`, même sous Windows.


### Charger et explorer les données phylogénétiques

Exécuter le code ci-dessous dans R:  
```r
## Charger le paquet (à refaire à chaque session)
library(phytools)

## Charger l'arbre et les données d'exemple fournies avec phytools
data(eel.tree)
data(eel.data)

## Les renommer en français pour la suite
arbre <- eel.tree
donnees <- eel.data

## Explorer l'arbre: un objet de classe "phylo"
arbre                     # indique la structure interne de l'objet
Ntip(arbre)               # nombre de feuilles
head(arbre$tip.label)     # les premiers noms de feuilles
is.rooted(arbre)          # l'arbre est-il enraciné?
is.ultrametric(arbre)     # toutes les feuilles à la même distance?

## Afficher l'arbre
plotTree(arbre, fsize = 0.5, ftype = "i")

## Explorer le tableau de données
head(donnees)             # les 6 premières lignes
str(donnees)              # structure: types de chaque colonne
table(donnees$feed_mode)  # nombre d'espèces par état
summary(donnees$Max_TL_cm)

```

**Étape essentielle**: vérifier que les noms des feuilles de l'arbre
 correspondent exactement aux noms des lignes du tableau. Une seule faute de
 frappe suffit à faire échouer l'analyse (ou pire, à la fausser):  
```r
## Noms présents dans l'arbre mais absents du tableau, et vice-versa.
## Les deux réponses doivent être character(0), c.-à-d. vides.
setdiff(arbre$tip.label, rownames(donnees))
setdiff(rownames(donnees), arbre$tip.label)

```

### Importer et exporter des arbres et des données de traits

Avec vos propres données, l'arbre et les traits seront dans des fichiers
 plutôt que dans un paquet R. Voici les fonctions les plus utiles (paquet
 **ape**, chargé automatiquement avec phytools).

#### Les arbres

```r
## Lire un arbre en format Newick (par ex. le .treefile d'IQ-Tree)
arbre <- read.tree("mon_arbre.tre")

## Lire un arbre en format NEXUS (par ex. le .con.tre de MrBayes,
## ou un arbre de TreeAnnotator)
arbre <- read.nexus("mon_arbre.nex")

## Si le fichier contient plusieurs arbres (par ex. des arbres bootstrap),
## on obtient un objet "multiPhylo"; on accède au premier avec [[1]]
arbres <- read.tree("mes_arbres.tre")
arbres[[1]]

## Écrire un arbre en Newick ou en NEXUS
write.tree(arbre, file = "anguilles.tre")
write.nexus(arbre, file = "anguilles.nex")

```

**Note**: `read.nexus()` lit la topologie et les longueurs de branches,
 mais **pas** les annotations des noeuds (probabilités postérieures, âges,
 intervalles HPD) produites par MrBayes ou BEAST. Pour les récupérer, on
 peut utiliser la fonction `read.beast()` du paquet **treeio** (Bioconductor).

#### Les données de traits

Le format le plus simple est un fichier **CSV** (valeurs séparées par des
 virgules), qu'on peut créer et modifier dans un tableur (Excel,
 LibreOffice). Une colonne contient les noms des taxons, **écrits exactement**
 **comme dans l'arbre**, et chaque autre colonne correspond à un caractère.
 Voici un exemple du format CSV:  
```
taxon,feed_mode,Max_TL_cm
Albula_vulpes,suction,104
Anguilla_anguilla,suction,50
...etc...
```

Créez un fichier CSV avec des données inventées comme dans le format ci-dessus,
 sauvegardez ces données sous le nom `anguilles_traits.csv` dans votre dossier
 de travail R, puis exécutez les commandes ci-dessous pour importer ces données
 dans R:  
```r
## Lire un tableau: row.names = 1 indique que la première colonne contient
## les noms des taxons; stringsAsFactors = TRUE convertit les colonnes de
## texte en facteurs (caractères discrets), head = TRUE indique que la première
## ligne contient les noms des colonnes
donnees <- read.csv("anguilles_traits.csv", row.names = 1,
                    stringsAsFactors = TRUE, head = TRUE)
str(donnees)

## Remettre les lignes dans le même ordre que les feuilles de l'arbre
donnees <- donnees[arbre$tip.label, ]

## Écrire le tableau de données (les noms de lignes deviennent la
## première colonne du fichier)
write.csv(donnees, file = "anguilles_traits_reorder.csv")

```

**Attention**: avec un tableur en français, le fichier est parfois
 sauvegardé avec des points-virgules (`;`) et des virgules décimales. Dans
 ce cas, utilisez `read.csv2()` plutôt que `read.csv()`.


### Caractère discret: ajuster un modèle de Markov (Mk)

Les fonctions de phytools ont besoin d'un **vecteur nommé**: une valeur par
 espèce, avec le nom de l'espèce comme nom de l'élément.
```r
## Extraire le mode d'alimentation et lui donner les noms d'espèces
alim <- setNames(donnees$feed_mode, rownames(donnees))
head(alim)

```

On ajuste ensuite deux modèles par maximum de vraisemblance:

  - **ER** (*equal rates*): un seul taux de transition, identique dans les
 deux directions (équivalent au modèle Mk1 de Mesquite);
  - **ARD** (*all rates different*): un taux différent pour chaque direction
 (équivalent au modèle AsymmMk de Mesquite).

```r
## Modèle à taux égaux
fit.er <- fitMk(arbre, alim, model = "ER")
fit.er

## Modèle à taux différents
fit.ard <- fitMk(arbre, alim, model = "ARD")
fit.ard

## Comparer les modèles: le plus petit AIC est le meilleur.
## Le "weight" est le poids d'Akaike (support relatif de chaque modèle).
anova(fit.er, fit.ard)

```

La sortie montre la **matrice Q** (les taux de transition entre états), la
 log-vraisemblance et le prior utilisé pour l'état de la racine.

- **Question**: Quel modèle est préféré selon l'AIC? Les log-vraisemblances
  sont-elles semblables à celles obtenues dans Mesquite?


### Caractère discret: états ancestraux (reconstruction marginale)

À l'étape précédente, les paramètres des modèles évolutifs ont été estimés. 
 Il faut maintenant utiliser ces modèles pour estimer les états ancestraux 
 les mieux supportés par les modèles. La probabilité marginale d'un état d'un
 caractère est la probabilité de retrouver cet état de caractère à ce noeud, peuimporte l'état retrouvé à tous les autres noeuds (donc la somme des 
 probabilités de toutes les histoires évolutives possibles où cet état est 
 observé à ce noeud). C'est ce qui est utilisé pour estimer l'état ancestral
 dans un modèle probabiliste.
 
Pour estimer les états ancestraux, exécuter dans R:  
```r
## Calculer les probabilités marginales de chaque état à chaque noeud,
## à partir du modèle choisi
anc <- ancr(fit.er)

# Le résultat est un tableau avec une ligne par noeud, une colonne par état
# La colonne qui a la plus grande probabilité marginale est l'état le plus
# vraisemblable à ce noeud, selon ce modèle.
head(anc$ace)    

## Représenter les probabilités sur l'arbre (diagrammes en tarte)
plot(anc,
     args.plotTree = list(fsize = 0.5),
     args.nodelabels = list(piecol = c("blue", "red"), cex = 0.4))
     
## Estimer en prenant en compte l'incertitude sur le modèle optimal

# Il est aussi possible d'utiliser le résultat de la fonction anova() pour
# estimer les états ancestraux. Dans ce cas, l'estimation de différents modèles
# est pris en compte, pondéré par le poids AIC de ce modèle. Ça évite de devoir
# manuellement décider du meilleur modèle.
anc.pondere <- ancr(anova(fit.er, fit.ard))

## Représenter les probabilités sur l'arbre (diagrammes en tarte)
plot(anc.pondere,
     args.plotTree = list(fsize = 0.5),
     args.nodelabels = list(piecol = c("blue", "red"), cex = 0.4))

```

### Caractère discret: cartographie stochastique

La reconstruction marginale donne des probabilités **aux noeuds**. La
 **cartographie stochastique** (*stochastic character mapping*) simule
 plutôt des **histoires complètes** du caractère le long des branches,
 compatibles avec les données et le modèle. En répétant la simulation
 plusieurs fois, on peut estimer le **nombre de changements** et le **temps**
 passé dans chaque état. Noter qu'il faut faire un grand nombre de ces 
 simulations si on veut bien prendre en compte la variabilité dans les histoires
 évolutives qui sont compatibles avec notre arbre et nos données.
 
Exécuter la cartographie stochastique dans R:  
```r
## Fixer le chiffre initial du générateur de chiffres aléatoires,
## pour que les résultats soient reproductibles
set.seed(123)

## Simuler 100 histoires sous le modèle ER
cartes <- make.simmap(arbre, alim, model = "ER", nsim = 100)

## Visualiser une seule des histoires simulées
choix <- 1
plot(cartes[[choix]], colors = setNames(c("blue", "red"),
     c("bite", "suction")), fsize = 0.5, ftype = "i")
     
## Visualiser une autre histoires simulée, différente de la précédente
## (changer la valeur de 1 à 100 de la variable "choix" pour afficher une autre)
choix <- 2
plot(cartes[[choix]], colors = setNames(c("blue", "red"),
     c("bite", "suction")), fsize = 0.5, ftype = "i")

## Résumer les 100 histoires, pour connaître nb moyens de changements 
## et temps dans chaque état
resume <- summary(cartes)
resume

## Nombre moyen de changements, total et par direction
colMeans(resume$count)

## Résumé graphique: probabilités aux noeuds sur l'ensemble des histoires
plot(resume, colors = setNames(c("blue", "red"), c("bite", "suction")),
     fsize = 0.5, ftype = "i")

## Carte de densité: probabilité postérieure d'être dans un état
## le long de chaque branche (peut prendre quelques secondes)
densityMap(cartes, fsize = 0.4, lwd = 3)
```

- **Question**: Combien de changements de mode d'alimentation sont estimés en
  moyenne? Comparez ce nombre au nombre de pas obtenu par parcimonie dans
  Mesquite. Pourquoi la parcimonie sous-estime-t-elle souvent le nombre de
  changements?
- **Question**: Les changements sont-ils plus fréquents de la morsure vers
  la succion, ou l'inverse?

### Exporter les résultats d'estimation d'états ancestraux

Exécuter dans R:  
```r
## Sauvegarder les probabilités des états ancestraux (une ligne par noeud)
## dans un fichier CSV, qu'on peut ouvrir dans un tableur
write.csv(anc$ace, file = "etats_ancestraux.csv")

## Ajouter un résultat comme étiquettes des noeuds internes, puis écrire
## l'arbre: ces étiquettes s'affichent dans FigTree (Node Labels)
arbre.annote <- arbre
arbre.annote$node.label <- round(anc$ace[, "suction"], 2)
write.tree(arbre.annote, file = "anguilles_prob_succion.tre")

## Sauvegarder n'importe quelle image produite par R sous la forme d'un pdf.
## Noter que largeur et longueur (width/height) sont en pouces (in).
pdf(file="arbre.annote.pdf", height=11, width=8.5)
plot(resume, colors = setNames(c("blue", "red"), c("bite", "suction")),
     fsize = 0.5, ftype = "i")
dev.off()

```


### Caractère continu: mouvement brownien

Pour les caractères continus, le modèle le plus simple est le **mouvement**
 **brownien** (*Brownian motion*): le caractère varie au hasard le long des
 branches, avec une variance proportionnelle au temps écoulé.

On transforme d'abord la longueur en **logarithme**. Les tailles varient
 souvent de façon multiplicative (une même variation relative chez une petite
 ou une grande espèce), et le log rend la distribution plus symétrique, ce
 qui convient mieux au modèle brownien.
 
Exécuter dans R:  
```r
## Vecteur nommé du log de la longueur maximale
taille <- setNames(log(donnees$Max_TL_cm), rownames(donnees))

## Comparer la distribution des valeurs brutes vs log
par(mfrow=c(1,2), mar=c(2,2,2,2))
hist(donnees$Max_TL_cm)
hist(taille)
par(mfrow=c(1,1))

## Estimation des états ancestraux par maximum de vraisemblance,
## avec intervalles de confiance à 95 %
anc.taille <- fastAnc(arbre, taille, CI = TRUE)
head(anc.taille$ace)     # valeurs estimées aux noeuds (en log)
head(anc.taille$CI95)    # bornes inférieure et supérieure

## Valeur à la racine, retransformée en cm avec exp()
exp(anc.taille$ace["62"])
exp(anc.taille$CI95["62", ])

## Carte de couleurs du caractère le long de l'arbre
contMap(arbre, taille, fsize = c(0.5, 0.8), lwd = 3)

## Phénogramme ("traitgram"): l'arbre projeté dans l'espace du caractère
phenogram(arbre, taille, fsize = 0.5, spread.labels = TRUE)

```

- **Question**: Comparez la valeur à la racine obtenue avec `fastAnc()` à
 celle obtenue par parcimonie des changements au carré dans Mesquite. Que
 constatez-vous?
- **Question**: Que pensez-vous de la largeur des intervalles de confiance
 près de la racine? Qu'est-ce que cela implique pour l'interprétation des
 estimations de caractères continus?

### le signal phylogénétique

Le lambda (λ) de Pagel (1999) et le K de Blomberg (Blomberg et al. 2003)
 mesurent tous deux le signal phylogénétique, c’est-à-dire à quel point des
 espèces apparentées se ressemblent pour un caractère, en prenant le mouvement
 brownien comme référence. 
 
Le λ est un paramètre qui multiplie les longueurs des branches internes de
 l’arbre (donc la covariance attendue entre les espèces) et qui est estimé par
 maximum de vraisemblance, généralement entre 0 et 1. Une valeur de 0 indique
 que le caractère est indépendant de la phylogénie (comme si l’arbre était une
 étoile), et une valeur de 1 indique qu’il varie exactement comme prévu par le
 mouvement brownien; sa significativité se teste par un rapport de vraisemblance
 contre λ = 0. 
 
Le K compare plutôt la variance observée du caractère à celle attendue sous le
 mouvement brownien: K = 1 correspond au mouvement brownien, K < 1 indique que
 les espèces apparentées se ressemblent moins que prévu (par ex. à cause de
 convergence, d’adaptation rapide ou d’erreur de mesure), et K > 1 qu’elles se
 ressemblent davantage que prévu (conservatisme phylogénétique), sa
 signification étant testée en permutant aléatoirement les valeurs du caractère
 entre les feuilles de l’arbre.
 
En pratique, λ indique surtout s’il y a du signal phylogénétique, alors que K
 permet en plus de comparer l’intensité du signal entre caractères ou entre
 groupes.

Pour obtenir les valeurs de ces deux variables dans R, exécuter ceci:  
```r
## Lambda de Pagel: 0 = aucun signal, 1 = conforme au brownien
phylosig(arbre, taille, method = "lambda", test = TRUE)

## K de Blomberg: test par permutations
phylosig(arbre, taille, method = "K", test = TRUE, nsim = 1000)
```

- **Question**: Comment interprétez-vous l'évolution de la taille chez les
  anguilles, maintenant que vous connaissez la valeur et la significativement du
  lambda de Pagel et du K de Blomberg?

### Sauvegarder les figures

Rappel: pour sauvegarder des figures: dans RStudio, le bouton `Export` du
 panneau des figures permet de sauvegarder la figure affichée. Pour garder une
 trace dans le script, on peut aussi ouvrir un fichier PDF avec `pdf()`,
 dessiner avec n'importe quelle fonction de R qui produit des figures, comme
 `plot()` ou `contMap()`, puis fermer la connection au fichier pdf avec
 `dev.off()`. Il existe aussi des fonctions pour produire d'autres types de
 figures (`jpeg()`, `png()`, etc.):  
```r
pdf("contmap_taille.pdf", width = 8, height = 10)
contMap(arbre, taille, fsize = c(0.5, 0.8), lwd = 3)
dev.off()     # ne pas oublier: c'est ce qui écrit le fichier
```

### Description des principales fonctions utilisées

  1. **`library(phytools)`** : Charge le paquet phytools (et ape) dans la
  session de R.

  2. **`data()`** : Charge un jeu de données fourni avec un paquet.

  3. **`setNames(valeurs, noms)`** : Crée un vecteur nommé; c'est le format
  attendu par la plupart des fonctions de phytools.

  4. **`setdiff(a, b)`** : Retourne les éléments de `a` absents de `b`.
  Utilisé pour vérifier la correspondance entre l'arbre et les données.

  5. **`fitMk(arbre, x, model)`** : Ajuste un modèle de Markov à `k` états
  par maximum de vraisemblance. `model = "ER"` (taux égaux), `"SYM"` (taux
  symétriques, pour 3 états et plus) ou `"ARD"` (tous les taux différents).

  6. **`anova(modele1, modele2)`** : Compare des modèles ajustés (AIC et
  poids d'Akaike).

  7. **`ancr(modele)`** : Calcule les probabilités marginales des états
  ancestraux à chaque noeud.

  8. **`make.simmap(arbre, x, model, nsim)`** : Cartographie stochastique;
  simule `nsim` histoires du caractère sur l'arbre.

  9. **`summary()`** et **`densityMap()`** : Résument les histoires
  simulées (nombre de changements, probabilités le long des branches).

  10. **`fastAnc(arbre, x, CI = TRUE)`** : Estime les états ancestraux d'un
  caractère continu sous le mouvement brownien, avec intervalles de confiance.

  11. **`contMap()`** et **`phenogram()`** : Visualisent un caractère continu
  sur l'arbre.

  12. **`phylosig()`** : Mesure et teste le signal phylogénétique (lambda de
  Pagel ou K de Blomberg).

### Appliquer à vos propres données

Pour utiliser l'arbre obtenu dans un tutoriel précédent (par ex.
 `test.treefile` d'IQ-Tree) avec un tableau de caractères préparé dans un
 tableur et sauvegardé en format CSV:
```r
library(phytools)

## Lire l'arbre (format Newick)
arbre <- read.tree("test.treefile")

## Enraciner l'arbre avec un groupe externe (remplacer par votre taxon)
arbre <- root(arbre, outgroup = "Nom_du_groupe_externe",
              resolve.root = TRUE)

## Lire le tableau: une colonne "taxon", puis une colonne par caractère.
## Les noms dans "taxon" doivent être identiques aux noms de l'arbre.
donnees <- read.csv("caracteres.csv", row.names = "taxon")

## Retirer de l'arbre les taxons sans données (s'il y a lieu)
arbre <- keep.tip(arbre, rownames(donnees))

## Vérifier la correspondance, puis continuer comme plus haut
setdiff(arbre$tip.label, rownames(donnees))
setdiff(rownames(donnees), arbre$tip.label)
```

Un arbre de maximum de vraisemblance n'est pas ultramétrique: ses longueurs de
 branches sont en substitutions par site plutôt qu'en temps. Les analyses
 ci-dessus fonctionnent quand même, mais les taux estimés sont alors
 « par substitution » plutôt que « par unité de temps ». Un arbre daté
 (voir tutoriel de datation) est préférable.

---

## Questions et exercices

1. **Choix de la méthode**:
   - Comparez les états reconstruits à la racine par parcimonie (Mesquite),
   par vraisemblance (Mesquite et R) et par cartographie stochastique (R).
   Sont-ils cohérents? Lequel de ces résultats rapporteriez-vous dans un
   article, et pourquoi?

2. **Choix du modèle**:
   - Ajustez aussi dans R un modèle où une seule direction est permise (par
   ex. la morsure peut devenir la succion, mais jamais l'inverse) avec
   `fitMk(arbre, alim, model = matrix(c(0, 1, 0, 0), 2, 2, byrow = TRUE))`.
   Comparez-le aux modèles ER et ARD avec `anova()`. Que concluez-vous sur
   l'irréversibilité de ce caractère?

3. **Sensibilité aux hypothèses**:
   - Refaites `make.simmap()` avec le modèle `"ARD"`. Le nombre moyen de
   changements change-t-il?
   - Refaites `fastAnc()` sur la longueur **non transformée**
   (`donnees$Max_TL_cm`). Comment les valeurs à la racine et les intervalles
   de confiance changent-ils?

4. **Incertitude sur l'arbre**:
   - Toutes nos analyses supposent que l'arbre est connu sans erreur. Dans
   Mesquite, `Analysis:Tree > Trace Character Over Trees` permet de résumer la
   reconstruction sur un ensemble d'arbres (par ex. les arbres bootstrap ou
   un échantillon de la distribution postérieure). Expliquez en quoi cela
   améliore l'interprétation.

5. **Interprétation biologique**:
   - En quelques phrases, répondez à la question de départ: quel était
   probablement le mode d'alimentation de l'ancêtre commun des anguilles, et
   combien de fois ce mode a-t-il changé? Quel est votre degré de confiance?

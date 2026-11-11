# Biogéographie historique avec BioGeoBEARS

Ce tutoriel fait suite aux tutoriels sur la
 [reconstruction des états ancestraux](Analyses_apres_phylo/01--Etats_ancestraux.md)
 et sur les
 [modèles d'évolution des caractères](Analyses_apres_phylo/02--Modeles_evolutifs.md).
 Comme eux, il se fait entièrement dans **R**, sur votre ordinateur personnel.

La **biogéographie historique** cherche à estimer l'histoire de la
 distribution géographique d'un groupe: où vivaient les ancêtres, et par quels
 événements (dispersions, extinctions locales, vicariance, spéciation par
 effet fondateur) les espèces actuelles ont-elles atteint leur aire de
 répartition? C'est une forme d'estimation des états ancestraux, mais
 avec des caractères particuliers: les **aires de répartition**, où une
 espèce peut occuper plusieurs régions à la fois.

On utilisera le paquet **BioGeoBEARS** (Matzke 2013, 2014), qui permet
 d'ajuster et de comparer plusieurs modèles de biogéographie historique par
 maximum de vraisemblance.

---

## Le jeu de données

On utilise l'exemple des ***Psychotria*** (Rubiaceae) des **îles
 hawaïennes** (Ree et Smith 2008), fourni avec BioGeoBEARS:

- un arbre daté de **19 taxons**, dont la racine a environ **5,2 Ma**;
- leur présence dans **4 aires**: **K** (Kauai), **O** (Oahu), **M**
 (Maui Nui) et **H** (île d'Hawaï).

Ces îles se sont formées l'une après l'autre: Kauai est la plus vieille
 (environ 5 Ma) et l'île d'Hawaï la plus jeune (moins de 1 Ma).

- **Question**: Avant de commencer, quelle aire vous attendez-vous à trouver
 à la racine de l'arbre?

---

## Installation

BioGeoBEARS n'est **pas** installé à partir du CRAN, mais à partir de GitHub
 (la version du CRAN est très ancienne). Deux paquets dont il dépend,
 **rexpokit** et **cladoRcpp**, s'installent par contre à partir du CRAN.
 Les instructions à jour sont sur la page du projet:
 <https://github.com/nmatzke/BioGeoBEARS>.

```r
## À faire une seule fois
install.packages(c("rexpokit", "cladoRcpp", "remotes"))

## Installer BioGeoBEARS à partir de GitHub
remotes::install_version("MultinomialCI", version = "1.2")
remotes::install_github("nmatzke/BioGeoBEARS")
```

Si l'installation de `rexpokit` ou `cladoRcpp` échoue (ces paquets contiennent
 du code compilé), consultez la page GitHub de BioGeoBEARS: elle indique où
 trouver des versions précompilées.

Chargez ensuite les paquets:
```r
library(ape)
library(BioGeoBEARS)
```

**Note**: BioGeoBEARS est un paquet très complet, mais son interface est
 assez « brute »: on modifie directement des tableaux de paramètres, et les
 fonctions affichent beaucoup de messages à l'écran. Ne vous laissez pas
 impressionner par la quantité de texte produite: ce sont des messages
 d'information, pas des erreurs.

---

## Les fichiers d'entrée

BioGeoBEARS a besoin de deux fichiers: un **arbre daté** (format Newick) et
 une **matrice des aires** (format PHYLIP utilisé par le logiciel Lagrange).
 On copie les fichiers d'exemple dans le répertoire de travail:
```r
## Choisir le répertoire de travail (adapter le chemin)
setwd("C:/Users/Etienne/Downloads/biogeo")

## Dossier où sont installés les fichiers d'exemple de BioGeoBEARS
extdata <- np(system.file("extdata", package = "BioGeoBEARS"))

## Copier l'arbre et la matrice des aires dans le répertoire de travail
file.copy(file.path(extdata, "Psychotria_5.2.newick"), "Psychotria.newick")
file.copy(file.path(extdata, "Psychotria_geog.data"),
          "Psychotria_geog.data")
```
La fonction `np()` de BioGeoBEARS normalise simplement le chemin d'accès,
 quel que soit le système d'exploitation.

Si vous voulez utiliser votre propre arbre et votre propre matrice de
 répartition biogéographique, vous pouvez sauter cette étape et simplement les
 placer dans votre répertoire de travail.

### L'arbre

Exécuter dans R, pour charger l'arbre de *Psychotria*:  
```r
arbre <- read.tree("Psychotria.newick")
arbre
plot(arbre, cex = 0.7)
axisPhylo()          # échelle de temps en Ma
is.ultrametric(arbre)
```
L'arbre doit être **daté** (ultramétrique, longueurs de branches en unités de
 temps, par ex. obtenu avec BEAST), car les taux de dispersion et
 d'extinction sont exprimés **par unité de temps**.

### La matrice des aires

Ouvrez `Psychotria_geog.data` dans un éditeur de texte:
```
19	4 (K O M H)
P_mariniana_Kokee2	1000
P_mariniana_Oahu	0100
P_mariniana_MauiNui	0010
P_hawaiiensis_Makaopuhi	0001
...
```
- La première ligne donne le **nombre de taxons**, le **nombre d'aires**, et
 le **nom des aires** entre parenthèses (des noms courts, idéalement une
 seule lettre).
- Chaque ligne suivante donne le nom d'un taxon (**exactement** comme dans
 l'arbre), une tabulation, puis une suite de 0 et de 1: **1** si le taxon
 est présent dans l'aire, **0** sinon, dans l'ordre des aires de la
 première ligne. Par exemple, `1100` signifierait une présence à Kauai (K) et à
 Oahu (O).

```r
## Lire la matrice des aires dans R
aires <- getranges_from_LagrangePHYLIP("Psychotria_geog.data")
aires@df
```

Pour vos propres données, vous pouvez créer ce fichier dans un éditeur de
 texte ou un tableur (en le sauvegardant en texte délimité par des
 tabulations).

---

## Les deux modèles biogéographiques comparés ici

Le modèle **DEC** (*Dispersal-Extinction-Cladogenesis*; Ree et Smith 2008)
 a deux paramètres, qui agissent **le long des branches**:
- **d**: taux de **dispersion** (l'aire s'agrandit, par ex. K → KO);
- **e**: taux d'**extinction locale** (l'aire rétrécit, par ex. KO → K).

Au moment de chaque spéciation, l'aire de l'ancêtre est répartie entre les
 deux espèces filles selon des règles fixes (par ex. une fille hérite de K,
 l'autre de O si l'ancêtre occupait KO).

Le modèle **DEC+J** (Matzke 2014) ajoute un troisième paramètre:
- **j**: la **spéciation par effet fondateur**. Au moment de la spéciation,
 une espèce fille « saute » dans une nouvelle aire, où l'ancêtre n'était pas
 (par ex. un ancêtre à Kauai donne une fille à Kauai et une fille à Oahu).

Selon Ree et Sanmartín (2018), le modèle DEC+J est mal conçu et ne devrait pas
 être comparé à DEC par l'AIC. Selon Matzke (2022), la comparaison est valide.
 On y reviendra dans les questions à la fin du tutoriel.

---

## Ajuster le modèle DEC

Une analyse BioGeoBEARS se prépare en trois étapes: créer un objet
 d'analyse, y indiquer les fichiers et les options, puis lire les données.
```r
## Créer un objet d'analyse (par défaut, c'est le modèle DEC)
dec <- define_BioGeoBEARS_run()

## Indiquer les fichiers
fichier.arbre <- file.path(getwd(), "Psychotria.newick")
fichier.aires <- file.path(getwd(), "Psychotria_geog.data")

dec$trfn <- fichier.arbre
dec$geogfn <- fichier.aires

## Nombre maximal d'aires qu'une espèce peut occuper à la fois
dec$max_range_size <- 4

## Lire l'arbre et les aires
dec <- readfiles_BioGeoBEARS_run(dec)

## Ajuster le modèle par maximum de vraisemblance (quelques secondes)
res.dec <- bears_optim_run(dec)
```
BioGeoBEARS affiche beaucoup de texte et quelques avertissements
 (*warnings*): ce sont des messages d'information, pas des erreurs.

---

## Ajuster le modèle DEC+J

Les paramètres du modèle sont dans un tableau. On copie l'analyse DEC, puis
 on modifie seulement la ligne du paramètre j:
```r
## Copier l'analyse DEC
decj <- dec

## Extraire le tableau des paramètres (une ligne par paramètre)
p <- decj$BioGeoBEARS_model_object@params_table
p[c("d", "e", "j"), c("type", "init", "est", "desc")]

## changer j de "fixed" (fixé à 0) à "free" (estimé), avec une petite
## valeur de départ
p["j", "type"] <- "free"
p["j", "init"] <- 0.0001

## Pour aider l'optimisation, partir des valeurs de d et e trouvées par DEC
p["d", "init"] <- res.dec$outputs@params_table["d", "est"]
p["e", "init"] <- res.dec$outputs@params_table["e", "est"]

## Remettre le tableau modifié dans l'analyse, puis l'ajuster
decj$BioGeoBEARS_model_object@params_table <- p
res.decj <- bears_optim_run(decj)
```

---

## Comparer les modèles

```r
## Paramètres estimés par chaque modèle
res.dec$outputs@params_table[c("d", "e", "j"), "est"]
res.decj$outputs@params_table[c("d", "e", "j"), "est"]

## Log-vraisemblance, nombre de paramètres (k) et AIC
lnL <- c(DEC = res.dec$total_loglikelihood,
         DEC_J = res.decj$total_loglikelihood)
k <- c(DEC = 2, DEC_J = 3)
data.frame(lnL, k, AIC = -2 * lnL + 2 * k)

## Test du rapport de vraisemblance (DEC est DEC+J avec j = 0)
lrt <- 2 * (lnL["DEC_J"] - lnL["DEC"])
pchisq(lrt, df = 1, lower.tail = FALSE)
```

- **Question**: Quel modèle est le mieux supporté?
- **Question**: Dans DEC+J, que valent d et e? Qu'est-ce que cela signifie
  sur la façon dont les *Psychotria* ont colonisé les îles?

---

## Visualiser les aires ancestrales

Chaque noeud est représenté par un diagramme en tarte: les couleurs sont les
 aires possibles, et la taille des parts est leur probabilité.
```r
plot_BioGeoBEARS_results(res.dec, analysis_titletxt = "DEC",
                         plotwhat = "pie", tipcex = 0.7, statecex = 0.7,
                         tr = arbre, tipranges = aires)

plot_BioGeoBEARS_results(res.decj, analysis_titletxt = "DEC+J",
                         addl_params = list("j"), plotwhat = "pie",
                         tipcex = 0.7, statecex = 0.7,
                         tr = arbre, tipranges = aires)
```
Avec `plotwhat = "text"` plutôt que `"pie"`, la figure affiche seulement
 l'aire la plus probable à chaque noeud. Pour sauvegarder une figure, placez
 la commande entre `pdf("fichier.pdf")` et `dev.off()`, comme dans le
 tutoriel précédent.

- **Question**: Quelle aire est reconstruite à la racine par chaque modèle?
  Est-ce cohérent avec l'âge des îles?
- **Question**: DEC reconstruit souvent des ancêtres répartis sur plusieurs
  îles à la fois (par ex. KO). Est-ce plausible pour des plantes d'îles
  séparées par la mer? Comment DEC+J explique-t-il plutôt les mêmes données?

---

## Questions de synthèse

1. Ree et Sanmartín (2018) reprochent notamment à DEC+J de pouvoir expliquer
  toute l'histoire par des sauts au moment des spéciations, sans aucune
  dispersion ni extinction le long des branches. Vos résultats illustrent-ils
  cette critique?
2. Ces analyses supposent que l'arbre daté est connu sans erreur. Comment
  pourriez-vous tenir compte de l'incertitude sur l'arbre (par ex. avec les
  arbres échantillonnés par BEAST)?

---

## Pour aller plus loin

BioGeoBEARS offre beaucoup d'autres possibilités, décrites dans les scripts
 d'exemple de la [page GitHub](https://github.com/nmatzke/BioGeoBEARS) et sur
 le [site PhyloWiki](http://phylo.wikidot.com/biogeobears):
- d'autres modèles: **DIVALIKE** (inspiré de DIVA; plus de vicariance) et
  **BAYAREALIKE** (pas de vicariance), avec ou sans j;
- la **cartographie stochastique biogéographique** (BSM), pour compter le
  nombre de dispersions entre chaque paire d'aires;
- la **stratification temporelle**, pour tenir compte, par exemple, du fait
  qu'une île n'existait pas avant une certaine date.

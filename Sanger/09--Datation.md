# Estimation des temps de divergence

Ce tutoriel vous montrera comment estimer les **temps de divergence** (l'âge
 des noeuds) d'une phylogénie à l'aide de fossiles, avec les programmes
 **BEAST 2** et **MrBayes**. Il suppose que vous avez déjà fait les tutoriels
 sur l'[analyse Bayesienne](Sanger/07--Analyse_Bayesienne.md) (MrBayes,
 Tracer) et sur l'[estimation d'arbres d'espèces](Sanger/08--Arbre_despeces.md)
 (BEAUti, BEAST 2, TreeAnnotator, FigTree).

Il existe deux grandes façons d'utiliser les fossiles pour calibrer un arbre:

1. **Calibration des noeuds** (*node dating*): on attribue chaque fossile à
 un clade d'espèces actuelles, puis on utilise son âge pour imposer une
 distribution *a priori* sur l'âge du noeud correspondant. Les fossiles
 eux-mêmes ne font pas partie de l'arbre.
2. **Fossiles comme feuilles** (*tip dating*, ou *total-evidence dating*):
 les fossiles sont inclus directement dans l'analyse comme des feuilles de
 l'arbre, avec leurs caractères morphologiques et leur âge. Leur position dans
 l'arbre est estimée en même temps que tout le reste, sous un modèle nommé
 *fossilized birth-death* (FBD). Cela nécessite de coder des caractères
 morphologiques à la fois sur les fossiles, et aussi sur les espèces vivantes
 qu'on a séquencé.

Chaque partie est présentée dans **BEAST 2** (interface BEAUti) puis dans
 **MrBayes** (ligne de commande).

---

## Quelques notions essentielles

- **Le problème de base**: une séquence d'ADN nous renseigne sur le nombre de
  substitutions le long de chaque branche, c.-à-d. sur le produit
  *taux × temps*. On ne peut pas séparer le taux du temps sans information
  externe: c'est le rôle des calibrations.
- **Horloge moléculaire**: une horloge **stricte** suppose un taux de
  substitution identique sur toutes les branches. Une horloge **relâchée**
  (*relaxed clock*) permet au taux de varier d'une branche à l'autre, selon
  une distribution (log-normale dans BEAST, gamma dans MrBayes).
- **A priori sur l'arbre** (*tree prior*): un modèle de diversification
  (Yule, birth-death sans fossiles, ou FBD) qui décrit la distribution attendue
  des temps de divergence. Il interagit avec les calibrations.
- **Les distributions de calibration** les plus utilisées:
    - **Uniforme** (« plate »): toutes les valeurs entre un âge minimal et un
    âge maximal sont également probables. Les bornes sont **strictes**:
    l'âge ne peut jamais sortir de l'intervalle.
    - **Log-normale décalée** (*offset lognormal*): l'âge minimal (le décalage,
    *offset*) correspond à l'âge du plus vieux fossile du clade. Le noeud doit
    être plus vieux que ce minimum, avec une probabilité qui diminue
    graduellement vers les âges plus anciens (une « queue » souple).
    - **Exponentielle décalée** et **gamma décalée**: même logique que la
    log-normale, avec des formes un peu différentes.
- **L'a priori effectif n'est pas l'a priori spécifié**: les calibrations
  interagissent entre elles et avec l'a priori sur l'arbre (par ex. un noeud
  ne peut pas être plus vieux que son parent). Il faut **toujours** faire une
  analyse qui échantillonne seulement l'a priori (*sample from prior*) pour
  voir quelles distributions sont réellement utilisées.

---

## Le jeu de données

On utilisera le jeu de données des **Osmundaceae** (fougères royales) de
 [Grimm et al. (2015)](https://doi.org/10.1093/sysbio/syu108). Ces auteurs ont
 justement comparé la calibration des noeuds dans BEAST et la datation avec
 les fossiles comme feuilles dans MrBayes. Les fichiers proviennent du
 [tutoriel *Total-Evidence* de Taming the BEAST](https://taming-the-beast.org/tutorials/Total-Evidence-Tutorial/)
 (B.J. Allen et J. Barido-Sottani), qu'on suivra dans la partie 2.

Le jeu de données contient 33 unités:
- **15 espèces vivantes** de *Leptopteris*, *Todea* et *Osmunda*
  (13 d'entre elles ont des séquences d'ADN);
- **18 fossiles** (rhizomes perminéralisés) datés du Jurassique au Néogène;
- **8628 sites** d'ADN plastidique (7 régions) pour les espèces actuelles;
- **25 caractères morphologiques** du rhizome, pour presque toutes les unités.

Le nom de chaque unité se termine par son âge approximatif en millions
 d'années (Ma), par ex. `Todea_tidwellii_135`. Les espèces actuelles ont un
 âge de 0.

Les fichiers à télécharger sont dans le dossier "Sanger/fichiers" du Github du
 cours:  
| Fichier | Contenu | Utilisé dans |
|---|---|---|
| `Osmundaceae_actuelles.nex` | ADN des 13 espèces actuelles séquencées | Partie 1 (BEAST et MrBayes) |
| `Osmundaceae_dna.nex` | ADN des 33 unités (`?` pour les fossiles) | Partie 2 (BEAST) |
| `Osmundaceae_morph.nex` | Morphologie des 33 unités | Partie 2 (BEAST) |
| `Osmundaceae_TED.nex` | ADN + morphologie dans une seule matrice | Partie 2 (MrBayes) |

Les fichiers `Osmundaceae_actuelles.nex` et `Osmundaceae_TED.nex` ont été
 créés à partir des deux fichiers de Taming the BEAST: le premier ne garde que
 les espèces actuelles avec de l'ADN, le second combine l'ADN et la
 morphologie en une matrice mixte, comme l'exige MrBayes.

- **Question**: Ouvrez `Osmundaceae_dna.nex` et `Osmundaceae_morph.nex` dans
 un éditeur de texte. Pourquoi les fossiles ont-ils des séquences d'ADN
 entièrement composées de `?`? Que signifie un état codé `{12}` dans la
 matrice morphologique?

---

## Partie 1 – Calibration des noeuds

### Les calibrations utilisées

Dans cette partie, on n'utilise que les 13 espèces actuelles séquencées.
 Dans les Osmundaceae, la racine sépare le clade *Todea*/*Leptopteris* du
 clade *Osmunda* au sens large (incluant *O. cinnamomea*, parfois placé dans
 le genre *Osmundastrum*). Pour visualiser l'arbre, voir l'étude de Grimm
 et al. (2015).
 
On place deux calibrations sur les noeuds ancestraux de l'arbre:

| Noeud | Fossile utilisé | Distribution | Paramètres (Ma) |
|---|---|---|---|
| Racine (toutes les espèces) | *Todea tidwellii* (129–140 Ma), sur la lignée de *Todea*/*Leptopteris* | **Uniforme** (plate) | min. 129, max. 360 |
| Ancêtre commun d'*Osmunda* s.l. | *O. cinnamomea* du Crétacé du Canada (72–83 Ma) | **Log-normale décalée** | décalage 72, moyenne 150, écart-type 40 |

Justification:
- *Todea tidwellii* appartient à la lignée de *Todea*/*Leptopteris*, donc la
 séparation de cette lignée et de celle d'*Osmunda* (la racine) est au moins
 aussi vieille que ce fossile: **129 Ma minimum**. L'âge maximal de 360 Ma
 (début du Carbonifère) est une borne volontairement conservatrice, plus
 vieille que les plus anciens fossiles connus de l'ordre.
- Des rhizomes du Crétacé supérieur du Canada sont attribués à
 *O. cinnamomea* elle-même. La lignée d'*O. cinnamomea* existait donc il y a
 au moins **72 Ma**, et sa séparation d'avec les autres *Osmunda* est au
 moins aussi vieille.
- Le fossile le plus ancien de *O. pulchella*, daté d'environ **185 Ma**, n'est
  pas utilisé ici car sa position phylogénétique n'est pas certaine.

> **Note**: Ces calibrations sont **simplifiées pour l'enseignement**. Elles
> s'inspirent des fossiles du jeu de données, mais ne reproduisent pas
> exactement celles de Grimm et al. (2015). Dans une vraie étude, le choix et
> la justification des calibrations est l'étape la plus importante de toute
> l'analyse (voir Parham et al. 2012, *Systematic Biology* 61: 346-359).

#### Traduire une calibration log-normale

MrBayes définit une log-normale décalée par son **âge minimal**, sa
 **moyenne** et son **écart-type**, en millions d'années:
 `offsetlognormal(72, 150, 40)`. BEAUti la définit plutôt par les paramètres
 **M** et **S**, qui sont la moyenne et l'écart-type du **logarithme** de
 (âge − décalage). La conversion est:

```
m = moyenne - décalage = 150 - 72 = 78
s = écart-type        = 40
S = racine( ln(1 + (s/m)^2) ) = 0.483
M = ln(m) - S^2 / 2           = 4.240
```

Avec ces valeurs, l'âge médian de la calibration est de 72 + e<sup>M</sup>
 ≈ 141 Ma, et 95 % de la probabilité se trouve entre ≈ 99 et ≈ 251 Ma.

- **Question**: En quoi une calibration log-normale décalée est-elle plus
 réaliste qu'une calibration uniforme pour représenter l'information
 fournie par un fossile?

### Calibration des noeuds dans BEAST 2

#### Préparer l'analyse dans BEAUti

1. Ouvrez **BEAUti** avec le modèle par défaut (`File > Template > Standard`).
  Si vous venez de faire le tutoriel sur \*BEAST, assurez-vous de ne plus être
  dans le modèle StarBeast3.

2. **Importer les données**: `File > Import Alignment`, puis choisir
  `Osmundaceae_actuelles.nex`. L'onglet *Partitions* devrait indiquer
  13 taxons et 8628 sites.

3. **Onglet *Site Model***: choisir le modèle **HKY**, mettre
  *Gamma Category Count* à **4**, et cocher *estimate* pour *Kappa* et
  *Frequencies* (ou choisir *Estimated* dans le menu *Frequencies*).

4. **Onglet *Clock Model***: choisir **Relaxed Clock Log Normal**. Vérifiez
  que la case *estimate* du taux d'horloge est cochée. Si elle est grisée,
  décochez `Mode > Automatic set clock rate` dans le menu.

5. **Ajouter le prior sur la topologie de l'arbre***:
    - Pour **Tree.t**, choisir **Birth Death Model**.

6. **Ajouter la calibration log-normale** (*Osmunda* s.l.):
    - Cliquer sur **+ Add Prior** au bas de l'onglet *Priors*. Si une fenêtre
   demande le type d'a priori, choisir **MRCA prior**.
    - Dans *Taxon set label*, écrire `OsmundaSL`. Sélectionner les 8 espèces
   d'*Osmunda* dans la liste de gauche et les déplacer à droite avec `>>`.
   Cliquer sur **OK**.
    - Cocher la case **monophyletic**: la calibration n'a de sens que si le
   clade existe dans l'arbre.
    - Dans le menu de distribution, choisir **Log Normal**. Ouvrir les options
   avec le triangle à gauche, puis entrer **M = 4.24**, **S = 0.483**,
   **Offset = 72**. Laisser *Mean In Real Space* **décoché** (M est en
   échelle logarithmique).
    - Le petit graphique à droite montre la distribution: vérifiez que la
   densité commence à 72 Ma et que sa médiane est d'environ 141 Ma.

7. **Ajouter la calibration uniforme** (racine):
    - Cliquer de nouveau sur **+ Add Prior** et choisir **MRCA prior**.
    - Nommer le groupe `racine` et y mettre **les 13 espèces**. Cliquer sur
   **OK**. Pas besoin de cocher *monophyletic*: tous les taxons forment
   forcément un clade.
    - Choisir la distribution **Uniform**, puis **Lower = 129** et
   **Upper = 360**.


8. **Ajouter une probabilité a priori sur le taux de mutation**: Comme les
  calibrations sont en millions d'années avant le présent (Ma), il faut aussi
  que l'analyse prenne en compte et estime le taux de mutations pour être
  capable de relier les données génétiques de l'ADN à l'âge de divergence des
  noeuds de l'arbre. Pour ce faire, on doit placer une probabilité a priori sur
  le taux d'évolution qui a du sens basé sur les études précédentes.
    - Pour le taux moyen de l'horloge moléculaire (`ucldMean.c:...` ou
    `clockRate.c:...` selon la version), choisir une distribution **Log Normal**
    avec **M = -7** et **S = 1**. Cela correspond à un taux d'environ 0,001
    substitution/site/Ma, un ordre de grandeur raisonnable pour l'ADN
    plastidial. Les âges étant en Ma, le taux est exprimé par Ma. Pour 
    déterminer le taux à utiliser comme "prior" pour vos propres analyses, vous
    devez consulter la littérature sur les taux de mutation dans votre groupe
    (plantes, animaux, bactéries...) pour votre génome à l'étude (nucléaire, 
    mitochondrial...) et inscrire un "prior" qui a du sens selon ce qui a déjà
    été trouvé dans les études précédentes. 
    - Cliquer sur le petit bouton à droite de celui qui permet de sélectionner
    une distribution Log Normale pour le prior ("initial = [0.1] [0.0,Inf]").
    Changer la valeur de départ de la chaîne MCMC à la même valeur que le taux
    de mutation moyen que vous avez mis comme prior (0,001), de façon à ce que
    la chaîne débute avec une valeur probable.
    - **Attention!** Il n'est pas possible de placer ce prior sur le taux de
    l'horloge moléculaire/taux de mutation avant d'avoir créé une première
    calibration fossile, car ça ne sert à rien d'estimer le taux de mutation
    si l'arbre n'est pas calibré avec des fossiles.

8. **Onglet *MCMC***:
    - *Chain Length*: **10 000 000**.
    - Ouvrir *tracelog* et *treelog* et mettre *Log Every* à **1000** pour
   les deux.

9. **Sauvegarder le fichier**: `File > Save As` → `Osmundaceae_noeuds.xml`.

10. **Analyse de l'a priori seulement**:
    - Toujours dans l'onglet *MCMC*, cocher **Sample From Prior**.
    - Sauvegarder sous un autre nom: `File > Save As` →
    `Osmundaceae_noeuds_prior.xml`. Les fichiers de sortie de cette analyse
    porteront le même préfixe et n'écraseront donc pas ceux de l'analyse
    principale.

#### Lancer l'analyse

Ce jeu de données est assez petit pour être analysé sur votre ordinateur:
 ouvrez **BEAST**, choisissez `Osmundaceae_noeuds.xml`, puis **Run**. Faites
 de même avec `Osmundaceae_noeuds_prior.xml` (cette analyse est beaucoup plus
 rapide, puisqu'il n'y a pas de calcul de vraisemblance).

Vous pouvez aussi lancer les deux analyses sur le serveur de calcul, avec le
 même script SLURM que dans le
 [tutoriel sur les arbres d'espèces](Sanger/08--Arbre_despeces.md), en
 changeant seulement la variable `XML_INPUT`.

#### Analyser les résultats

1. **Tracer**: ouvrir `Osmundaceae_noeuds.log` et
 `Osmundaceae_noeuds_prior.log`.
    - Vérifiez la convergence (traces stationnaires, ESS > 200) comme dans le
    tutoriel sur MrBayes.
    - Les âges des noeuds calibrés apparaissent dans des colonnes comme
    `mrca.age(OsmundaSL)` et `mrca.age(racine)` (le nom exact peut varier
    légèrement). La hauteur de l'arbre (`TreeHeight`) correspond à l'âge de la
    racine.
    - Sélectionnez les deux fichiers en même temps, puis l'onglet
    *Marginal Density*, pour superposer la distribution **a priori**
    (*sample from prior*) et la distribution **a posteriori** de chaque âge.

2. **TreeAnnotator**: choisir un *burn-in* de 10 %, l'option
  **Maximum clade credibility tree** et **Median heights**. Fichier d'entrée:
  `Osmundaceae_noeuds.trees` (ou le fichier `.trees` produit par votre
  analyse). Fichier de sortie: `Osmundaceae_noeuds_MCC.tre`.

3. **FigTree**: ouvrir `Osmundaceae_noeuds_MCC.tre`.
    - Cocher *Node Bars* et choisir `height_95%_HPD` pour afficher les
    intervalles de crédibilité des âges.
    - Cocher *Scale Axis*, puis *Reverse axis* dans ses options, pour avoir
    une échelle de temps en Ma avant le présent.
    - Cocher *Node Labels* et choisir `height_median` pour afficher les âges.

- **Question**: L'a priori effectif sur l'âge d'*Osmunda* s.l. (analyse
  *sample from prior*) ressemble-t-il à la log-normale que vous avez
  spécifiée? Si non, pourquoi?
- **Question**: Pour quels noeuds la distribution a posteriori est-elle
  très différente de l'a priori? Qu'est-ce que cela indique sur
  l'information apportée par les séquences d'ADN?

---

### Calibration des noeuds dans MrBayes

Commencez par téléverser les fichiers de données `Osmundaceae_actuelles.nex` et
 `Osmundaceae_TED.nex` sur le serveur de calcul. Ensuite, naviguez vers le
 dossier où vous avez placé ces fichiers, lancez MrBayes avec la commande `mb`,
 puis exécuter les commandes ci-dessous.
 
**!!! Attention !!!** Dans certaines lignes de commande, MrBayes n'accepte pas
 plus qu'une ligne de commande à la fois. Même lorsque c'est le cas, il ne
 renvoi pas d'avertissement. En conséquence, lorsque vous exécutez les commandes
 ci-dessous, assurez-vous de copier et coller une seule ligne à la fois pour
 vous assurer que toutes les commandes seront bel et bien exécutées, même si
 un bloc de code contient plus d'une ligne de commande.

1. **Charger les données**: 
  ```MrBayes
  execute Osmundaceae_actuelles.nex;
  ```
  
2. **choisir le modèle de substitution** (HKY+G):
  ```MrBayes
  lset nst=2 rates=gamma;
  ```

3. **Définir le clade calibré et contraindre sa monophylie**. Dans MrBayes,
  une calibration ne peut être placée que sur un noeud défini par une
  contrainte:
  ```MrBayes
  constraint OsmundaSL = Osmunda_cinnamomea Osmunda_claytoniana Osmunda_banksiaefolia Osmunda_javanica Osmunda_vachellii Osmunda_japonica Osmunda_lancea Osmunda_regalis;
  prset topologypr = constraint(OsmundaSL);
  ```

4. **Demander un arbre ultramétrique** (horloge moléculaire) avec un a priori
  de naissance-mort. `sampleprob` est la proportion des espèces actuelles du
  groupe incluses dans l'analyse (13 sur environ 28):
  ```MrBayes
  prset brlenspr = clock:birthdeath;
  prset samplestrat = random sampleprob = 0.46;
  ```

5. **Calibrer la racine** (uniforme) et le noeud *Osmunda* s.lat.
  (log-normale décalée). Dans MrBayes, l'âge de la racine se règle avec
  `treeagepr`, et les autres noeuds avec `calibrate`. La commande
  `prset nodeagepr = calibrated` active les calibrations:
  ```MrBayes
  prset treeagepr = uniform(129, 360);
  calibrate OsmundaSL = offsetlognormal(72, 150, 40);
  prset nodeagepr = calibrated;
  ```

6. **Choisir l'horloge relâchée et l'a priori sur le taux**. Le modèle `igr`
  (*independent gamma rates*) permet à chaque branche d'avoir son propre taux.
  Attention: par défaut, MrBayes fixe le taux d'horloge (taux de mutation
  moyen) à 1, ce qui n'a pas de sens quand les âges sont en Ma. Il faut donc
  toujours le changer. Les mêmes paramètres sont utilisés que dans l'analyse
  BEAST ci-haut:  
  ```MrBayes
  prset clockvarpr = igr;
  prset clockratepr = lognorm(-7, 1);
  ```

7. **Montrer le modèle**:
  ```MrBayes
  showmodel;
  ```

8. **Ajuster les paramètres MCMC**:
  ```MrBayes
  mcmcp ngen=2000000 samplefreq=1000 printfreq=10000 diagnfreq=10000 nchains=4 nruns=2 filename=osm_noeuds;
  ```

9. **Lancer la chaîne MCMC**:
  ```MrBayes
  mcmc;
  ```
  
10. Lorsque terminé, **analyser les résultats de la chaîne**:
  ```MrBayes
  sump;
  sumt;
  ```

Pour lancer tout le fichier de commandes en mode non interactif sur le
 serveur de calcul (plus pratique pour une longue analyse):
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/test_datation/MrBayes_nodeOnly
DATA=/home/$USER/Osmundaceae_actuelles.nex
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
CPUS=8
MEM_PER_CPU=1G

## Créer et naviguer vers le dossier de travail
mkdir -p $WD
cd $WD

## Créer un lien vers l'alignement qui sera analysé
ln -s $DATA infile.nex

## Créer un fichier contenant les commandes à exécuter dans MrBayes
echo "#NEXUS

begin mrbayes;
  set autoclose=yes nowarn=yes;
  execute infile.nex;

  lset nst=2 rates=gamma;

  constraint OsmundaSL = Osmunda_cinnamomea Osmunda_claytoniana
    Osmunda_banksiaefolia Osmunda_javanica Osmunda_vachellii
    Osmunda_japonica Osmunda_lancea Osmunda_regalis;
  prset topologypr = constraint(OsmundaSL);

  prset brlenspr = clock:birthdeath;
  prset samplestrat = random sampleprob = 0.46;

  prset treeagepr = uniform(129, 360);

  calibrate OsmundaSL = offsetlognormal(72, 150, 40);
  prset nodeagepr = calibrated;

  prset clockvarpr = igr;
  prset clockratepr = lognorm(-7, 1);

  showmodel;

  mcmcp ngen=2000000 samplefreq=1000 printfreq=10000 diagnfreq=10000
    nchains=4 nruns=2 filename=osm_noeuds;
  mcmc;

  sump;
  sumt;
  
end;
" > mrbayes_node_dating.commands


## Lancer la tâche sur SLURM
sbatch \
  --job-name=mb_node_dating \
  --output=mb_node_dating.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --ntasks=$CPUS \
  --time=$TIME \
  --mem-per-cpu=$MEM_PER_CPU \
  --wrap="mpirun.openmpi -np $CPUS /opt/MrBayes-mpi/src/mb-mpi mrbayes_node_dating.commands"

```


#### Analyser les résultats

- `sump` affiche les statistiques des paramètres. Vérifiez la convergence
  (ESS, PSRF près de 1,0), comme dans le tutoriel sur MrBayes. Vous pouvez
  aussi ouvrir les fichiers `osm_noeuds.run*.p` dans **Tracer**.
- `sumt` produit l'arbre consensus `osm_noeuds.con.tre`. Pour les arbres avec
  horloge, ce fichier contient l'âge de chaque noeud. Ouvrez-le dans
  **FigTree** et affichez `age_median` (*Node Labels*) et `age_95%HPD`
  (*Node Bars*).
- Pour obtenir l'a priori effectif dans MrBayes, ajoutez la commande
  `mcmcp data=no` avant `mcmc`, avec un autre nom de fichier
  (`filename=osm_noeuds_prior`). MrBayes échantillonne alors seulement l'a
  priori.

- **Question**: Les âges obtenus avec MrBayes sont-ils semblables à ceux de
  BEAST? Quelles différences de modèle entre les deux analyses pourraient
  expliquer les écarts (forme de l'horloge relâchée, a priori sur l'arbre,
  etc.)?

---

## Partie 2 – Les fossiles comme feuilles de l'arbre (FBD)

### Principe

La calibration des noeuds n'utilise qu'une petite partie de l'information
 paléontologique: souvent un seul fossile (le plus vieux) par clade, dont il
 faut décider d'avance la position. L'approche des fossiles comme feuilles:

- inclut **tous** les fossiles disponibles, comme des feuilles d'âge connu;
- estime leur **position** dans l'arbre à partir de leurs **caractères**
  **morphologiques** (modèle **Mk/Mkv** de Lewis 2001), plutôt que de l'imposer;
- tient compte de l'**incertitude** sur l'âge de chaque fossile (un intervalle
  stratigraphique plutôt qu'une valeur fixe);
- remplace les calibrations arbitraires par un modèle explicite, le processus
  de **fossilized birth-death** (FBD), dont les paramètres sont estimés:
  taux de spéciation, d'extinction, de fossilisation (échantillonnage des
  fossiles), et proportion des espèces actuelles échantillonnées (**rho**).

Le modèle FBD permet aussi qu'un fossile soit un **ancêtre échantillonné**
 (*sampled ancestor*): un fossile situé directement sur une branche, sans
 descendance propre, qui représente l'ancêtre d'autres espèces de l'arbre.

Ici, on utilise les 33 échantillons pour l'analyse phylogénétique et de
 datation: l'ADN des espèces actuelles, la morphologie de tous les échantillons
 et l'âge des 18 échantillons fossiles (celui des échantillons vivant est de
 "0"):

| Fossile | Âge (Ma) | Fossile | Âge (Ma) |
|---|---|---|---|
| *Todea tidwellii* | 129–140 | *O. dowkeri* | 56–59 |
| *O. cinnamomea* Néogène É.-U. | 16 | *O. ilianensis* | 4–6 |
| *O. cinnamomea* Néogène Japon | 12–14 | *O. oregonensis* | 34–40 |
| *O. cinnamomea* Crétacé Canada | 72–83 | *O. pluma* | 56–66 |
| *O. precinnamomea* | 56–66 | *O. shimokawaensis* | 12–14 |
| *O. chengii* | 145–160 | *O. wehrii* | 16 |
| *O. bromeliaefolia* | 11–14 | *O. pulchella* | 183–185 |
| *O. arnoldii* | 56–66 | *O. liaoningensis* | 153–165 |
| *O. wangii* | 153–165 | *O. plumites* | 153–165 |

### Fossiles comme feuilles dans BEAST 2

Cette section résume le
 [tutoriel *Total-Evidence* de Taming the BEAST](https://taming-the-beast.org/tutorials/Total-Evidence-Tutorial/),
 qui contient plus d'explications et des figures pour chaque étape. Vous
 pouvez le consulter en parallèle.

#### Installer les modules BEAST nécessaires

Deux modules (*packages*) de BEAST 2 sont nécessaires:
- **SA** (*Sampled Ancestors*): le modèle FBD avec ancêtres échantillonnés;
- **MM** (*Morph Models*): les modèles d'évolution des caractères
 morphologiques.

Dans BEAUti: `File > Manage Packages`, sélectionner **SA** puis cliquer sur
 **Install/Upgrade**. Faire de même pour **MM**. Fermer, puis **redémarrer
 BEAUti**.

Sur le serveur de calcul, il faut aussi installer ces modules dans votre
 compte avant de lancer l'analyse:
```bash
/opt/beast/bin/packagemanager -add SA
/opt/beast/bin/packagemanager -add MM
```

#### Préparer l'analyse dans BEAUti

1. **Importer l'ADN**: `File > Import Alignment`, choisir `Osmundaceae_dna.nex`
  (33 taxons, 8628 sites).

2. **Importer la morphologie**: `File > Add Morphological Data`, choisir
  `Osmundaceae_morph.nex`. À la question sur le conditionnement sur les
  caractères variables seulement (Mkv), répondre **Yes**: les caractères
  invariables ne sont presque jamais codés dans une matrice morphologique
  (biais de détermination, *ascertainment bias*). BEAUti crée deux partitions
  morphologiques: une pour les caractères à 2 états et une pour ceux à 3 états.

3. **Onglet *Partitions***: les trois partitions doivent partager **un seul
  arbre**. Dans la colonne *Tree*, renommer l'arbre de la partition d'ADN
  `tree`, puis choisir `tree` pour les deux partitions morphologiques. Garder
  les deux horloges par défaut (une pour l'ADN, une pour la morphologie).

4. **Onglet *Tip Dates***: cocher **Use tip dates**. Choisir
  *numerically as… year* et **Before the present**, puis cliquer sur
  **Auto-configure**. Choisir *use everything*, l'option **after last** et le
  caractère `_`, puis **OK**. BEAUti lit l'âge de chaque unité à la fin de son
  nom. Ce sont des âges **initiaux**; l'incertitude sera ajoutée à l'étape 8.

5. **Onglet *Site Model***:
    - ADN: **HKY**, *Gamma Category Count* = **4**, *Kappa* et
    *Frequencies* estimés.
    - Partitions morphologiques: garder **Lewis Mk** et mettre
    *Gamma Category Count* à **0** (un seul taux par partition).

6. **Onglet *Clock Model***: garder une **Strict Clock** estimée pour l'ADN
  et pour la morphologie. Avec si peu de caractères morphologiques, un modèle
  d'horloge plus complexe serait difficile à estimer.

7. **Onglet *Priors***:
    - **Tree.t**: choisir **Fossilized Birth-Death Model**.
    - **rho**: **0.536** (15 espèces actuelles échantillonnées sur environ
    28 espèces acceptées).
    - Taux d'horloge de l'ADN et de la morphologie (`clockRate.c:...`):
    **Exponential** de moyenne 1,0.
    - **OriginFBD**: **Uniform** entre **230** et **360** Ma, avec une valeur
    initiale de **250** (bouton *initial =*). L'origine est le début du
    processus, sur la branche qui mène à la racine: elle est donc plus vieille
    que la racine elle-même.
    - **diversificationRateFBD**: **Exponential** de moyenne 1,0.
    - **samplingProportionFBD**: **Exponential** de moyenne **0.1**, valeur
    initiale 0.1.
    - **turnoverFBD**: garder l'a priori uniforme par défaut.

8. **Ajouter l'incertitude sur l'âge des fossiles**. Pour chaque fossile avec
  un intervalle d'âge:
    - Cliquer sur **+ Add Prior** et choisir
    **Sampled Ancestors MRCA Prior**.
    - Nommer le groupe (par ex. `tidwellii`), y mettre **un seul taxon**
    (`Todea_tidwellii_135`), puis **OK**.
    - Choisir la distribution **Uniform**, avec les bornes du tableau
    ci-dessus (ici **129** et **140**).
    - Cocher **Tipsonly**: l'a priori porte sur l'âge de la feuille, et non
    sur celui de son ancêtre.

Répétez pour les 16 fossiles qui ont un intervalle d'âge. Les deux
 fossiles avec un âge fixe (16 Ma) n'ont pas besoin d'a priori: ils
 garderont leur âge initial. C'est une étape répétitive; le tutoriel de
 Taming the BEAST fournit un script R (`add_age_uncertainty.R`) pour
 l'automatiser dans le fichier XML.

9. **Onglet *MCMC***: *Chain Length* = **30 000 000**. Sauvegarder sous
  `Osmundaceae_FBD.xml`. Cocher ensuite **Sample From Prior** et sauvegarder
  sous `Osmundaceae_FBD_prior.xml`.

#### Lancer l'analyse

Cette analyse est plus longue que celle de la partie 1: utilisez de
 préférence le serveur de calcul, avec le script SLURM du
 [tutoriel sur les arbres d'espèces](Sanger/08--Arbre_despeces.md)
 (`XML_INPUT=$WD/Osmundaceae_FBD.xml`). Le tutoriel de Taming the BEAST
 fournit aussi des résultats précalculés (dossier
 [`precooked_runs`](https://github.com/bethany-j-allen/Total-Evidence-Tutorial/tree/main/precooked_runs))
 si le temps manque.

#### Analyser les résultats

1. **Tracer**: ouvrir le fichier `.log` de l'analyse et celui de l'analyse
  *sample from prior*.
    - Vérifier la convergence (ESS > 200).
    - Comparer a priori et a posteriori (*Marginal Density*) pour les âges
    des fossiles, les paramètres du modèle FBD et les taux d'horloge.

2. **TreeAnnotator**: *burn-in* de 10 %, **Maximum clade credibility tree**,
  **Median heights**, sur le fichier `.trees` de l'analyse. Un avertissement
  sur les ancêtres échantillonnés est normal.

3. **Visualiser l'arbre**: FigTree n'affiche pas bien les ancêtres
  échantillonnés (branches de longueur nulle). Utilisez plutôt
  [IcyTree](https://icytree.org) dans un navigateur:
    - `Style > Axis > Age` pour l'échelle de temps;
    - décocher `Style > Collapse zero-length edges` pour voir les ancêtres
    échantillonnés;
    - `Style > Node height error bars > height_95%_HPD` pour l'incertitude
    sur les âges.

- **Question**: Quels fossiles sont placés comme ancêtres échantillonnés?
  Leur position dans l'arbre est-elle bien soutenue? Pourquoi la position des
  fossiles est-elle souvent beaucoup plus incertaine que celle des espèces
  actuelles?

### Fossiles comme feuilles dans MrBayes

Toutes les commandes sont regroupées dans le fichier
 `datation_feuilles_mb.nex`. Contrairement à BEAST, MrBayes lit l'ADN et la
 morphologie dans **une seule matrice** de type mixte. Ouvrez
 `Osmundaceae_TED.nex` et repérez la ligne
 `FORMAT DATATYPE=MIXED(DNA:1-8628,STANDARD:8629-8653)`.

**!!! Attention !!!** Comme précédemment, MrBayes peut ne pas être en mesure
 d'exécuter plus d'une ligne de commande à la fois. Il est donc préférable
 de copier et coller dans la console intéractive de MrBayes les commandes une 
 à une, même lorsqu'elles sont présentées comme un bloc de plusieurs commandes.

Ouvrez MrBayes en exécutant `mb` dans le dossier qui contient le fichier
 `Osmundaceae_TED.nex`, puis exécutez ces commandes une à une.

1. **Charger les données**
  ```MrBayes
  execute Osmundaceae_TED.nex;
  ```

2. **définir deux partitions**:
  ```MrBayes
  charset ADN = 1-8628;
  charset morpho = 8629-8653;
  partition donnees = 2: ADN, morpho;
  set partition = donnees;
  ```

3. **Modèles de substitution**: HKY+G pour l'ADN, et Mkv pour la morphologie.
  L'option `coding=variable` est l'équivalent de répondre « Yes » à la
  question sur les caractères variables dans BEAUti:
  ```MrBayes
  lset applyto=(1) nst=2 rates=gamma;
  lset applyto=(2) coding=variable rates=equal;
  unlink statefreq=(all) tratio=(all) shape=(all);
  prset applyto=(all) ratepr=variable;
  ```
 La dernière commande permet à chaque partition d'évoluer à sa propre vitesse
 relative.

4. **Donner un âge à chaque fossile**. Ici, `calibrate` s'applique à des
  **feuilles** plutôt qu'à des noeuds. Les espèces actuelles, sans
  calibration, ont automatiquement un âge de 0. On définit aussi un groupe
  `fossiles`, qui servira au moment du résumé:
  ```MrBayes
  taxset fossiles = Todea_tidwellii_135 Osmunda_cinnamomea_Neogene_USA_16     Osmunda_cinnamomea_Neogene_Japan_13 Osmunda_cinnamomea_Cretaceous_Canada_80 Osmunda_precinnamomea_60 Osmunda_chengii_150 Osmunda_bromeliaefolia_12 Osmunda_arnoldii_60 Osmunda_dowkeri_58 Osmunda_ilianensis_5 Osmunda_oregonensis_36 Osmunda_pluma_60 Osmunda_shimokawaensis_13 Osmunda_wehrii_16 Osmunda_pulchella_184 Osmunda_liaoningensis_160 Osmunda_wangii_160 Osmunda_plumites_160;
  
  calibrate Todea_tidwellii_135 = uniform(129, 140) Osmunda_cinnamomea_Neogene_USA_16 = fixed(16) Osmunda_cinnamomea_Neogene_Japan_13 = uniform(12, 14) Osmunda_cinnamomea_Cretaceous_Canada_80 = uniform(72, 83) Osmunda_precinnamomea_60 = uniform(56, 66) Osmunda_chengii_150 = uniform(145, 160) Osmunda_bromeliaefolia_12 = uniform(11, 14) Osmunda_arnoldii_60 = uniform(56, 66) Osmunda_dowkeri_58 = uniform(56, 59) Osmunda_ilianensis_5 = uniform(4, 6) Osmunda_oregonensis_36 = uniform(34, 40) Osmunda_pluma_60 = uniform(56, 66) Osmunda_shimokawaensis_13 = uniform(12, 14) Osmunda_wehrii_16 = fixed(16) Osmunda_pulchella_184 = uniform(183, 185) Osmunda_liaoningensis_160 = uniform(153, 165) Osmunda_wangii_160 = uniform(153, 165) Osmunda_plumites_160 = uniform(153, 165);
  
  prset nodeagepr = calibrated;
  ```

5. **A priori FBD sur l'arbre**:
  ```MrBayes
  prset brlenspr = clock:fossilization;
  prset samplestrat = random sampleprob = 0.536;
  prset speciationpr = exp(10);
  prset extinctionpr = beta(1, 1);
  prset fossilizationpr = beta(1, 1);
  prset treeagepr = uniform(185, 360);
  ```
    - `clock:fossilization` active le modèle FBD.
    - `sampleprob` correspond au **rho** de BEAST, la proportion des espèces
    du groupe qui a été échantillonné (par rapport à la diversité totale
    connue).
    - Dans MrBayes, `speciationpr` porte sur le taux de diversification
    nette (spéciation − extinction), `extinctionpr` sur l'extinction
    relative (extinction / spéciation), et `fossilizationpr` sur la
    proportion de fossilisation. On attribue des distributions de probabilités
    a priori assez neutres (plates) sur ces paramètres, car ils sont inconnus.
    - Attention: dans MrBayes, `treeagepr` porte sur l'âge de la couronne
    de la **racine**, alors que l'a priori *OriginFBD* de BEAST porte sur 
    l'**origine**, qui est plus vieille. La borne minimale de 185 Ma correspond
    au plus vieux fossile (*O. pulchella*, 183–185 Ma).

6. **Horloge** (stricte, comme dans l'analyse BEAST):
  ```MrBayes
  prset clockvarpr = strict;
  prset clockratepr = lognorm(-7, 1);
  showmodel;
  ```

7. **Ajuster les paramètres MCMC**:
  ```MrBayes
  mcmcp ngen=5000000 samplefreq=1000 printfreq=10000 diagnfreq=10000 nchains=4 nruns=2 filename=osm_feuilles;
  ```

8. **Lancer la chaîne MCMC**:
  ```MrBayes
  mcmc;
  ```

9. **Analyser les résultats**:
  ```MrBayes
  sump;
  sumt;
  delete fossiles;
  sumt output=osm_feuilles_actuelles;
  ```
Le deuxième `sumt`, après `delete fossiles`, résume les mêmes arbres sans
 les fossiles. On obtient un arbre daté des espèces actuelles seulement,
 directement comparable à celui de la partie 1. Lors de ce résumé, MrBayes
 peut afficher un avertissement indiquant qu'un caractère morphologique est
 exclu: sans les fossiles, ce caractère devient invariable. C'est sans
 conséquence pour le résumé des arbres.

Pour lancer l'analyse sur le serveur, utilisez le même code SLURM que dans
 la partie de la datation des noeuds avec MrBayes, en ajustant quelques lignes.
 Voici le code à exécuter pour cela:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/test_datation/MrBayes_tipDating
DATA=/home/$USER/Osmundaceae_TED.nex
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
CPUS=8
MEM_PER_CPU=1G

## Créer et naviguer vers le dossier de travail
mkdir -p $WD
cd $WD

## Créer un lien vers l'alignement qui sera analysé
ln -s $DATA infile.nex

## Créer un fichier contenant les commandes à exécuter dans MrBayes
echo "#NEXUS

begin mrbayes;
  set autoclose=yes nowarn=yes;
  execute infile.nex;

  charset ADN = 1-8628;
  charset morpho = 8629-8653;
  partition donnees = 2: ADN, morpho;
  set partition = donnees;

  lset applyto=(1) nst=2 rates=gamma;
  lset applyto=(2) coding=variable rates=equal;
  unlink statefreq=(all) tratio=(all) shape=(all);
  prset applyto=(all) ratepr=variable;

  taxset fossiles = Todea_tidwellii_135 Osmunda_cinnamomea_Neogene_USA_16
    Osmunda_cinnamomea_Neogene_Japan_13
    Osmunda_cinnamomea_Cretaceous_Canada_80 Osmunda_precinnamomea_60
    Osmunda_chengii_150 Osmunda_bromeliaefolia_12 Osmunda_arnoldii_60
    Osmunda_dowkeri_58 Osmunda_ilianensis_5 Osmunda_oregonensis_36
    Osmunda_pluma_60 Osmunda_shimokawaensis_13 Osmunda_wehrii_16
    Osmunda_pulchella_184 Osmunda_liaoningensis_160 Osmunda_wangii_160
    Osmunda_plumites_160;

  calibrate
    Todea_tidwellii_135 = uniform(129, 140)
    Osmunda_cinnamomea_Neogene_USA_16 = fixed(16)
    Osmunda_cinnamomea_Neogene_Japan_13 = uniform(12, 14)
    Osmunda_cinnamomea_Cretaceous_Canada_80 = uniform(72, 83)
    Osmunda_precinnamomea_60 = uniform(56, 66)
    Osmunda_chengii_150 = uniform(145, 160)
    Osmunda_bromeliaefolia_12 = uniform(11, 14)
    Osmunda_arnoldii_60 = uniform(56, 66)
    Osmunda_dowkeri_58 = uniform(56, 59)
    Osmunda_ilianensis_5 = uniform(4, 6)
    Osmunda_oregonensis_36 = uniform(34, 40)
    Osmunda_pluma_60 = uniform(56, 66)
    Osmunda_shimokawaensis_13 = uniform(12, 14)
    Osmunda_wehrii_16 = fixed(16)
    Osmunda_pulchella_184 = uniform(183, 185)
    Osmunda_liaoningensis_160 = uniform(153, 165)
    Osmunda_wangii_160 = uniform(153, 165)
    Osmunda_plumites_160 = uniform(153, 165);
  prset nodeagepr = calibrated;

  prset brlenspr = clock:fossilization;
  prset samplestrat = random sampleprob = 0.536;
  prset speciationpr = exp(10);
  prset extinctionpr = beta(1, 1);
  prset fossilizationpr = beta(1, 1);
  prset treeagepr = uniform(185, 360);

  prset clockvarpr = strict;
  prset clockratepr = lognorm(-7, 1);

  showmodel;

  mcmcp ngen=5000000 samplefreq=1000 printfreq=10000 diagnfreq=10000
    nchains=4 nruns=2 filename=osm_feuilles;
  mcmc;

  sump;
  sumt;

  delete fossiles;
  sumt output=osm_feuilles_actuelles;
  
end;
" > mrbayes_tip_dating.commands


## Lancer la tâche sur SLURM
sbatch \
  --job-name=mb_tip_dating \
  --output=mb_tip_dating.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --ntasks=$CPUS \
  --time=$TIME \
  --mem-per-cpu=$MEM_PER_CPU \
  --wrap="mpirun.openmpi -np $CPUS /opt/MrBayes-mpi/src/mb-mpi mrbayes_tip_dating.commands"

```

#### Analyser les résultats

- Comme dans la partie sur la datation des noeuds: examiner le résultat de
  `sump`, Tracer et FigTree (`age_median`, `age_95%HPD`).
- `osm_feuilles.con.tre` contient l'arbre avec les fossiles, et
  `osm_feuilles_actuelles.con.tre` l'arbre des espèces actuelles seulement.
- MrBayes permet aussi d'estimer si certains fossiles pourraient être des
  ancêtres directs des espèces vivantes dans le présent: les fossiles qui
  sont sur une branche de longueur 0 sont des ancêtres directs.

### Fichiers de sortie de MrBayes

En plus des fichiers déjà décrits dans le tutoriel sur MrBayes:

1. **`*.con.tre`**: pour une analyse avec horloge, l'arbre consensus contient
  pour chaque noeud l'âge moyen (`age_mean`), l'âge médian (`age_median`) et
  l'intervalle de crédibilité à 95 % (`age_95%HPD`), en Ma.

2. **`*.run*.p`**: en plus des paramètres habituels, ces fichiers contiennent
  le taux de l'horloge (`clockrate`) et, selon le modèle, la variance de
  l'horloge relâchée (`igrvar`), les paramètres du modèle de naissance-mort
  (`net_speciation`, `relative_extinction`, `relative_fossilization`) et,
  pour le modèle FBD, la proportion de fossiles qui sont des ancêtres
  échantillonnés (`prop_ancfossil`). La colonne `TH` est la hauteur de
  l'arbre en substitutions par site (et non en Ma).

3. **`osm_feuilles_actuelles.*`**: les résumés produits après
  `delete fossiles`.

---

## Questions et exercices

1. **Comparaison des approches**:
   - Comparez l'âge de la racine et celui d'*Osmunda* s.lat. dans les quatre
   analyses (noeuds/feuilles × BEAST/MrBayes). Présentez-les dans un tableau
   avec leurs intervalles à 95 %.
   - Laquelle des deux approches donne les âges les plus vieux, et les
   intervalles les plus larges? Comparez vos résultats à ceux de Grimm et
   al. (2015).

2. **L'importance de l'a priori**:
   - Pour chaque analyse de datation des noeuds, comparez l'a priori effectif
   (analyse sans données) à l'a posteriori pour l'âge de la racine. Les données
   moléculaires apportent-elles beaucoup d'information sur les âges, ou
   sont-ils surtout déterminés par les calibrations?

3. **Les fossiles**:
   - Dans la partie sur la datation avec les fossiles comme feuilles, comparez
   l'âge a posteriori des fossiles qui avaient un intervalle d'âge avec leur
   intervalle d'origine. Les données permettent-elles de préciser leur âge?
   - Le fossile *O. pulchella* a d'abord été décrit dans le genre *Osmunda*,
   puis transféré dans *Osmundastrum* (le groupe d'*O. cinnamomea*). Où se
   place-t-il dans votre arbre? Comment ce genre d'incertitude sur la
   position d'un fossile affecte-t-il la calibration des noeuds?

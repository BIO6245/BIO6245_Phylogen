# Des formats de fichiers à l'analyse de parcimonie dans Mesquite

Ce tutoriel vous montrera comment les données phylogénétiques sont structurées
 dans les formats de fichiers les plus courants (FASTA non aligné, FASTA aligné,
 PHYLIP et NEXUS), en vous faisant créer, à la main, dans un éditeur de texte,
 une petite matrice de caractères morphologiques. Vous importerez ensuite cette
 matrice dans [Mesquite](https://mesquiteproject.org/) pour la visualiser et
 effectuer une recherche du (ou des) arbre(s) le(s) plus parcimonieux.

**Important** : utilisez un éditeur de texte brut (Notepad++, Bloc-notes /
 Notepad, TextEdit en mode texte brut, VS Code, BBEdit, etc.), jamais Word ou un
 autre traitement de texte : ces logiciels insèrent du formatage invisible qui
 brise les fichiers de données.

---

## La matrice à coder

Voici la matrice de caractères que vous allez transcrire dans chacun des
 formats. Elle décrit trois caractères morphologiques discrets chez quatre
 taxons :

| Taxon | Présence de pattes | Différenciation des pattes | Pattes avant pinces (0) ou mandibules (1) |
|---|---|---|---|
| Nematode | 0 | - | - |
| Arthropode ancestral | 1 | 0 | - |
| Homard | 1 | 1 | 0 |
| Insecte | 1 | 1 | 1 |

Le symbole `-` indique un caractère **inapplicable** : par exemple, on ne peut
 pas coder la différenciation des pattes chez un taxon qui n'a pas de pattes du
 tout comme un nématode. C'est différent d'une donnée **manquante**
 (habituellement codée `?`), qui signifierait qu'on ne sait tout simplement pas
 quel état le taxon possède. Toutefois, dans la majorité des programmes 
 d'analyse, les `-` et les `?` sont traités exactement de la même façon, comme
 des données manquantes.

---

## FASTA non aligné

Le format FASTA encode chaque taxon par un en-tête commençant par `>`, suivi de
 sa séquence sur la ou les lignes suivantes. Pour une matrice de caractères
 comme la nôtre, où chaque taxon a déjà exactement un état par caractère, ce
 concept de « non aligné » n'a pas vraiment de sens: il y a toujours 3 états,
 peu importe le taxon. C'est plutôt avec de vraies séquences d'ADN, telles
 qu'elles sortent d'un séquenceur, que ce concept devient concret.

Imaginez que vous avez reçu ces 4 lectures brutes (non alignées) d'un même gène
 chez 4 taxons. Elles n'ont pas toutes la même longueur, parce que certaines
 lectures sont incomplètes (bouts manquants) et qu'un des taxons possède une
 insertion de 3 paires de bases que les autres n'ont pas :

```
>Taxon_A
ATGCTAGCTA
>Taxon_B
GCGGGTAGCTA
>Taxon_C
ATGCTAGC
>Taxon_D
TGCTAGCTA
```

- **Question** : Sans alignement, comment un logiciel pourrait-il savoir que la
 base en position 5 chez `Taxon_A` correspond à la même position que la base en
 position 6 chez `Taxon_B`?

Ce format n'est pas utilisable tel quel pour une analyse phylogénétique: sans
 alignement, un logiciel ne peut pas savoir quelles positions sont homologues
 (c'est-à-dire dérivées d'une même position ancestrale) entre les taxons. Des
 séquences non alignés sont donc équivalentes à une description morphologique
 qui n'a pas été codée en matrice de caractères. Il faut donc aligner les
 séquences.

---

## FASTA aligné

### FASTA aligné: exemple ADN

Un algorithme d'alignement (comme ceux vus dans le tutoriel sur l'alignement de
 séquences Sanger) insère des espaces (« gaps », représentés par `-`) là où une
 séquence est plus courte qu'une autre, de façon à faire correspondre les
 positions homologues entre les colonnes. Voici à quoi ressembleraient nos 4
 séquences une fois alignées :

```
>Taxon_A
ATGC---TAGCTA
>Taxon_B
--GCGGGTAGCTA
>Taxon_C
ATGC---TAGC--
>Taxon_D
-TGC---TAGCTA
```

- **Question** : Toutes les séquences ont maintenant la même longueur. Combien
 de colonnes (positions) l'alignement compte-t-il?
- **Question** : Chez `Taxon_D`, il y a des `-` à la fois au début, au milieu et
 à la fin de la séquence. Représentent-ils le même phénomène biologique?

---

### FASTA aligné: exemple morphologie

Revenons à la matrice de caractères morphologiques. Puisque chaque taxon a déjà
 exactement un état par caractère, elle est en un sens déjà « alignée » dès le
 départ: il suffit de représenter les positions inapplicables ou manquantes
 avec un symbole (`-` ou `?`) plutôt que de les omettre, exactement comme on l'a
 fait ci-dessus pour l'ADN.

- **Exercice** : Dans votre éditeur de texte, créez un fichier contenant la
 matrice de caractères morphologiques au format FASTA aligné. Chaque séquence
 doit faire exactement 3 caractères de long. Enregistrez le fichier sous le nom
 `arthropodes.fasta`.

Le résultat devrait ressembler à ceci:  
```
>Nematode
0--
>Arthropode_ancestral
10-
>Homard
110
>Insecte
111
```

- **Question** : Combien de caractères y a-t-il dans cet alignement? Combien de
 taxons?

---

## PHYLIP

Le format PHYLIP place d'abord une ligne d'en-tête indiquant le nombre de taxons
 et le nombre de caractères, séparés par un espace. Chaque ligne suivante
 contient le nom du taxon, puis la séquence.

Dans le format **PHYLIP strict**, le nom du taxon doit occuper exactement 10
 caractères (complétés par des espaces au besoin), suivi directement de la
 séquence, sans espace supplémentaire. Le format **PHYLIP relâché** (« relaxed
 »), plus tolérant, sépare simplement le nom de la séquence par un ou plusieurs
 espaces, peu importe la longueur du nom.

### PHYLIP: exemple morphologie

- **Exercice** : Créez un fichier `arthropodes.phy` au format PHYLIP relâché
 contenant la matrice de caractères morphologiques.

Le résultat final devrait ressembler à ceci:  
```
4 3
Nematode             0--
Arthropode_ancestral 10-
Homard               110
Insecte              111
```

- **Question** : Le nom `Arthropode_ancestral` dépasse 10 caractères. 
 Pouvez-vous créer un fichier équivalent en format PHYLIP strict?

### PHYLIP: exemple ADN

Le même principe s'applique aux séquences moléculaires : la ligne d'en-tête
 indique le nombre de taxons et le nombre de positions dans l'alignement, puis
 chaque taxon est suivi de sa séquence alignée. Voici notre alignement d'ADN
 (celui de la section « FASTA aligné: exemple ADN ») au format PHYLIP
 relâché:  

```
4 13
Taxon_A ATGC---TAGCTA
Taxon_B --GCGGGTAGCTA
Taxon_C ATGC---TAGC--
Taxon_D -TGC---TAGCTA
```

- **Question** : D'où vient le chiffre `13` dans la ligne d'en-tête?

---

## NEXUS

Le format NEXUS est plus verbeux, mais aussi plus explicite et plus flexible:
 il déclare formellement le nombre de taxons, le nombre de caractères, le type
 de données et les symboles utilisés avant de présenter la matrice. C'est le
 format natif de Mesquite.

Le format NEXUS (Maddison, Swofford et Maddison, 1997) est le format de fichier
 le plus utilisé en phylogénétique. Il est conçu pour être à la fois lisible par
 un humain et extensible: on peut y stocker dans le même fichier des séquences,
 des arbres, des modèles d'évolution, des contraintes d'analyse, et plus encore.
 **Mesquite**, **PAUP\***, **MrBayes** et de nombreux autres logiciels le lisent
  nativement.
 
### Principes généraux

Un fichier NEXUS commence obligatoirement par une ligne contenant uniquement
 `#NEXUS`. Cette ligne indique au logiciel qu'il s'agit d'un fichier NEXUS. Elle doit être la toute première ligne du fichier, sans espace ni caractère avant
  elle.

Le reste du fichier est organisé en **blocs**. Chaque bloc commence par
 `BEGIN <nom>;` et se termine par `END;`. À l'intérieur d'un bloc se trouvent
 des **commandes**, chacune terminée par un point-virgule (`;`). Les espaces,
 tabulations et retours à la ligne entre les commandes sont ignorés: c'est le
 `;` qui délimite chaque commande, pas le saut de ligne.

```
#NEXUS

BEGIN <nom_du_bloc>;
    commande1 argument1 argument2;
    commande2 argument;
    commande3
      argument1
      argument2
      argument3
      argument4
    ;
END;
```

Le format n'est pas sensible à la casse: `BEGIN DATA`, `begin data` et
 `Begin Data` sont équivalents. Par convention, on écrit les mots-clés du format
 en majuscules pour les distinguer des données (noms de taxons, séquences).

Les **commentaires** s'écrivent entre crochets et peuvent apparaître n'importe
 où:  
```
[Ceci est un commentaire, il sera ignoré par les logiciels.]
```

#### Le bloc DATA

Le bloc `DATA` est le plus fondamental : il contient à la fois les dimensions du
 jeu de données (nombre de taxons, nombre de caractères) et la matrice
 elle-même. C'est l'équivalent condensé d'un bloc `TAXA` suivi d'un bloc
 `CHARACTERS` (voir sections suivantes), et c'est la forme à privilégier pour sa
 concision.

```
#NEXUS

BEGIN DATA;
    DIMENSIONS NTAX=4 NCHAR=13;
    FORMAT DATATYPE=DNA MISSING=? GAP=-;
    MATRIX
        Taxon_A    ATGC---TAGCTA
        Taxon_B    ATGCGGGTAGCTA
        Taxon_C    ATGC---TAGC--
        Taxon_D    ATGC---TAGCTA
    ;
END;
```

La commande `DIMENSIONS` déclare le nombre de taxons (`NTAX`) et le nombre de
 caractères (`NCHAR`). Ces valeurs doivent correspondre exactement à ce qui se
 trouve dans la matrice: si `NTAX=4` mais que la matrice n'a que 3 lignes, le
 logiciel produira une erreur.

La commande `FORMAT` décrit comment interpréter les données de la matrice. Les
 arguments les plus courants sont:
 
  - `DATATYPE` : le type de données. Les valeurs possibles incluent `DNA`,
   `RNA`, `PROTEIN` et `STANDARD` (pour des caractères discrets quelconques,
    comme une matrice morphologique).
  - `MISSING`: le symbole utilisé pour une donnée manquante (conventionnellement
   `?`).
  - `GAP`: le symbole utilisé pour un gap ou une position inapplicable
   (conventionnellement `-`).
  - `SYMBOLS`: pour `DATATYPE=STANDARD` seulement, liste les symboles d'états
   autorisés (ex. `SYMBOLS="01"` pour une matrice binaire). Pour `DATATYPE=DNA`,
   les symboles `A`, `C`, `G`, `T` (et leurs codes IUPAC pour les ambiguïtés) sont reconnus automatiquement.
  - `INTERLEAVE` : si présent (valeur `YES`), indique que la matrice est en
   format entrelacé (voir plus bas).


#### La commande MATRIX

`MATRIX` contient les données elles-mêmes. Chaque ligne commence par le nom du
 taxon, suivi de sa séquence ou de ses états. Les espaces entre le nom et la
 séquence sont ignorés, ce qui permet d'aligner visuellement les colonnes pour
 faciliter la lecture. La commande se termine par un `;` après la dernière ligne
 de données.

Les noms de taxons contenant des espaces doivent être entourés d'apostrophes:  
```
  MATRIX
    'Arthropode ancestral'  10-
  ;
```


## Les blocs TAXA et CHARACTERS séparés

Avant que le bloc `DATA` ne soit standardisé, il était courant (et certains
 logiciels l'exigent encore) de séparer la liste des taxons du contenu de la
 matrice en deux blocs distincts. Le bloc `TAXA` déclare les noms des taxons, et
 le bloc `CHARACTERS` contient le reste.

```
#NEXUS

BEGIN TAXA;
    DIMENSIONS NTAX=4;
    TAXLABELS
        Taxon_A
        Taxon_B
        Taxon_C
        Taxon_D
    ;
END;

BEGIN CHARACTERS;
    DIMENSIONS NCHAR=13;
    FORMAT DATATYPE=DNA MISSING=? GAP=-;
    MATRIX
        Taxon_A    ATGC---TAGCTA
        Taxon_B    ATGCGGGTAGCTA
        Taxon_C    ATGC---TAGC--
        Taxon_D    ATGC---TAGCTA
    ;
END;
```

Cette forme est fonctionnellement équivalente au bloc `DATA` pour les logiciels
 courants (Mesquite, PAUP\*, MrBayes). Elle présente cependant un inconvénient
 pratique: si vous ajoutez un taxon dans la matrice, vous devez penser à mettre
 à jour `NTAX` dans le bloc `TAXA` *et* la liste `TAXLABELS`. Avec le bloc
 `DATA`, une seule déclaration `NTAX` suffit.


#### Le bloc TREES

Le bloc `TREES` stocke un ou plusieurs arbres phylogénétiques au format Newick
 (également appelé format parenthésé). Il peut coexister avec un bloc `DATA`
 dans le même fichier, ce qui permet de tout garder au même endroit.

```
BEGIN TREES;
    TREE resultat = (Taxon_A,(Taxon_B,(Taxon_C,Taxon_D)));
END;
```

Dans la notation Newick, les parenthèses indiquent le regroupement (clade), et
 les virgules séparent les taxons au même niveau. Les longueurs de branches
 s'ajoutent après un `:` :

```
    TREE resultat = (A:0.05,(B:0.12,(C:0.08,D:0.03):0.15):0.02);
```

Un même fichier peut contenir plusieurs arbres (par exemple, tous les arbres
 également parcimonieux, ou l'ensemble des arbres d'une analyse Bayésienne) :

```
BEGIN TREES;
    TREE arbre_1 = (Taxon_A,(Taxon_B,(Taxon_C,Taxon_D)));
    TREE arbre_2 = (Taxon_A,(Taxon_C,(Taxon_B,Taxon_D)));
END;
```


#### Le bloc ASSUMPTIONS

Le bloc `ASSUMPTIONS` (ou son alias `PAUP` dans certains contextes) permet de
 définir des paramètres d'analyse supplémentaires, notamment l'**ordre** des
 caractères et leurs **poids**.

Par défaut, tous les caractères dans un fichier NEXUS sont traités comme **non**
 **ordonnés** (non additifs): une transformation de l'état 0 à l'état 2 coûte 1
 pas, exactement comme de l'état 0 à l'état 1. Si vous voulez qu'un caractère
 soit **ordonné** (additif, c'est-à-dire qu'une transformation de l'état 0 à
 l'état 2 coûte 2 pas parce qu'elle doit passer par l'état 1), vous devez le
 déclarer dans ce bloc:

```
BEGIN ASSUMPTIONS;
    OPTIONS DEFTYPE=unord;
    TYPESET * default = ord: 2, unord: 1 3;
END;
```

Dans cet exemple, le caractère 2 est ordonné, et les caractères 1 et 3 sont non
 ordonnés. Le `*` indique que c'est le typeset actif par défaut.


#### Le bloc CHARSTATELABELS

Pour les matrices de caractères discrets (`DATATYPE=STANDARD`), il est possible
 d'ajouter à l'intérieur du bloc `DATA` (ou `CHARACTERS`) une commande
 `CHARSTATELABELS` qui donne un nom à chaque caractère et à chacun de ses états.
 Mesquite utilisera ces noms dans son interface graphique.

```
BEGIN DATA;
    DIMENSIONS NTAX=4 NCHAR=3;
    FORMAT DATATYPE=STANDARD SYMBOLS="01" MISSING=? GAP=-;
    CHARSTATELABELS
        1 Presence_de_pattes / absentes presentes,
        2 Differenciation_des_pattes / non_differenciees differenciees,
        3 Pattes_avant / pinces mandibules
    ;
    MATRIX
        Nematode                0--
        Arthropode_ancestral    10-
        Homard                  110
        Insecte                 111
    ;
END;
```

La syntaxe est: `<numéro> <nom_du_caractère> / <état_0> <état_1> ...`. Les noms
 ne peuvent pas contenir d'espaces (utilisez des tirets bas).


#### Format entrelacé (interleaved)

Pour de longues séquences, il peut être pratique de découper la matrice en blocs
 de colonnes plutôt que d'avoir des lignes très longues. C'est le format
 **entrelacé**: chaque taxon apparaît plusieurs fois dans la matrice, et ses
 séquences sont lues dans l'ordre. 
 
Il faut alors déclarer `INTERLEAVE=YES` dans la commande `FORMAT`:  
```
BEGIN DATA;
    DIMENSIONS NTAX=4 NCHAR=20;
    FORMAT DATATYPE=DNA MISSING=? GAP=- INTERLEAVE=YES;
    MATRIX
        Taxon_A    ATGCTAGCTA
        Taxon_B    ATGCTAGCTA
        Taxon_C    ATGCTAGCTA
        Taxon_D    ATGCTAGCTA

        Taxon_A    ATGCTAGCTA
        Taxon_B    ATGCTAGCTA
        Taxon_C    ATGCTAGCTA
        Taxon_D    ATGCTAGCTA
    ;
END;
```

Les lignes vides entre les blocs sont optionnelles mais améliorent la
 lisibilité. C'est souvent le format produit par défaut par des logiciels comme
 Mesquite lorsque les séquences sont longues.

---

### NEXUS: exemple morphologie

- **Exercice** : Créez un fichier `arthropodes.nex` avec le contenu suivant.
 Pour simplifier la tâche, il est possible de copier et coller les lignes
 après la 1ère ligne du fichier PHYLIP relaxé pour construire la partie MATRIX.

```
#NEXUS

BEGIN DATA;
    DIMENSIONS NTAX=4 NCHAR=3;
    FORMAT DATATYPE=STANDARD SYMBOLS="01" MISSING=? GAP=-;
    
    CHARSTATELABELS
        1 Presence_de_pattes / absentes presentes,
        2 Differenciation_des_pattes / non_differenciees differenciees,
        3 Pattes_avant / pinces mandibules
    ;

    MATRIX
        Nematode             0--
        Arthropode_ancestral 10-
        Homard               110
        Insecte              111
    ;

END;
```

Notez la ligne `FORMAT` : `DATATYPE=STANDARD` indique qu'il s'agit de caractères
 morphologiques standards (plutôt que de nucléotides ou d'acides aminés),
 `SYMBOLS="01"` liste les états possibles, et `GAP=-` déclare explicitement que `-` représente une position inapplicable.

---

### NEXUS: exemple ADN

Pour des séquences d'ADN, il faut déclarer `DATATYPE=DNA` plutôt que
 `DATATYPE=STANDARD`, et il n'y a pas besoin de déclarer les symboles 
 (`A`, `C`,`G`, `T`), puisqu'ils sont compris automatiquement. 
 
Voici notre alignement d'ADN en format NEXUS:  
```
#NEXUS

BEGIN DATA;
    DIMENSIONS NTAX=4 NCHAR=13;
    FORMAT DATATYPE=DNA MISSING=? GAP=-;
    MATRIX
        Taxon_A    ATGC---TAGCTA
        Taxon_B    --GCGGGTAGCTA
        Taxon_C    ATGC---TAGC--
        Taxon_D    -TGC---TAGCTA
    ;
END;
```

---

## Importer la matrice dans Mesquite

Mesquite est conçu autour du format NEXUS: c'est le format qu'il lit et écrit
 nativement. Bien qu'il puisse importer certains autres formats (surtout pour
 des données moléculaires), les matrices de caractères discrets (« standard »),
 comme la nôtre, doivent passer par un fichier NEXUS. **C'est pourquoi vous**
 **importerez `arthropodes.nex`**, et non les fichiers FASTA ou PHYLIP créés
 plus tôt (ceux-ci vous ont plutôt servi à comprendre comment un même jeu de
 données peut être structuré différemment selon le logiciel utilisé).

  1. **Ouvrez Mesquite**, puis choisissez **File > Open File...** et
   sélectionnez `arthropodes.nex`.
  2. Mesquite devrait ouvrir une fenêtre listant vos taxons et une fenêtre
   montrant la matrice de caractères. Si la matrice ne s'affiche pas
   automatiquement, ouvrez-la via **List of Characters** ou en double-cliquant
   sur la matrice dans la fenêtre des taxons.
  3. Examinez la matrice affichée. Si vous avez inclus le bloc
   `CHARSTATELABELS`, les noms des caractères et de leurs états devraient 
   apparaître dans l'interface.

- **Question** : Est-ce que Mesquite affiche correctement les positions
 inapplicables (`-`) chez `Nematode` et `Arthropode_ancestral`? Comment
 sont-elles représentées visuellement?


### Analyse phylogénétique dans Mesquite

1. Dans la fenêtre des taxons, sélectionnez **Taxa&Trees > Make New Trees**
 **Block from > Other Tree Search > Mesquite Heuristic (Add & rearrange)**.
2. Choisissez **Treelength** comme critère de recherche.
3. Choisissez **SPR Rearranger** comme méthode de recherche d'arbre.
4. Choisissez **100** comme nombre maximum d'arbres égaux à conserver. 
5. Mesquite ouvrira une fenêtre d'arbre montrant le ou les arbres les plus
 parcimonieux trouvés, ainsi que leur longueur (nombre de changements d'état
 nécessaires).

## Exercices

1. **Modifiez un état** dans `arthropodes.nex` (par exemple, changez l'état du
 caractère 3 chez `Homard` de `0` à `1`), puis réimportez le fichier dans
 Mesquite.
2. **Ajoutez un cinquième taxon** de votre choix (réel ou fictif) à votre
 fichier NEXUS, avec ses propres états pour les 3 caractères. Réimportez et
 refaites la recherche.
3. **Convertissez à la main** votre fichier NEXUS modifié (exercice 2) vers les
 formats FASTA aligné et PHYLIP, pour vous assurer que vous maîtrisez bien les
 trois formats.

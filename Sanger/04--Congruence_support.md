# Congruence et support avec PAUP*

Ce tutoriel montre comment utiliser PAUP\* pour calculer des indices de support (CI, RI, 
Bootstrap, Jackknife), faire des tests d'incongruence (ILD) et l'application de **tests KS** 
(Kishino-Hasegawa) avec le critère de parcimonie et des **tests AU** (Approximately Unbiased) avec 
le critère de maximum de vraisemblance sur PAUP\*.

---

## Chargement du fichier d'alignement Nexus dans PAUP*

1. Assurez-vous d'avoir déjà **transféré le fichier Nexus** [`test.nex`](fichiers/test.nex) depuis 
votre ordinateur local vers le serveur de calcul à l'aide de la commande `rsync`.

2. **Connectez-vous au serveur**.  

3. **Ouvrez le fichier Nexus** dans PAUP*:  
   ```bash
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

---

## Indice d'homoplasie (CI et RI)


Effectuer une recherche heuristique:  
```paup
hs addseq=random nreps=100 multre=yes nchuck=5 chucklen=1;

```

Examiner un des meilleurs arbres:  
```paup
descr 1;

```

- **Question**: Quel est l'indice de cohérence (CI) et l'indice de rétention (RI) de cet arbre?
- **Question**: Examinez un autre arbre. Est-ce que le CI et le RI sont différents? Pourquoi?

---

## Consensus strict et majoritaire

Étant donné qu'il existe un grand nombre d'arbres avec un même score de parcimonie, il est 
préférable de calculer un arbre de consensus strict pour regarder quelles branches ont du support 
avec la méthode de maximum de parcimonie.

Voici comment calculer un arbre de consensus strict:  
```paup
contre / strict=yes majrule=no;

```

Et pour un arbre de consensus majoritaire:  
```paup
contre / strict=no majrule=yes;

```

- **Question**: Y a-t-il une différence entre les deux arbres de consensus? Pourquoi?
- **Question**: Combien d'arbres ont contribué à la création de ce consensus?

---

## Analyse de bootstrap

Le **bootstrap** est une méthode statistique permettant d'évaluer la robustesse des branches dans 
un arbre phylogénétique.

Effectuer une analyse de bootstrap:  
```paup
boots nreps=100 search=heuristic grpfreq=no / addseq=random nreps=3 multre=yes steepest=no nchuck=3 chucklen=1 limitperrep=yes;

```

- **Question**: Quelles sont les branches les mieux supportées? Lesquelles sont les moins bien 
supportées?
- **Question**: Essayez d'**augmenter le nombre de ré-échantillonnages bootstrap** pour améliorer 
la précision de l'analyse. Est-ce que le support pour certains groupes change?
- **Question**: Si vous répétez l'analyse avec le même nombre de réplicats bootstrap (100), est-ce 
que le support pour certains groupes change?

---

## Test ILD (Incongruence Length Difference)

Le **test ILD** est utilisé pour vérifier l'incongruence entre plusieurs partitions de données. 
Imaginez un scénario où la matrice contient les données combinées de deux gènes différents. Disons 
que le premier gène est 13 nucléotides de long, et le deuxième est 7 nucléotides de long. Nous 
voulons tester si le signal phylogénétique des deux gènes est congruent ou non.   

1. Créez les partitions dans votre fichier Nexus ou utilisez la commande:  
   ```paup
   charset gene1 = 1-13;
   charset gene2 = 14-20;
   charpartition NomDeMaPartition = partition1: gene1, partition2: gene2;
   
   ```

2. Faites une analyse phylogénétique basée uniquement sur le 1er gène:   
   ```paup
   exclude all;
   include gene1;
   hs;
   contre / strict=yes majrule=no;
   
   ```

3. Comparez le résultat avec celui d'une analyse phylogénétique basée sur le 2e gène:  
   ```paup
   exclude all;
   include gene2;
   hs;
   contre / strict=yes majrule=no;
   
   ```

- **Question**: Où sont les différences entre les topologies estimées de ces deux gènes?

2. Maitenant, nous voulons déterminer si le signal phylogénétique des deux gènes est 
significativement différant, ou si les différences sont juste dû au hasard. Exécuter le test ILD 
avec la commande suivante:  
   ```paup
   include all;
   hompart partition=NomDeMaPartition nreps=100 search=heuristic / addseq=random nreps=3 multre=yes steepest=no nchuck=3 chucklen=1 limitperrep=yes;;

   ```

Le test ILD génère un **p-value**. Un p-value faible (< 0.05) indique une incongruence 
significative entre les partitions.

- **Question**: Est-ce que les deux gènes sont significativement différents dans leur signal?
- **Quesiton**: Si les deux gènes étaient significativement différents, qu'aurait-on dû faire?

---

## Test KS (Kishino-Hasegawa) avec le critère de parcimonie

Le **test KS** (Kishino-Hasegawa) est utilisé pour comparer des arbres en fonction du nombre de 
changements sous le critère de parcimonie.

Il est très utile notamment lorsqu'on obtient un résultat phylogénétique qui semble différent de notre 
hypothèse, et qu'on veut savoir si la différence est significative. Par exemple, si la classification 
présentement acceptée du genre *Aus* place *Aus bus*, *Aus cus* , *Aus dus*, *Aus eus*, et *Aus fus* 
dans le même sous-genre, on pourrait émettre l'hypothèse que ces espèces forment un groupe 
monophylétique. 

Si le résultat de notre analyse de parcimonie ne place pas ces espèces comme formant 
un groupe monophylétique, on aimerait savoir si la différence entre le résultat obtenu et notre 
hypothèse est significatif. On pourrait se fier aux valeurs bootstrap; si elles sont élevées, on peut 
conclure que nos données contredisent assez fortement notre hypothèse. Toutefois, le bootstrap n'est pas 
un test statistique.

Un alternative est de faire un **test KS**, qui permet de déterminer si notre meilleur arbre est 
**significativement plus court** en parcimonie qu'un arbre où on obligerait *Aus bus*, *Aus cus*, 
*Aus dus*, *Aus eus*, et *Aus fus* de former un groupe monophylétique. Cet arbre où une certaine 
relation est forcée se nomme un **arbre sous contrainte**. Pour l'obtenir, il faut créer à la main 
un arbre où toutes les espèces sont en polytomie, sauf le groupe qu'on veut forcer d'être 
monophylétique. Ensuite, on demande à PAUP\* de chercher les arbres les plus parcimonieux, mais 
respectant notre contrainte. Évidemment, il est possible que cette contrainte mène à des arbres plus 
longs que ceux sans contrainte. 

Le **test KS** permet de savoir si la différence de longueur entre les meilleurs arbres sans 
contrainte et les arbres contraints est significative d'un point de vue statistique. Pour se faire, 
le test procède en créant des matrices rééchantillonnées par la méthode du boostrap. Ensuite, pour 
chaque matrice rééchantillonnée, il estime le score de parcimonie sur tous les arbres (meilleur arbre 
et arbres contraints), et détermine la différence de score entre le meilleur arbre et les arbres 
contraints. Finalement, la *distribution statistique* de ces différences est comparée à la différence 
de score observée dans la matrice originale. Si la différence dans la matrice originale est plus 
grande que celle de la distribution statistique, alors le test détermine qu'il y a une différence 
significative.

1. Transférer le fichiers contenant des arbres [`contraintes.nex`](fichiers/contraintes.nex) depuis 
votre ordinateur local vers le serveur de calcul à l'aide de la commande `rsync`. Vous pouvez ouvrir un 
nouveau terminal de votre ordinateur pour transférer ce fichier sans fermer la ligne de commande PAUP\*.

2. Charger les arbres dans PAUP\* avec cette commande et examinez les, un après l'autre:  
   ```paup
   gettrees file=contraintes.nex;
   descr 1;
   descr 2;

   ```

3. Faire une recherche heuristique, mais en imposant une contrainte: vous obligez PAUP\* à trouver des 
topologies qui sont congruentes soit avec le premier arbre ou le deuxième arbre dans le fichier. Cela 
permet de déterminer quelle est la topologie la mieux supportée, mais qui est congruente avec une de ces 
deux hypothèses.  
   ```paup
   loadconstr file=contraintes.nex asBackBone=yes;
   hs constraint=contrainte1 enforce=yes;
   savetrees file=meilleurs_arbres_contraints.nex;
   contre / strict=yes majrule=no;
   
   ```

   - **Question**: En quoi ces arbres est différent du meilleur arbre sans contrainte?
   - **Question**: Quelle est la longueur de ces arbres en comparaison avec celle du meilleur sans 
   contrainte?

4. Il y a deux arbres de contraintes dans le fichier [`contraintes.nex`](fichiers/contraintes.nex). 
Déterminer quel serait le meilleur arbre qui suit la 2e contrainte:  
   ```paup
   loadconstr file=contraintes.nex;
   hs constraint=contrainte2 enforce=yes;
   savetrees file=meilleurs_arbres_contraints.nex append=yes;
   contre / strict=yes majrule=no;
   
   ```

   - **Question**: En quoi ces arbres sont différents des précédents et du meilleur arbre sans 
   contraintes?
   - **Question**: Quelle est la longueur de ces arbres en comparaison avec celle du meilleur sans 
   contrainte?

5. Faire une analyse sans la contrainte, et ajouter cet arbre au fichier 
`meilleurs_arbres_contraints.nex` qui contient les meilleurs arbres des recherches précédantes faites 
sous contrainte:  
   ```paup
   hs enforce=no;
   savetrees file=meilleurs_arbres_contraints.nex append=yes;
   
   ```

6. Utiliser le test KS basé sur la parcimonie pour déterminer si on peut exclure l'hypothèse que 
*Aus bus*, *Aus cus* , *Aus dus*, *Aus eus*, et *Aus fus* (hypothèse représentée par notre 
"contrainte1") forment un groupe monophylétique. Nous allons examiner tous les arbres sauvegardés 
dans le fichier `meilleurs_arbres_contraints.nex`, calculer score en parcimonie de ces arbres, et 
déterminer s'il y a une différence significative entre certains de ces arbres et les meilleurs arbres 
(valeur p).  
   ```paup
   gettrees file=meilleurs_arbres_contraints.nex allBlocks=yes;
   pscore / khtest=yes;
   
   ```

### Conclusion

Ce tutoriel vous a montré comment utiliser plusieurs indices de support et tests d'incongruence dans 
**PAUP\***, notamment le **bootstrap**, le **test ILD** et le **test KS** sous parcimonie . Ces outils 
permettent d'évaluer la robustesse des arbres phylogénétiques et de comparer différentes matrices de 
données et topologies pour en déterminer la meilleure selon les données disponibles.

- **Exercice**: Écrivez un script qui fait toutes les étapes en même temps, dans un ordre logique, en 
vous basant sur le code du tutoriel. Votre script doit charger le jeu de données, effectuer un test ILD, 
estimer le meilleur arbre et son support bootstrap, puis tester si le meilleur arbre est 
significativement différent des arbres sous contraintes.  

# Congruence et support avec PAUP*

Ce tutoriel montre comment utiliser PAUP\* pour calculer des indices de support (CI, RI, 
Bootstrap, Jackknife), faire des tests d'incongruence (ILD) et l'application de **tests KS** 
(Kishino-Hasegawa) avec le critère de parcimonie et des **tests AU** (Approximately Unbiased) avec 
le critère de maximum de vraisemblance sur PAUP\*.

---

### 1. Chargement des données dans PAUP*

#### Étape 1 : Charger la matrice de données

Ouvrez PAUP* et chargez le fichier Nexus `test.nex` avec la commande :

```bash
paup> execute test.nex;
```

Cela lira votre matrice de données et sera prêt à effectuer des analyses phylogénétiques.

---

### 2. Indices de support des arbres

#### Étape 2 : Calcul du Consistency Index (CI) et du Retention Index (RI)

Ces deux indices mesurent la quantité d'homoplasie dans un arbre.

- **CI** (Consistency Index) : Mesure l'efficacité des caractères à expliquer la variabilité phylogénétique. Un CI proche de 1 indique peu d'homoplasie.
- **RI** (Retention Index) : Mesure la proportion de synapomorphies par rapport aux homoplasies.

Pour calculer ces indices après une recherche d'arbre, exécutez une recherche heuristique et affichez les indices associés à l'arbre optimal :

```bash
paup> hsearch;
paup> describe /tree=1 stats=brlens;
```

Les valeurs de **CI** et **RI** apparaîtront dans la sortie de PAUP*.

---

#### Étape 3 : Bootstrap

Le **Bootstrap** est une méthode de ré-échantillonnage des caractères pour évaluer la robustesse des branches.

Pour effectuer une analyse bootstrap, utilisez la commande suivante :

```bash
paup> bootstrap nreps=100 search=heuristic;
```

Cela exécute un bootstrap avec 100 répétitions en utilisant une recherche heuristique.

---

### 3. Tests d'incongruence

#### Étape 4 : Test ILD (Incongruence Length Difference)

Le **test ILD** est utilisé pour vérifier l'incongruence entre plusieurs partitions de données. 

1. Créez les partitions dans votre fichier Nexus ou utilisez la commande :

   ```bash
   paup> charpartition mine = part1:1-500, part2:501-1000;
   ```

2. Ensuite, exécutez le test ILD avec la commande suivante :

   ```bash
   paup> hompart partition=mine nreps=100;
   ```

Le test ILD génère un **p-value**. Un p-value faible (< 0.05) indique une incongruence significative entre les partitions.

---

### 4. Test KS (Kishino-Hasegawa) avec le critère de parcimonie

Le **test KS** (Kishino-Hasegawa) est utilisé pour comparer des arbres en fonction du nombre de changements sous le critère de parcimonie.

#### Étape 5 : Comparer plusieurs arbres avec KS sous parcimonie

1. **Obtenez plusieurs arbres** à partir d'une recherche heuristique ou manuelle.
   ```bash
   paup> hsearch;
   ```

2. **Chargez les arbres** dans PAUP* pour comparaison :
   ```bash
   paup> gettrees file=alternative_trees.tre;
   ```

3. **Exécutez le test KS** sur ces arbres en utilisant le critère de parcimonie :
   ```bash
   paup> pscores /treefile=alternative_trees.tre kh=yes;
   ```

Le test **KS** génère des **p-values** pour comparer les arbres sous parcimonie. Si le p-value est faible (< 0.05), cela indique une différence significative entre les topologies d'arbres.

---

### 5. Test AU (Approximately Unbiased) avec le critère de vraisemblance

Le **test AU** est une méthode statistique pour comparer des topologies d'arbres en utilisant des log-vraisemblances. Il permet de tester si une ou plusieurs topologies sont significativement différentes de la meilleure topologie sous le critère de vraisemblance.

#### Étape 6 : Comparer plusieurs arbres avec le test AU

1. **Obtenez plusieurs arbres** à partir d'une recherche heuristique sous le critère de vraisemblance :
   ```bash
   paup> set criterion=likelihood;
   paup> hsearch;
   ```

2. **Chargez les arbres** dans PAUP* pour comparaison :
   ```bash
   paup> gettrees file=alternative_trees.tre;
   ```

3. **Calculez les scores de vraisemblance** pour ces arbres :
   ```bash
   paup> lscore all;
   ```

4. **Exécutez le test AU** en utilisant les vraisemblances :
   ```bash
   paup> pscores /treefile=alternative_trees.tre au=yes;
   ```

Le test **AU** vous donnera des **p-values** indiquant si les topologies alternatives sont statistiquement différentes de la meilleure topologie. Un p-value élevé indique que les arbres ne sont pas significativement différents, tandis qu'un p-value faible suggère qu'ils le sont.

---

### Conclusion

Ce tutoriel vous a montré comment utiliser plusieurs indices de support et tests d'incongruence dans **PAUP\***, notamment le **test KS** sous parcimonie et le **test AU** sous vraisemblance. Ces outils permettent d'évaluer la robustesse des arbres phylogénétiques et de comparer différentes topologies pour en déterminer la meilleure selon les données disponibles.
# Tutoriel : Analyse de parcimonie avec PAUP*

Ce tutoriel vous guidera à travers une analyse de parcimonie à l'aide du logiciel **PAUP***, en 
commençant par le transfert d'un fichier d'alignement au format Nexus vers un serveur de calcul, 
puis en explorant diverses commandes interactives sur PAUP*, telles que la sélection d'un hors 
groupe, l'exclusion ou l'inclusion de taxa, et la réalisation de recherches de type 
**branch-and-bound** et **heuristique**. Vous apprendrez également à évaluer la robustesse des 
arbres à l'aide du **bootstrap**.

---

## Transfert de l'alignement Nexus vers le serveur de calcul

**Transférez le fichier Nexus** depuis votre ordinateur local vers le serveur de calcul à l'aide 
de la commande `rsync`. Utilisez cette commande dans un terminal (sur votre ordinateur, pas le 
serveur):  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
CLUSTER_USERNAME=Votre_nom_utilisateur_serveur
LOCAL=/Chemin/vers/test.nex
REMOTE=/home/$CLUSTER_USERNAME

rsync --progress $LOCAL $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/
```

Notez que si vous êtes sur Windows, il faut remplacer les "backslash" (\\) dans votre chemin avec 
des "forward slash" (\/). De plus, le chemin "C:/" doit être remplacé par "/mnt/c/". Finalement, 
notez que les espaces dans les noms des dossiers et fichiers causent problème sur tous les 
terminal Linux/Mac. Il y a deux solutions pour les espaces: soit ne jamais utiliser d'espaces dans 
vos noms de fichier, ou soit entourer les espaces par des guillemets simples ('), ou précéder 
chaque espace par un "backslash" (\\) lorsque vous spécifiez un chemin vers un dossier ou fichier. 
Par exemple:  
- Le chemin Windows: `C:\Users\Moi\Documents\BIO 6245\ficher de travail.txt`  
- Doit être remplacé par: `/mnt/c/Users/Moi/Documents/BIO' '6245/fichier' 'de' 'travail.txt`  
- Ou bien par: `/mnt/c/Users/Moi/Documents/BIO\ 6245/fichier\ de\ travail.txt`


---

## Chargement du fichier d'alignement Nexus dans PAUP*

1. **Connectez-vous au serveur**.  

2. **Ouvrez le fichier Nexus** dans PAUP*:  
   ```
   /opt/paup4a168_centos64 test.nex
   
   ```

3. Vérifiez que les données sont bien chargées avec la commande:  
   ```
   showmatrix;
   
   ```
4. Regardez la liste des commandes en exécutant `?` dans PAUP*:
   ```
   ?;
   
   ```

5. Vous pouvez avoir les options d'une commande en particulier en exécutant le nom de la commande, 
suivi de `?`. Par exemple:  
   ```
   bandb ?;
   
   ```

6. Un manuel qui explique toutes les commandes est disponible 
[ici](https://phylosolutions.com/paup-documentation/paupmanual.pdf).

---

## Ajuster certains paramètres de base

Il est utile d'ajuster certains paramètres de base dans PAUP\*, qui permettent d'afficher les noms 
complets des taxa même lorsqu'ils sont longs, ainsi que d'augmenter la limite d'arbres qu'on 
permet de conserver en mémoire. Ces ajustements se font en exécutant ces commandes sur la ligne 
de commande de PAUP\*:  
```
set autoclose=yes;
set taxlabels=full;
set increase=auto autoinc=100 autoclose=yes;
set warnreset=no warntree=no;

```

## Ajustement du hors-groupe (outgroup)

Le hors-groupe est nécessaire pour enraciner l'arbre phylogénétique. Définir un ou plusieurs 
hors-groupes:  
```
outgroup Aus_aus;

```

---

## Recherche exhaustive par la méthode branch-and-bound

Une recherche exhaustive, comme celle effectuée avec la méthode **branch-and-bound**, explore 
toutes les combinaisons possibles de relations entre les taxa pour trouver l'arbre le plus 
parcimonieux. Cependant, le nombre de topologies d'arbres possibles augmente de manière plus 
qu'**exponentielle** avec l'ajout de nouveaux taxa, ce qui peut rendre la recherche extrêmement 
lente et parfois "infinie" pour de grands ensembles de données.

### Croissance rapide du nombre de topologies en fonction du nombre de taxa

Le nombre d'arbres phylogénétiques non racinés pour un nombre donné de taxa peut être calculé par 
la formule suivante:  

\[
T = \frac{(2n - 5)!}{(n-3)! \times 2^{n-3}}
\]

où **n** est le nombre de taxa, et **T** est le nombre total d'arbres non racinés possibles.

Pour donner une idée de cette croissance:  
- Avec 5 taxa, il y a **15** arbres possibles.  
- Avec 10 taxa, il y a **2 027 025** arbres possibles.  
- Avec 20 taxa, il y a environ **8,200,794,532,637,891,559** arbres.  

Ainsi, pour des ensembles de données comportant 20 ou plus de taxa, la méthode 
**branch-and-bound** peut prendre un temps **astronomique** pour explorer toutes les topologies 
possibles, et la recherche peut ne jamais finir.

Tentez une recherche branch-and-bound avec les 17 premiers taxa de l'alignement, mais en excluant 
les autres (pour ne pas trop surcharger le programme):  
```
delete all;
restore 1-17;

bandb;

```

- **Question**: Combien de temps est-ce que la recherche a pris à finir?  
- **Question**: Quel est le score du meilleur arbre trouvé?  

Tentez maintenant une recherche avec 18 taxa:
```
exclude all;
restore 1-18;

bandb;

```

**NOTEZ QUE VOUS POUVEZ EN TOUT TEMPS CESSER CETTE RECHERCHE**, vous n'avez qu'à appuyer sur 
Ctrl+C pour cesser la recherche du meilleur arbre avec l'algorithme branch-and-bound.

- **Question**: Qu'est-ce qui se passe ici?
- **Question**: Essayez avec tous les taxa. Combien de temps croyez-vous que ça va prendre?

---

## Recherche heuristique

Pour éviter le problème des approches exhaustives avec les alignements contenant de nombreux taxa, 
vous pouvez utiliser une méthode **heuristique**, qui est plus rapide mais approximative. Elle 
n'explore pas toutes les topologies d'arbres possibles, mais utilise des raccourcis algorithmiques 
pour trouver une solution proche de l'optimum. Ainsi, cette approche permet de rapidement avoir un 
arbre parcimonieux, mais **ne nous assure pas qu'il soit le plus parcimonieux d'entre tous les **
**arbres possibles**.

Restorez tous les taxa, pour déterminer le meilleur arbre phylogénétique entre les 26 taxa de 
l'alignement. Vous pouvez répondre oui (Yes) lorsque le programme demande s'il peut supprimer les 
arbres en mémoire.  
```
restore all;

```

Effectuer une recherche heuristique:  
```
hs addseq=random nreps=100 multre=yes nchuck=5 chucklen=1;

```

- **Question**: Combien de temps la recherche a-t-elle pris?  
- **Question**: Quel est le score des meilleurs arbres?  
- **Question**: Combien y a-t-il de meilleurs arbres?  

Examiner un des meilleurs arbres:  
```
descr 1;

```

- **Question**: Examinez un autre arbre parmi les meilleurs. Quelle est la commande pour le faire? 
En quoi cet autre arbre est différent?  

---

## Consensus strict

Étant donné qu'il existe un grand nombre d'arbres avec un même score de parcimonie, il est 
préférable de calculer un arbre de consensus strict pour regarder quelles branches ont du support 
avec la méthode de maximum de parcimonie. Voici comment faire:  
```
contre /majrule=no;

```

- **Question**: Combien d'arbres ont contribué à la création de ce consensus?

---

## Analyse de bootstrap

Le **bootstrap** est une méthode statistique permettant d'évaluer la robustesse des branches dans 
un arbre phylogénétique.

Effectuer une analyse de bootstrap:  
```bash
boots nreps=100 search=heuristic grpfreq=no / addseq=random nreps=3 multre=yes steepest=no nchuck=3 chucklen=1 limitperrep=yes;

```

- **Question**: Quelles sont les branches les mieux supportées? Lesquelles sont les moins bien 
supportées?

---

## Exercices

1. **Changer le hors-groupe** et ré-exécuter une recherche branch-and-bound. 
Comparez les résultats.  
2. **Exclure différents taxa** et observer comment cela affecte la topologie de l'arbre.
3. **Augmenter le nombre de ré-échantillonnages bootstrap** pour améliorer la précision de 
l'analyse.  
4. **Comparez les temps d'exécution** entre les recherches branch-and-bound et heuristiques pour 
différents nombres de taxa.  

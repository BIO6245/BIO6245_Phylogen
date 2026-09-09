# Alignement et ambiguïté

Le choix du placement des gaps (indels) lors de l'alignement de séquences a
 des conséquences majeures sur l'arbre phylogénétique qui en résulte. Cet
 exercice en deux parties montre d'abord un cas d'ambiguïté complète (où il
 est impossible de savoir quel alignement est « correct »), puis un cas où
 l'algorithme automatique produit un alignement de mauvaise qualité
 phylogénétique que vous pourrez corriger en cherchant l'arbre plus court.

---

## Partie 1: alignement complètement ambigü

Un alignement est **complètement ambigu** quand au moins deux placements de
 gap donnent un résultat final identique (même séquences alignées à la fin,
 même longueur totale) et qu'aucune autre information (structure secondaire,
 intron, contexte génomique) ne peut trancher. Dans ce cas, il n'existe aucun
 critère phylogénétique pour préférer l'un à l'autre.

### Étape préalable: construire le fichier manuellement

Voici les 5 séquences que nous allons utiliser (non alignées, avec des
 longueurs différentes) en format FASTA non aligné:

```
>Outgroup
CCTATTAGCCGACTCAGGAATT
>Taxon A
CCTATTAGAGCACTCAGGAATT
>Taxon B
CCTATTAGTGAAGCCAGGAATT
>Taxon C
CCTATTTGAGCCAGGAATT
>Taxon D
CCTATTTGAGCCAGGAATT
```

Ouvrez un éditeur de texte brut, copiez-collez le texte ci-dessus et
 sauvegardez le fichier sous le nom `fullamb_unaligned.fas`.

---

### Étape 1: importer le fichier non aligné dans MEGA

1. Ouvrez **MEGA**.
2. **File > Open a File/Session...** et sélectionnez `fullamb_unaligned.fas`.
3. Choisissez **Align** quand MEGA le demande.
4. L'Alignment Explorer affiche 5 séquences. Notez que les séquences de Taxon C
 et D sont plus courtes que les autres.


### Étape 2: aligner avec MUSCLE

1. Sélectionnez toutes les séquences (**Ctrl+A**).
2. Allez dans **Alignment > Align by MUSCLE** (ou ClustalW si MUSCLE n'est pas
 disponible). Cliquez sur OK.
3. MEGA lance l'algorithme et insère automatiquement 3 tirets dans les
 séquences des Taxon C et D pour créer un alignement où toutes les séquences
 ont la même longueur.

### Étape 3: sauvegarder l'alignement automatique

1. **Data > Export Alignment > FASTA Format**.
2. Sauvegardez sous `fullamb_auto.fas`.


### Étape 4 : construire un arbre de parcimonie pour l'alignement automatique

1. Fermez l'Alignment Explorer et ouvrez `fullamb_auto.fas` pour l'analyse
   (**File > Open a File/Session... > Analyze**).
2. Choisissez "No" quand **MEGA** demande si ces séquences codent pour des
   protéines.
3. Exécutez **Analysis > Phylogeny > Construct/Test Maximum Parsimony**
   **Tree...**. Sélectionnez "Yes" lorsqu'on demande s'il faut utiliser les
   données actives (fullamb_auto.fas).
4. Utilisez les paramètres par défaut (vérifiez notamment dans Gaps/Missing
   Data Treatment que l'option "Use all sites" est sélectionnée).
5. Cliquez sur **OK**. MEGA affiche le meilleur arbre.
6. Enracinez l'arbre en sélectionnant **Subtree > Root Tree** puis en cliquant
   sur la branche qui mène vers l'extra-groupe ("Outgroup").
5. Notez soigneusement :
    - La **longueur du meilleur arbre** (nombre de pas = nombre de mutations).
      Le chiffre est visible dans le coin en bas à gauche de la fenêtre.
    - La **topologie**: quel groupe de taxons se forme? Notamment, quel est le
      taxon le plus apparenté aux taxons C et D qui contiennent la délétion?

Sauvegardez une copie d'écran ou écrivez les valeurs exactes dans un bloc note.

### Étape 5: créer un alignement alternatif manuellement

Maintenant, vous allez créer un deuxième alignement du même jeu de séquences,
 en plaçant le gap ailleurs. L'objectif est de montrer que plusieurs
 alignements peuvent être valides pour les mêmes données brutes.

1. Ouvrez `fullamb_auto.fas` dans MEGA pour l'édition (**File > Open a
   File/Session... > Align**).
2. Repositionnez-manuellement le gap à une **position différente** qui minimise
   ne changera pas le score (longeur d'arbre = nombre de mutations) du meilleur
   arbre en parcimonie.
3. Sauvegardez cet alignement sous le nom de `fullamb_alt.fas`.
4. Réimportez ce nouvel alignement en mode "Analyze", puis faites un arbre de
   Parcimonie comme à l'étape 4.
5. Notez la longueur de l'arbre et la topologie. Ont-ils changés? Y a-t-il une
   raison de préférer un alignement plutôt qu'un autre? 
6. Est-ce que ces données permettent de dire quel taxon est le plus apparenté
   aux taxons C et D?
7. Si on avait jamais examiné l'alignement et on s'était fié uniquement à
   l'alignement automatisé, y aurait-il eu moyen de savoir que les données
   sont ambigues?


### Conclusion de la Partie 1: ambiguïté complète

Si les deux alignements produisent des arbres de **même score** et aucune autre
 information biologique n'est disponible, il est **impossible** de savoir
 lequel est correct. C'est un cas d'ambiguïté complète. Les deux placements de
 gap sont biologiquement plausibles et produisent des phylogénies très
 différentes, mais aucune n'est plus supportée par les données que l'autre.
 
Cela représente bien l'importance d'un bon alignement pour obtenir des
 résultats phylogénétiques fiables.

---

## Partie 2: alignement automatique suboptimal

### Concepts

Dans ce second cas, l'alignement automatique n'est pas la meilleure solution au
 sens de la parcimonie. Une modification manuelle de l'alignement produit un
 arbre avec un meilleur score (moins de mutations). Votre tâche est de trouver
 cette modification et de l'expliquer.

### Analyse 

1. Créez à l'aide d'un éditeur de texte brut le fichier FASTA non aligné
   ci-dessous nommé `subtle.fas`:  
   
```
>Outgroup
CCTATTAGCCGACTCAGGAATT
>Taxon A
CCTATTAGAGCACTCAGGAAGT
>Taxon B
CCTATTAGTGAAGCCAGGAATT
>Taxon C
CCTATTTGAGCCAGGAAGT
>Taxon D
CCTATTTGAGCCAGGAAGT
```

2. Comparez ces séquences à celles de `fullyamb.fas`. Remarquez-vous que la
 seule différence est une mutation supplémentaire en avant-dernière position
 chez les taxons A, C et D?
3. Alignez automatiquement ces séquences à l'aide de ClustalW ou MUSCLE dans 
   **MEGA**, puis sauvegardez le résultat sous `subtle_auto.fas`.
4. Faites un arbre de parcimonie et notez le score et la topologie.
5. Réouvrez le fichier, et tentez de créer un alignement alternatif qui
   améliorerait le score du meilleur arbre (alignement qui s'explique avec 
   moins de mutations).
6. Si vous ne trouvez pas la solution, demandez à votre prof de vous aider.
7. Pourquoi est-ce que les mutations ajoutées dans les taxons A, C et D
   permettent maintenant de résoudre l'ambiguïté de l'alignement, et pourquoi 
   les algorithmes automatisés ne trouvent pas toujours la solution optimale?

### Conclusion de la Partie 2 : Optimisation phylogénétique

Contrairement à la Partie 1 (où les deux alignements avaient la même longueur),
 la Partie 2 montre que **l'alignement automatique ne permet pas toujours**
 d'obtenir l'alignement optimal d'un point de vu phylogénétique. Ici, nous
 avons utilisé le score de parcimonie (nombre de mutations / longueur d'arbre),
 mais les mêmes principes sont applicables pour n'importe quel autre critère
 d'optimisation.

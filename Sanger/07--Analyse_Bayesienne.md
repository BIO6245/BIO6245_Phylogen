# Analyse phylogénétique Bayesienne

Ce tutoriel vous montrera comment faire une analyse phylogénétique Bayesienne simple en utilisant 
le programme **MrBayes**. Ce programme est facile d'utilisation et fonctionne bien. Il existe 
d'autres programmes plus poussés qui peuvent être utilisés pour faire des analyses plus complexes, 
tel que **BEAST** et **RevBayes**. Nous allons examiner l'utilisation de **BEAST** dans le 
tutoriel sur l'estimation des arbres d'espèce. Pour ce qui est de **RevBayes**, il s'agit d'un 
programme pour utilisateur un peu plus expérimenté, puisqu'il faut nous-même coder tous les 
paramètres des modèles.

## Analyse avec MrBayes

### Estimation phylogénétique avec MrBayes

1. **Ouvrir MrBayes**:
   - Lancez MrBayes en mode interactif à partir du serveur de calcul:
     ```bash
     mpirun.openmpi -np 2 /opt/MrBayes-mpi/src/mb-mpi
	 
     ```
   - Vérifiez que la console affiche le nom et la version du programme, ainsi que le nombre de 
   processeurs qui ont été alloués pour votre session de calcul (2, à cause de l'option `-np 2`).

2. **Chargement de l'alignement** :
   - Si vous avez déjà fait les tutoriels sur l'analyse de [parcimonie](Sanger/03--Parcimonie.md) 
   ou la [sélection de modèle et l'analyse de distance](Sanger/05--Selection_modele-distance.md), 
   vous devriez déjà avoir téléversé un alignement test sur le serveur de calcul. Si ce n'est pas 
   le cas, téléversez l'alignement [`test.nex`](fichiers/test.nex) sur le serveur. Vous pouvez 
   aussi utiliser l'alignement de séquences de *Dryopteris* généré dans le 
   [tutoriel sur Genbank](Sanger/01--Telecharger_donnees.md).
   
   - Chargez le fichier Nexus contenant votre alignement de séquences. Utilisez la commande 
   `execute` à partir de la ligne de commande de MrBayes:
     ```MrBayes
     execute test.nex
	 
     ```

3. **Définir les paramètres du modèle** :
   - La commande `lset` (pour "likelihood settings") permet de sélectionner la complexité du 
   modèle de substitution à estimer. La variable `nst` (pour "number of substitution types") 
   indique le nombre de paramètres à estimer. Pour un modèle GTR, il y a 6 paramètres à estimer 
   (A <-> C, A <-> G, A <-> T, C <-> G, C <-> T, et G <-> T). Les modèles HKY et K2P ont seulement 
   deux paramètres (`nst=2`) et JC et F81 ont un seul paramètre.
   
   Pour ajouter de la variation dans le taux d'évolution (+Gamma), indiquer `rates=gamma`, pour 
   estimer une proportion de sites invariables (+I) `rates=propinv`, et pour les deux (+Gamma+I) 
   `rates=invgamma`.
   
   Le modèle le plus complexe (GTR+Gamma+I) est donc sélectionné ainsi:
     ```MrBayes
	 lset applyto=(all) nst=6 rates=invgamma
	 
     ```
   
   - La commande `prset` (pour "prior settings") sert à ajuster les probabilités *a priori* sur 
   les paramètres. Pour empêcher l'estimation d'un paramètre, on peut donc mettre une probabilité 
   *a priori* de 1 pour la valeur du paramètre qu'on veut garder fixe.
   
   Ce genre de procédé est utilisé par exemple si on veut estimer un modèle d'évolution où tous 
   les nucléotides sont fixés comme étant en proportion égale (à l'équilibre). C'est le cas en 
   particulier du modèle Jukes-Cantor.
   
   Pour ce modèle, il faudrait donc exécuter ces commandes:
   ```MrBayes
   lset applyto=(all) nst=1 rates=equal
   prset applyto=(all) statefreqpr=fixed(equal)
   
   ```
   
   - Pour séparer notre alignement en deux régions, chacune associée à un modèle d'évolution 
   différent, il faut utiliser les commandes `charset` et `partition`. Par exemple:
   ```MrBayes
   charset codon1 = 1-20\3
   charset codon2 = 2-20\3
   charset codon3 = 3-20\3
   
   partition codon_1et2_vs_3 = 2: codon1 codon2, codon3
   
   set partition=codon_1et2_vs_3
   
   ```
   
   - Lorsqu'on définit plusieurs partitions, on veut généralement estimer séparément les 
   paramètres du modèle d'évolution de chaque partititon. Pour pouvoir faire ça dans 
   MrBayes, il faut *délier* l'estimation des paramètres des différentes partitions. On 
   fait ça avec la commande `unlink`. Exécuter la commande ci-dessous chaque vous que vous 
   estimez un arbre avec plus d'une seule partition:  
   ```MrBayes
   unlink revmat=(all) statef=(all) shape=(all)
   prset applyto=(all) ratepr=variable
   
   ```
   
   - Finalement, afficher le modèle avec la commande ci-dessous:
   ```MrBayes
   showmodel
   
   ```
   
   - Ajustez ces paramètres en fonction des besoins de votre analyse. Pour plus de détail sur les 
   options disponibles pour ce qui est des modèles d'évolution, référez-vous au 
   [manuel](https://github.com/NBISweden/MrBayes/blob/develop/doc/manual/Manual_MrBayes_v3.2.pdf).
   
   - Vous pouvez aussi lire l'aide à l'intérieur de MrBayes avec la commande `help`. Par exemple, 
   exécuter `help lset` vous donnera beaucoup d'informations sur les différents paramètres qu'on 
   peut ajuster au niveau du calcul de la vraisemblance.
   

4. **Exécution de la MCMC** :
   - Utiliser la commande `mcmcp` (pour "MCMC parameters") pour ajuster les paramètres de 
   l'estimation phylogénétique. À chaque `samp=1000` itérations de la chaîne, on conservera un 
   échantillon. On va sauvegarder les paramètres de chaque arbre échantilloné dans le fichier 
   `filename=test` et on va aussi sauvegarder les longueurs de branches (savebr=yes). On partira 
   un total de `nch=4` chaînes MCMC (1 normale, 3 "heated"). Les chaînes seront roulées pour un 
   total de 0.1 millions d'itérations (`ng=10000`). Notez que pour la majorité des analyses, 
   c'est préférable de viser au moins 1 million d'itérations ou plus.
     ```nexus
     mcmcp ng=100000 printf=1000 diagnf=1000 diagnst=maxstddev nch=4 savebr=yes filename=test
	 
     ```
   - Pour lancer l'analyse, simplement exécuter la commande:
   ```MrBayes
   mcmc
   
   ````
   
Une fois l'analyse lancée, vous allez pouvoir suivre le progrès à l'écran, car vous allez voir 
s'afficher la probabilité postérieure des arbres échantillonnés par la chaîne de Markov 
Monte-Carlo (MCMC). Ces probabilités devraient augmenter rapidement au début, puis se stabiliser.

Une fois l'analyse terminée, vous pouvez procéder avec la prochaine section.

### Analyse des résultats de MCMC dans MrBayes

1. **Résumé des paramètres avec `sump`** :
   - La commande `sump` permet de visualiser les statistiques des paramètres MCMC et de vérifier 
   la convergence. Assurez-vous que vos fichiers `.p` sont présents dans le même répertoire.
     ```MrBayes
     sump
	 
     ```

2. ** Vérification de la convergence des chaînes MCMC**:	 
   - Vérifiez les graphiques de l'évolution des paramètres pour repérer une stationnarité. Les 
   valeurs ESS (Effective Sample Size) doivent être suffisamment élevées (> 200) pour indiquer une 
   bonne convergence.
   - De plus, vérifiez le graphique des valeurs postérieures à travers le temps. Vous allez 
   probablement noter une augmentation initiale des valeurs, puis l'atteinte d'un plateau. Une fois 
   le plateau localisé, déterminé à quel génération (=itération) le plateau est atteint et 
   spécifiez cette valeur comme "burnin". Par exemple:
   ```MrBayes
   sump burnin=20000
   
   ```
   - **Question**: Est-ce que les chaînes MCMC semblent avoir convergé? Est-ce qu'elles ont roulé 
   suffisamment de temps pour obtenir un échantillon représentatif de la distribution postérieure?
   
4. **Résumé des arbres avec `sumt`** :
   - Utilisez `sumt` pour générer un arbre de consensus majoritaire basé sur les arbres 
   échantillonnés.
     ```bash
     sumt burnin=20000
	 
     ```
   - **Question**: que signifie les valeurs associées à chaque branche dans cet arbre?

Les fichiers de sortie de MrBayes sont:
- **`test.nex.con.tre`**: fichier contenant l'arbre consensus annoté avec les probabilités 
postérieures.
- **`test.nex.mcmc`**: informations détaillées sur l'exécution de la chaîne MCMC.
- **`test.nex.pstat`**: résumé des statistiques de la chaîne, incluant les ESS et les valeurs 
moyennes des paramètres.
- **`test.nex.run*.p`**: fichiers de compilation des valeurs des paramètres échantillonnés par 
chacune des deux chaînes MCMC (run1 et run2).
- **`test.nex.run*.t`**: fichiers de compilation des arbres échantillonnés par chacune des deux 
chaînes MCMC.

### Analyse des résultats de MCMC dans Tracer

**Tracer** est un outil puissant pour analyser les résultats de chaînes MCMC (Markov Chain Monte 
Carlo) et vérifier la convergence des paramètres.

Téléchargez le programme **[Tracer ici](https://beast.community/tracer)**.

1. **Préparation et importation des données**
   - Ouvrez **Tracer** et chargez vos fichiers de sortie `.p` (paramètres) de MrBayes en 
   utilisant l'option *File > Import Trace File*.
   - Si vous avez exécuté plusieurs chaînes (par exemple, `run1.p`, `run2.p`), importez-les 
   toutes pour une analyse comparative.

2. **Visualisation des Traces**
   - Une fois les fichiers importés, sélectionnez un paramètre spécifique (comme la 
   log-vraisemblance ou la proportion de branches).
   - Examinez les graphiques de trace pour évaluer la **stationnarité** de la chaîne. Une chaîne 
   bien convergée montre une courbe qui oscille de manière stable autour d'une valeur constante, 
   sans tendance notable.
   - Il faut examiner les traces des deux chaînes MCMC. Pour ce faire, sélectionner les deux 
   chaînes en maintenant la touche Ctrl enfoncée.

3. **Interprétation des résultats: stationnarité et convergence**
   - **Stationnarité** : Les graphiques doivent indiquer que la chaîne a atteint une zone de 
   probabilité stable après la période de « brûlage » initiale.
   - **Comparaison des chaînes** : Comparez plusieurs chaînes pour vous assurer qu'elles montrent 
   des comportements similaires. Des courbes similaires indiquent une bonne convergence 
   inter-chaînes.

4. **Valeurs ESS (Effective Sample Size)**
   - Dans la colonne **ESS** de Tracer, vérifiez les valeurs pour chaque paramètre. Une 
   ESS > 200 est généralement un signe que les chaînes ont suffisamment échantillonné l'espace de 
   probabilité, garantissant ainsi la fiabilité des moyennes estimées.
   - Si l'ESS est faible (< 200), cela peut indiquer une convergence insuffisante ou un 
   échantillonnage inefficace. 

5. **Histogrammes et distributions postérieures**
   - Consultez l'onglet *Marginal Density Plot* pour examiner la distribution des valeurs a 
   posteriori de chaque paramètre. Cela aide à comprendre la **forme** et la **distribution** des 
   estimations, ce qui est essentiel pour l'interprétation des modèles évolutifs.
   - Une distribution unimodale est un bon signe de convergence, tandis qu'une distribution 
   multimodale peut indiquer des problèmes de mélange ou des pics multiples de probabilité.

6. **Log-vraisemblance**
   - Le paramètre de **log-vraisemblance** doit montrer un plateau en fin de chaîne, suggérant 
   que la chaîne a atteint la zone la plus probable du paysage de vraisemblance.
   - Si la log-vraisemblance varie encore significativement, il se peut que la chaîne ait besoin 
   d'une durée d'exécution plus longue ou que des problèmes subsistent dans la configuration MCMC.

7. **Burn-in**
   - Tracer permet de spécifier un burn-in (par exemple, 25% des échantillons initiaux). 
   Assurez-vous d'ignorer les premiers échantillons avant que la chaîne n'atteigne la 
   stationnarité.
   - Le burn-in est ajusté dans la section *Burn-in* pour visualiser uniquement la partie 
   stationnaire de la chaîne.

8. **Analyse des paramètres de modèle**
   - Inspectez les estimations postérieures des paramètres de substitution ou des proportions de 
   sites invariables pour vérifier la concordance avec les modèles prédits.
   - Les valeurs de crédibilité (généralement les 95% a posteriori) doivent être cohérentes et 
   informatives pour soutenir vos conclusions sur les relations phylogénétiques.


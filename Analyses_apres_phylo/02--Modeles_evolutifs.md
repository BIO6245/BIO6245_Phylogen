# Modèles d'évolution des caractères et méthodes comparatives

Ce tutoriel fait suite au tutoriel sur la
 [reconstruction des états ancestraux](Analyse_apres_phylo/001-Etats_ancestraux.md).
 On y utilise  les mêmes données sur les anguilles, mais on passe de la question
 « quel était l'état de l'ancêtre? » à des questions sur les **processus**:

1. **Quel modèle décrit le mieux l'évolution d'un caractère continu?**
  (mouvement brownien, Ornstein-Uhlenbeck, *early burst*, etc.)
2. **Le mode d'évolution change-t-il dans l'arbre?** D'abord en testant une
  hypothèse précise (le mode d'alimentation), puis en cherchant les
  changements (*shifts*) sans hypothèse préalable.
3. **Deux caractères sont-ils corrélés** une fois la phylogénie prise en
  compte? (moindres carrés généralisés phylogénétiques, PGLS)
4. **Un caractère influence-t-il la diversification?** (modèles SSE)

Comme dans le tutoriel précédent, tout se fait dans **R**, sur votre
 ordinateur personnel. On suppose que vous connaissez déjà les bases de R
 vues dans ce tutoriel.

---

## Installation des paquets

En plus de **phytools**, on utilisera:

| Paquet | Utilisation |
|---|---|
| **geiger** | Ajuster des modèles d'évolution continus (`fitContinuous()`) |
| **OUwie** | Modèles BM et OU avec plusieurs régimes |
| **PhylogeneticEM** | Détection automatique des changements de régime |
| **nlme** | PGLS (installé d'office avec R) |
| **phylolm** | PGLS avec d'autres modèles d'évolution |
| **hisse** | Modèles de diversification BiSSE et HiSSE |

Pour installer tous ces paquets, exécuter une fois le code ci-dessous:  
```r
## À faire une seule fois (peut prendre quelques minutes)
install.packages(c("phytools", "geiger", "OUwie", "PhylogeneticEM",
                   "phylolm", "hisse"))
```

Chargez ensuite les paquets et les données:
```r
library(phytools)
library(geiger)
library(OUwie)
library(PhylogeneticEM)
library(nlme)
library(phylolm)
library(hisse)

## Données sur les anguilles (Collar et al. 2014), comme au tutoriel
## précédent. On met les lignes du tableau dans l'ordre de l'arbre.
data(eel.tree)
data(eel.data)
arbre <- eel.tree
donnees <- eel.data[arbre$tip.label, ]

## Les deux caractères, en vecteurs nommés
taille <- setNames(log(donnees$Max_TL_cm), rownames(donnees))
alim <- setNames(donnees$feed_mode, rownames(donnees))

## L'âge de la racine (en Ma), utile pour interpréter les paramètres
max(nodeHeights(arbre))
```

> **Remarque sur les résultats aléatoires**: plusieurs analyses ci-dessous
> utilisent des simulations ou des optimisations avec un point de départ
> aléatoire. Utilisez `set.seed()` avant chaque analyse pour obtenir les
> mêmes résultats d'une exécution à l'autre.

---

## Partie 1 – Comparer des modèles d'évolution

### Les modèles

On a vu le **mouvement brownien** (BM) au tutoriel précédent. Ce n'est qu'un
 modèle parmi d'autres, qui correspondent à des hypothèses biologiques
 différentes:

| Modèle | Paramètres | Hypothèse biologique |
|---|---|---|
| **BM** (*Brownian motion*) | σ² (taux), z₀ (valeur à la racine) | Dérive aléatoire, ou sélection qui change de direction au hasard; la variance augmente sans limite avec le temps. |
| **OU** (*Ornstein-Uhlenbeck*) | σ², z₀, **α** (force de rappel), θ (optimum) | Le caractère est « tiré » vers un optimum θ; souvent interprété comme de la sélection stabilisatrice. |
| **EB** (*early burst*) | σ², z₀, **a** (changement du taux) | Le taux d'évolution diminue avec le temps: beaucoup de changements tôt, puis un ralentissement (radiation adaptative). |
| **lambda** (λ de Pagel) | σ², z₀, **λ** | Brownien, mais avec un signal phylogénétique réduit (λ < 1). |
| **white** (bruit blanc) | σ², moyenne | Aucune structure phylogénétique: les espèces sont indépendantes. |

Pour le modèle OU, on interprète souvent α à l'aide de la **demi-vie**
 **phylogénétique**, ln(2)/α: le temps nécessaire pour que le caractère fasse
 la moitié du chemin vers l'optimum. Une demi-vie très courte par rapport à
 l'âge de l'arbre indique que le caractère « oublie » revient rapidement vers
 l'optimum, et donc que le signal phylogénétique est faible; une demi-vie très
 longue indique que le modèle se comporte presque comme un modèle brownien, 
 donc avec un signal phylogénétique fort.
 
On peut aussi interpréter α comme étant la force de sélection. Plus α est
 élevé, plus la force de sélection est forte. Moins il est élevé, plus la
 dérive génétique est importante par rapport à la sélection.

### Ajuster et comparer les modèles

La fonction `fitContinuous()` du paquet **geiger** ajuste chacun de ces
 modèles sur un arbre par maximum de vraisemblance:
```r
m.bm <- fitContinuous(arbre, taille, model = "BM")
m.ou <- fitContinuous(arbre, taille, model = "OU")
m.eb <- fitContinuous(arbre, taille, model = "EB")
m.lambda <- fitContinuous(arbre, taille, model = "lambda")
m.white <- fitContinuous(arbre, taille, model = "white")

## Afficher le résultat d'un modèle
m.ou
```

Les paramètres estimés sont dans `$opt`. On compare les modèles avec l'AICc
 (l'AIC corrigé pour les petits échantillons) et les **poids d'Akaike**, qui
 donnent le support relatif de chaque modèle (leur somme vaut 1):
```r
aicc <- c(BM = m.bm$opt$aicc, OU = m.ou$opt$aicc, EB = m.eb$opt$aicc,
          lambda = m.lambda$opt$aicc, white = m.white$opt$aicc)
aicw(aicc)

## Paramètres d'intérêt
m.ou$opt$alpha            # force de rappel
log(2) / m.ou$opt$alpha   # demi-vie phylogénétique (en Ma)
m.eb$opt$a                # changement du taux dans le modèle EB
m.lambda$opt$lambda       # lambda de Pagel
```

Un avertissement du type *Parameter estimates appear at bounds* signifie
 qu'un paramètre a atteint la limite permise lors de l'optimisation. Par
 exemple, un paramètre `a` de EB collé à zéro signifie que le modèle EB se
 réduit au BM: il n'y a aucune trace d'un ralentissement du taux.

- **Question**: Quel modèle est préféré pour la taille des anguilles? Le BM
 est-il un bon modèle pour ce caractère? Comparez la demi-vie du modèle OU à
 l'âge de la racine: qu'est-ce que cela signifie biologiquement?
- **Question**: Le modèle *white* (aucune structure phylogénétique) est-il
 loin derrière? Mettez ce résultat en relation avec le λ estimé et avec le
 test du signal phylogénétique du tutoriel précédent.

**Attention à l'interprétation du modèle OU**: avec peu d'espèces, les
 tests ont tendance à préférer le modèle OU à tort, et une erreur de mesure
 sur les caractères peut aussi mimer un modèle OU (Cooper et al. 2016). Un
 α élevé ne prouve donc pas l'existence d'une sélection stabilisatrice.

---

## Partie 2 – Changements de régime définis à l'avance

Les modèles de la partie 1 supposent un seul processus pour tout l'arbre.
 Mais le caractère pourrait évoluer différemment dans différents groupes.
 Collar et al. (2014) se demandaient si le **mode d'alimentation** (morsure
 ou succion) modifie l'évolution de la morphologie des anguilles. On peut
 tester cette hypothèse pour la taille: chaque mode d'alimentation est un
 **régime évolutif** qui influence comme la taille pourrait évoluer. Peut-être
 que différents modes d'alimentation influencent directement la taille des
 anguilles (modèles OU avec un optimum θ différent pour chaque type
 d'alimentation), ou peut-être qu'ils influencent la vitesse à laquelle la
 taille varie d'une espèce à l'autre (modèles BM avec une variance σ² 
 différente selon le mode d'alimentation).

Pour tester cette hypothèse, il faut:
  1. Déterminer tous les endroits dans la phylogénie où le mode d'alimentation
  change, à l'aide de **cartographie stochastique**.
  2. Pour chaque type d'alimentation différent, estimer les paramètres des
  modèles et déterminer si des modèles différents d'évolution de la taille sont
  fortement supportés lorsque l'alimentation change.
  3. Étant donné que la cartographie stochastique ne représente qu'une de
  multiples différentes histoires évolutives de l'alimentation qui sont
  possibles, répéter l'analyse plusieurs fois avec différentes cartographies
  stochastiques, pour voir si ça affecte nos résultats au niveau du mode
  d'évolution de la taille.

### Taux de BM différents selon le régime

La fonction `brownie.lite()` de phytools compare un BM à un seul taux et un
 BM avec un taux différent pour chaque régime (O'Meara et al. 2006):
```r
## Une histoire du mode d'alimentation (cartographie stochastique, comme
## dans le tutoriel précédent)
set.seed(1)
carte <- make.simmap(arbre, alim, model = "ER")

## Comparer un taux unique et un taux par régime
brownie.lite(carte, taille)
```

La sortie donne les deux modèles, leurs log-vraisemblances et un test du
 rapport de vraisemblance (*P-value*).
 
- **Question**: Le test est-il significatif? Qu'est-ce que ça signifie?

Roulez le code ci-dessus en changement la "graine" du générateur de nombres
 aléatoires. Est-ce que le résultat a changé (notamment la valeur p)?

### Modèles BM et OU à plusieurs régimes avec OUwie

Le paquet **OUwie** (Beaulieu et al. 2012) généralise cette idée au modèle
 OU. Les modèles les plus utilisés:

| Modèle | Description |
|---|---|
| **BM1** | Un seul BM pour tout l'arbre |
| **BMS** | Un BM avec un taux σ² différent par régime |
| **OU1** | Un seul OU, avec un seul optimum θ |
| **OUM** | Un OU avec un **optimum θ différent par régime** (même α et σ²) |
| **OUMV** | Un optimum θ **et** un taux σ² différents par régime |
| **OUMA** | Un optimum θ **et** une force de rappel α différents par régime |

OUwie demande un tableau à trois colonnes, dans cet ordre: le nom des
 espèces, le régime, puis la valeur du caractère:
```r
d.ouwie <- data.frame(espece = names(taille), alim = alim,
                      taille = taille)
head(d.ouwie)

## Ajuster les modèles sur l'histoire simulée plus haut
modeles <- c("BM1", "BMS", "OU1", "OUM", "OUMV")
ajust <- lapply(modeles, function(m)
  OUwie(carte, d.ouwie, model = m, simmap.tree = TRUE,
        algorithm = "invert", quiet = TRUE))
names(ajust) <- modeles

## Tableau de comparaison
comp <- data.frame(lnL = sapply(ajust, function(x) x$loglik),
                   AICc = sapply(ajust, function(x) x$AICc))
comp$delta <- comp$AICc - min(comp$AICc)
comp$poids <- round(exp(-comp$delta / 2) / sum(exp(-comp$delta / 2)), 3)
comp

## Paramètres du modèle OUM: alpha et sigma² ($solution),
## et l'optimum de chaque régime ($theta)
ajust$OUM$solution
ajust$OUM$theta
```

La fonction `lapply()` applique la même fonction à chaque élément d'un
 vecteur, ici chaque nom de modèle. C'est une façon compacte d'écrire cinq
 fois la commande `OUwie()`.

- **Question**: Les modèles à plusieurs régimes (BMS, OUM, OUMV) sont-ils
 mieux supportés que les modèles à un seul régime? Les optima θ des
 anguilles qui mordent et de celles qui aspirent leurs proies sont-ils
 différents, compte tenu de leurs erreurs-types (*standard-errors*, se)?

### Tenir compte de l'incertitude sur l'histoire des régimes

Le résultat précédent repose sur **une seule** histoire simulée du mode
 d'alimentation. Pour vérifier que la conclusion ne dépend pas de cette
 histoire en particulier, on répète l'analyse sur plusieurs cartes
 stochastiques:
```r
set.seed(123)
cartes <- make.simmap(arbre, alim, model = "ER", nsim = 10)

## Pour chaque carte, trouver le modèle avec le plus petit AICc
## (environ une minute)
meilleur <- sapply(cartes, function(une.carte) {
  aicc <- sapply(modeles, function(m)
    OUwie(une.carte, d.ouwie, model = m, simmap.tree = TRUE,
          algorithm = "invert", quiet = TRUE)$AICc)
  names(which.min(aicc))
})
table(meilleur)
```
Dans une vraie étude, on utiliserait plutôt 100 cartes ou plus, et on
 résumerait aussi les paramètres estimés sur l'ensemble des cartes.

---

## Partie 3 – Détecter les changements de régime sans a priori

Dans la partie 2, on a imposé l'emplacement des changements de régime (les
 changements de mode d'alimentation). On peut aussi poser la question
 inverse: **combien de changements de régime** y a-t-il dans l'arbre, et
 sur quelles branches?, sans hypothèse préalable.

Le paquet **PhylogeneticEM** (Bastide et al. 2017, 2018) ajuste un modèle
 OU (ou BM) avec 0, 1, 2… K changements de l'optimum, chacun placé sur la
 branche qui améliore le plus la vraisemblance. Il choisit ensuite le
 nombre de changements avec un **critère pénalisé**: la vraisemblance
 augmente **toujours** quand on ajoute des changements, il faut donc
 pénaliser la complexité du modèle pour ne pas en détecter partout.

```r
## PhyloEM demande une matrice: une ligne par caractère, une colonne
## par espèce (avec les noms d'espèces comme noms de colonnes)
Y <- matrix(taille, nrow = 1,
            dimnames = list("taille", names(taille)))
Y

## Modèle OU avec jusqu'à 10 changements de l'optimum
## ("scOU": OU à une seule force de rappel; environ une minute)
em.ou <- PhyloEM(phylo = arbre, Y_data = Y, process = "scOU",
                 K_max = 10, random.root = TRUE,
                 stationary.root = TRUE, progress.bar = FALSE)

## Résumé du modèle choisi (nombre de changements sélectionné)
em.ou

## Arbre avec les changements sélectionnés
plot(em.ou)

## Le critère de sélection en fonction du nombre de changements K
plot_criterion(em.ou)
```

PhyloEM peut afficher des avertissements sur une petite différence de
 log-vraisemblance entre l'arbre original et l'arbre transformé: ils sont
 sans conséquence ici.

On peut aussi examiner les solutions avec un nombre fixe de changements, par
 exemple 3, même si ce nombre n'est pas celui qui est sélectionné:
```r
## Branches et valeurs des changements pour K = 3
params_process(em.ou, K = 3)$shifts
plot(em.ou, params = params_process(em.ou, K = 3))

## Log-vraisemblance pour chaque nombre de changements
em.ou$alpha_max$results_summary[, c("K_try", "log_likelihood")]
```

La même analyse avec un modèle brownien, où les changements portent sur la
 moyenne du caractère plutôt que sur l'optimum:
```r
em.bm <- PhyloEM(phylo = arbre, Y_data = Y, process = "BM",
                 K_max = 10, progress.bar = FALSE)
em.bm
```

- **Question**: Combien de changements de régime sont sélectionnés pour la
  taille? Comment la log-vraisemblance évolue-t-elle quand K augmente, et
  pourquoi ne peut-on pas simplement choisir le K qui a la meilleure
  vraisemblance?
- **Question**: Avec K = 3, sur quelles branches les changements sont-ils
  placés? Correspondent-ils à des changements du mode d'alimentation?
  Comparez avec les conclusions de la partie 2.

D'autres méthodes existent pour détecter les changements de régime,
 par exemple **l1ou** (LASSO; Khabbazian et al. 2016) ou **bayou**
 (approche bayésienne; Uyeda et Harmon 2014). Elles ne donnent pas toujours
 le même nombre de changements: c'est un domaine de recherche encore très actif.

---

## Partie 4 – PGLS: régression en tenant compte de la phylogénie

### Pourquoi une régression phylogénétique?

Une régression ordinaire (moindres carrés, OLS) suppose que les observations
 sont **indépendantes**. Ce n'est pas le cas pour des espèces: deux espèces
 soeurs se ressemblent en partie parce qu'elles ont hérité des mêmes
 caractères de leur ancêtre commun. La régression par **moindres carrés** **généralisés phylogénétiques** (PGLS) intègre cette non-indépendance: elle
 suppose que les **résidus** de la régression sont corrélés selon la phylogénie,
 sous un modèle d'évolution (BM, λ de Pagel, OU…).

Ici, on teste si la taille diffère entre les anguilles qui mordent et
 celles qui aspirent leurs proies. Avec une variable explicative
 catégorielle, la PGLS est l'équivalent phylogénétique d'une ANOVA. La
 démarche serait exactement la même si on avait une variable explicative
 continue.

### PGLS avec nlme

La fonction `gls()` du paquet **nlme** accepte une structure de corrélation
 phylogénétique, fournie par **ape**:
```r
## Ajouter une colonne avec le nom des espèces (utilisée par form = ~espece)
## et la taille transformée en logarithme
donnees$espece <- rownames(donnees)
donnees$log_taille <- log(donnees$Max_TL_cm)

## 1. Régression ordinaire (aucune phylogénie)
m.ols <- gls(log_taille ~ feed_mode, data = donnees, method = "ML")
summary(m.ols)

## 2. PGLS sous mouvement brownien
m.pgls.bm <- gls(log_taille ~ feed_mode, data = donnees,
                 correlation = corBrownian(1, phy = arbre, form = ~espece),
                 method = "ML")
summary(m.pgls.bm)

## 3. PGLS avec lambda de Pagel estimé (valeur de départ 0.5)
m.pgls.lambda <- gls(log_taille ~ feed_mode, data = donnees,
                     correlation = corPagel(0.5, phy = arbre,
                                            form = ~espece, fixed = FALSE),
                     method = "ML")
summary(m.pgls.lambda)

## Comparer les trois modèles
AIC(m.ols, m.pgls.bm, m.pgls.lambda)

## Valeur de lambda estimée
m.pgls.lambda$modelStruct$corStruct
```

**Note**: si `corPagel()` donne l'erreur `NA/NaN/Inf in foreign function call`,
 essayez une autre valeur de départ pour lambda (par ex. 0.5 plutôt que 1).

Dans `summary()`, le coefficient `feed_modesuction` est la différence
 moyenne de log(taille) entre les anguilles qui aspirent et celles qui
 mordent, et sa *p-value* teste si cette différence est nulle.

### PGLS avec phylolm

Le paquet **phylolm** fait la même chose, plus rapidement, et offre d'autres
 modèles pour les résidus (par ex. OU):
```r
m.pgls.ou <- phylolm(log_taille ~ feed_mode, data = donnees, phy = arbre,
                     model = "OUfixedRoot")
summary(m.pgls.ou)
```

- **Question**: Comparez le coefficient `feed_modesuction` et sa *p-value*
 entre la régression ordinaire et les PGLS. La conclusion change-t-elle?
 Quel modèle des résidus est le mieux supporté selon l'AIC?
- **Question**: Pourquoi le λ estimé par la PGLS (sur les résidus) n'est-il
 pas forcément égal au λ estimé sur la taille seule dans la partie 1?

---

## Partie 5 – Modèles SSE: un caractère influence-t-il la diversification?

### Le principe

Les modèles **SSE** (*State-dependent Speciation and Extinction*) estiment
 en même temps l'évolution d'un caractère et les taux de spéciation (λ) et
 d'extinction (μ) associés à chaque état de caractère. Le modèle **BiSSE**
 (Maddison et al. 2007) le fait pour un caractère binaire: les anguilles qui
 aspirent leurs proies se diversifient-elles plus vite que celles qui mordent?

Attention! Des problèmes importants ont été mis en évidence avec ces modèles:

1. **Faux positifs**: BiSSE détecte très souvent un effet du caractère alors
 qu'il n'y en a pas, simplement parce que les taux de diversification
 varient dans l'arbre pour d'autres raisons (Rabosky et Goldberg 2015). Il
 faut donc comparer BiSSE à des modèles nuls appropriés: les modèles
 **CID** (*character-independent diversification*), où les taux varient
 dans l'arbre, mais indépendamment du caractère étudié.
2. **Puissance**: il faut des arbres de plusieurs centaines d'espèces pour
 estimer ces modèles de façon fiable (Davis et al. 2013). Avec nos
 58 espèces d'anguilles, l'analyse est **purement illustrative**.

Le paquet **hisse** (Beaulieu et O'Meara 2016) permet d'ajuster BiSSE, les
 modèles CID et le modèle **HiSSE**, qui ajoute un état « caché » pour
 représenter les autres facteurs pouvant influencer la diversification.
 Plutôt que λ et μ, hisse estime le **renouvellement** (*turnover*, λ + μ)
 et la **fraction d'extinction** (*eps*, μ/λ). C'est la meilleure façon de
 procéder si on veut bien faire ce type d'analyse.

### Préparer les données

L'arbre contient quelques groupes externes qui ne sont pas des anguilles
 (genres *Albula*, *Elops* et *Megalops*). On les retire, puisqu'on
 s'intéresse à la diversification des anguilles (Anguilliformes)
 seulement:
```r
## Retirer les groupes externes
externes <- grep("^(Albula|Elops|Megalops)_", arbre$tip.label,
                 value = TRUE)
externes
arbre.sse <- drop.tip(arbre, externes)
Ntip(arbre.sse)

## Un tableau: nom des espèces, puis l'état (0 = morsure, 1 = succion)
etat <- data.frame(espece = arbre.sse$tip.label,
                   alim = ifelse(donnees[arbre.sse$tip.label,
                                         "feed_mode"] == "suction", 1, 0))
table(etat$alim)

## Fraction des espèces échantillonnées dans chaque état. Les anguilles
## comptent de l'ordre de 1000 espèces, dont environ 6 % sont dans
## l'arbre (estimation approximative, la même pour les deux états).
f <- c(0.06, 0.06)
```

La fraction échantillonnée (`f`) est essentielle: un arbre qui ne contient
 que 6% des espèces paraîtra avoir très peu de spéciations récentes si on ne
 corrige pas pour l'échantillonnage.

### Ajuster et comparer quatre modèles

Les arguments `turnover` et `eps` indiquent quels paramètres sont estimés
 séparément: deux états avec la même valeur (par ex. `c(1, 1)`) partagent le
 même paramètre, des valeurs différentes (`c(1, 2)`) ont chacun le leur.
```r
## 1. Modèle nul: diversification identique pour les deux états
m.nul <- hisse(arbre.sse, etat, f = f, turnover = c(1, 1), eps = c(1, 1),
               hidden.states = FALSE,
               trans.rate = TransMatMakerHiSSE(hidden.traits = 0),
               sann = FALSE)

## 2. BiSSE: renouvellement différent selon le mode d'alimentation
m.bisse <- hisse(arbre.sse, etat, f = f, turnover = c(1, 2), eps = c(1, 1),
                 hidden.states = FALSE,
                 trans.rate = TransMatMakerHiSSE(hidden.traits = 0),
                 sann = FALSE)

## 3. CID-2: deux régimes de diversification cachés, indépendants du
##    mode d'alimentation (modèle nul de HiSSE)
m.cid2 <- hisse(arbre.sse, etat, f = f, turnover = c(1, 1, 2, 2),
                eps = c(1, 1, 1, 1), hidden.states = TRUE,
                trans.rate = TransMatMakerHiSSE(hidden.traits = 1,
                                                make.null = TRUE),
                sann = FALSE)

## 4. HiSSE: diversification qui dépend du mode d'alimentation ET d'un
##    état caché
m.hisse <- hisse(arbre.sse, etat, f = f, turnover = c(1, 2, 3, 4),
                 eps = c(1, 1, 1, 1), hidden.states = TRUE,
                 trans.rate = TransMatMakerHiSSE(hidden.traits = 1),
                 sann = FALSE)
```
Ces quatre analyses prennent une dizaine de minutes au total (HiSSE est le
 plus long). L'option `sann = FALSE` désactive une étape d'optimisation
 qui aide à trouver le maximum de vraisemblance, mais qui est beaucoup plus
 lente. Pour une vraie étude, gardez l'option par défaut (`sann = TRUE`).

```r
## Les paramètres estimés pour BiSSE
m.bisse

## Ceux pour HiSSE
m.hisse

## Comparer les quatre modèles
mods <- list(nul = m.nul, BiSSE = m.bisse, CID2 = m.cid2,
             HiSSE = m.hisse)
comp.sse <- data.frame(lnL = sapply(mods, function(x) x$loglik),
                       AICc = sapply(mods, function(x) x$AICc))
comp.sse$poids <- round(GetAICWeights(mods, criterion = "AICc"), 3)
comp.sse
```

- **Question**: BiSSE est-il mieux supporté que le modèle nul? Et que le
 modèle CID-2? Que pouvez-vous conclure (ou ne pas conclure) sur l'effet du
 mode d'alimentation sur la diversification des anguilles?
- **Question**: Pourquoi est-il trompeur de comparer seulement BiSSE au
 modèle nul à un seul taux de diversification?
- **Question**: Le modèle CID-2 contient le modèle nul comme cas
 particulier: sa log-vraisemblance devrait donc être au moins aussi élevée.
 Est-ce le cas dans vos résultats? Si non, qu'est-ce que cela indique sur
 l'optimisation, et que devriez-vous faire?

D'autres modèles de la même famille existent: **MuSSE/MuHiSSE** (caractères à
 plus de deux états, ou plusieurs caractères binaires), **QuaSSE** (caractère
 continu), **GeoSSE/GeoHiSSE** (aires géographiques) et **MiSSE** (taux
 variables sans caractère). Tous cherchent à estimer l'effet conjoint de
 différentes sortes de variables sur les taux de diversification (et 
 vice-versa).

---

## Questions et exercices

1. **Simuler l'évolution de caractères**:
   - Simulez un caractère sous un mouvement brownien sur l'arbre des
   anguilles avec `x <- fastBM(arbre)`, puis ajustez les modèles BM et OU
   avec `fitContinuous()`. Répétez une vingtaine de fois (avec une boucle for). 
   Dans quelle proportion des cas le modèle OU est-il préféré à tort selon
   l'AICc? Qu'est-ce que cela implique pour l'interprétation de la partie 1?

2. **Robustesse**:
   - Refaites les analyses OUwie sur 10 cartes stochastiques produites avec
   le modèle `"ARD"` plutôt que `"ER"`. Les conclusions changent-elles?
   - Ajoutez le modèle `"OUMA"` à la liste des modèles. Est-il mieux
   supporté?

3. **Méthodes avec et sans a priori**:
   - En quelques phrases, comparez les approches des parties 2 et 3. Quels
   sont les avantages et les risques de chacune?

4. **PGLS et causalité**:
   - Une PGLS significative entre deux caractères prouve-t-elle que l'un
   cause l'autre?

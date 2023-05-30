# Rapport | Projet Math Info S6
- Sujet : Logique CTL
- Encadrant : François Laroussini
- Membres : 
    - GAM Stani
    - BERRICHE Aaron

# Plan
- Introduction
    - Présentation du Model Checking
    - Introduction aux Logiques Temporelles
- Présentation de la Logique CTL
    - Syntaxe
    - Sémantique
    - Syntaxe alternative
    - Règles d'équivalences
    - Expressivité
- Algorithme d'évaluation CTL
    - Principe général
    - Opérateurs CTL
        - Opérateurs minimaux
        - Sucre syntaxique
    - AFG, EGF
    - E[ $\varphi$ $G$% $n$ ]
- Outil développé
    - Parser CTL, LTL, SL $\to$ Représentation abstraite
    - Sauvegarde / Lecture des structures de Kripke
    - Evaluation des formules CTL classiques
    - Enrichissement de la syntaxe
- Déroulement du projet
- Références

# Introduction

## Présentation du Model Checking
---
De manière informelle, la vérification de modèles consiste à vérifier que le modèle d'un système obéit à un certains nombres de propriétés. Les structures de Kripke sont une variante des systèmes de transition permettant de modéliser des systèmes dynamiques à nombres d'états finis et temps discret.

Formellement : \
Soient, $AP$ un ensemble de propositions atomiques. \
Alors une structure de Kripke est un quadruplet $M = (S, I, R, L)$
- $S$ un ensemble dont les éléments représentent des états
- $I \subset S$ un ensemble d'états initiaux
- $R \subset S\times S$ un ensemble de transitions tel que $R$ soit totale à gauche.
    - Rq : La totalité à gauche implique l'inexistence d'état final.
- $L : S \to 2^{AP}$ une fonction de labélisation

La ressource utilisé dans la production de ce rapport s'intèressent aux systèmes de transitions à états labélisés, les structures de Kripke sont un cas particulier de tels structures.

De manière générale, les propriétés évaluées sur les structures de Kripke concerneront ses états. On pourra donc définir pour une formule $\phi$ donnée, $Sat_\phi$ l'ensemble des états qui vérifient la formule $\phi$. On dira alors que notre structure vérifie la formule si ses états initiaux la vérifie : $I \subseteq Sat_\phi$.

Deux formules $\phi$, $\psi$ seront dites équivalentes sur le systèmes de transition si elles sont satisfaites sur les même états. Les deux définitions suivantes sont équivalentes : 
- $\phi \equiv_{TS} \psi :=\quad$ $TS \models \phi \iff TS \models \psi$ 
- $\phi \equiv_{TS} \psi :=\quad$ $Sat_\phi = Sat_\psi$


## Introduction aux Logiques Temporelles
---
De manière informelle, les logiques temporelles permettent d'énoncer des propriétés prenant compte de l'évolution du 'monde'. Dans le cadre du model checking, il faut comprendre par évolution du monde, évolution des états.

Typiquement, une logique temporelle permettra de définir pour propriété $\phi$ une formule qui vérifie $\phi$ pour tout les états à venir du système de transition.

Le $CTL$*, $LTL$ et le $CTL$ ont été sémantiquement définies pour pouvoir traiter des états d'un système de transitions.

# Présentation de la Logique CTL
La logique $CTL$ (Computational Tree Logic) permet d'énoncer des propriétés sur des arbres d'executions. 

A l'inverse de la logique à temps linéaire ($LTL$), les chemins d'executions peuvent être multiples, la logique $CTL$ permet donc d'encapsuler l'étude de systèmes non déterministes. 

Pour simplifier les algorithmes d'évaluation de structure nous n'étudierons que les arbres d'executions repliés sous la forme de structures de Kripke. Par conséquent nous définiront la sémantique uniquement sur les structures de Kripke.

## Syntaxe
---
Les formules CTL sont définies suivant la grammaire non contextuelles suivante :

$\phi ::= true \mid a \mid \phi_1 \land \phi_2 \mid \lnot \phi \mid \exists \varphi \mid \forall \varphi $ \
$\varphi ::= \bigcirc \phi \mid \phi_1 \bigcup \phi_2 $

Une formule CTL est toujours une formule d'état. \
Ici $\phi$ est une formule d'état, $\varphi$ une formule de chemin.

## Sémantique
---
Soit $K$ une structure de Kripke. \
On définit la relation de satisfaction $\models$ pour les états par :
- $s \models a \,\,\,\,\qquad ssi \quad a \in L(s)$
- $s \models \lnot \phi \qquad ssi \quad non \, s \models \phi$
- $s \models \phi \land \psi \quad ssi \quad (s\models \phi) \,et\, (s\models\psi)$
- $s \models \exists \varphi \qquad ssi \quad\pi \models \varphi$ pour un chemin $\pi \in Paths(s)$
- $s \models \forall \varphi \qquad ssi \quad\pi \models \varphi$ pour tout chemin $\pi \in Paths(s)$

On définit la relation de satisfaction $\models$ pour les chemins par :
- $\pi \models \bigcirc \phi \qquad ssi \qquad \phi[1] \models \phi$
- $\pi \models \phi \bigcup \psi \quad ssi \qquad \exists j \geq 0 \bigg(\pi[j] \models \psi \land (\forall 0 \leq k < j \, \pi[k] \models \phi)\bigg)$

Il faut comprendre un chemin $\pi$ comme $\pi = s_0s_1s_2...$ où $\pi[i]$ est le $i^e$ élément du chemin.

De plus, en langage naturel il faut comprendre :
- $\bigcirc \phi$ comme "$\phi$ sera vérifié au prochain état"
- $\phi \bigcup \psi$ comme "l'état présent ou un état future vérifiera $\psi$ et tout les états en attendant $\psi$ vérifieront $\psi$"

## Opérateurs supplémentaires
---
En se s'inspirant du LTL, on peut vouloir définir des opérateurs supplémentaires. Il est prouvé pour les opérateurs suivant qu'ils sont bien définis en CTL :

Les opérateurs supplémentaires de chemins :
- $\lozenge\phi := true \bigcup \phi$
    - Il existe sur le chemin un état qui vérifie $\phi$
- $\square \phi := \forall i \,\, \pi[i] \models \phi$
    - Tout les états du chemin vérifient $\phi$
    - On peut montrer que celui ci est bien définie pour chaque quantifieur de chemin :
        - $\exists \square \phi$ $= \lnot\forall\lozenge\lnot\phi$
        - $\forall \square \phi$ $= \lnot\exists\lozenge\lnot\phi$

Les opérateurs d'état supplémentaires :
- $EX := \exists(\bigcirc\phi)$
    - Un état directement accessible vérifie $\phi$
- $AX := \forall(\bigcirc\phi)$
    - Tout les états directement accessibles vérifient $\phi$
- $EF := \exists(\lozenge\phi)$
    - Il existe un état future ou présent qui vérifie $\phi$
- $AF := \forall(\lozenge\phi)$
    - Pour tout chemin il existe un état future ou présent vérifie $\phi$
- $EG := \exists(\square\phi)$
    - Il un chemin qui vérifie partout $\phi$
- $AG := \forall(\square\phi)$
    - Pour tout chemin, tout les états vont vérifier $\phi$
- $E[\phi U \psi] := \exists(\phi \bigcup \psi)$
    - Il existe un chemin tel que qu'un de ses états vérifie $\psi$ et ceux le préçédant vérifient $\phi$
- $A[\phi U \psi] := \forall(\phi \bigcup \psi)$
    - Pour tout un de ses états vérifie $\psi$ et ceux le préçédant vérifient $\phi$

Et de manière optionnelle :
- $E[\phi W \psi] := E[\phi U \psi] \lor EG \phi$
- $A[\phi W \psi] := \lnot\exists\bigg((\phi\land\lnot\psi)\bigcup(\lnot\phi\land\lnot\psi)\bigg)$
- $E[\phi$ R $\psi] := \lnot \forall\bigg[ \lnot \phi \bigcup \lnot \psi \bigg]$
- $A[\phi$ R $\psi] := \lnot \exists\bigg[ \lnot \phi \bigcup \lnot \psi \bigg]$

- $EFG \phi := EF \, EG \phi$
- $AGF \phi := AG \, AF \phi$

De plus nous avons décidé d'inventer un opérateur pour des raisons pratiques:
- A[ $\cdot$ FROM $\cdot$ TO $\cdot$ ] et E[ $\cdot$ FROM $\cdot$ TO $\cdot$ ]
    - *A[ behavior FROM start_condition TO end_condition ] :=* \
    *AG[ start_condition -> AX A[behavior U end_condition] ]*
    - *E[ behavior FROM start_condition TO end_condition ] :=* \
    *EG[ start_condition -> AX A[behavior U end_condition] ]*
        

Les opérateurs ci-dessus ne font pas partie de la logique CTL :
- $AFG \phi := \quad$ $\forall \pi  \, \exists j$ tq. $\forall i \geq j : \pi[i] \models \phi$
- $EGF \phi := \quad$ $\exists \pi \, \forall j$ tq. $\exists i \geq j : \pi[i] \models \phi$
- E[ $\varphi$ G% $n$ ] $:= \quad$ $\exists \pi$   tq.  $\forall k$,  $i=nk \implies s_i \models \varphi$

## Syntaxe alternatives
---
Les syntaxes ci-dessous sont complètes. Elles ont la même expressivité que la syntaxe standart du CTL. En d'autre terme pour chaque formule dans l'une des syntaxes, il existe une formule dans l'autre qui est sémantiquement équivalente.

### Forme Normale Positive (PNF)
$\phi ::= true \mid a \mid \lnot \phi \mid \phi_1 \land \phi_2 \mid \phi_1 \lor \phi_2 \mid \exists \varphi \mid \forall \varphi $ \
$\varphi ::= \bigcirc \phi \mid \phi_1 \bigcup \phi_2 \mid \phi_1 W \phi_2$

### Forme Normale Existentielle (ENF)
$\phi ::= true \mid a \mid \lnot \phi \mid \phi_1 \land \phi_2 \mid \exists \bigcirc \phi \mid \exists(\phi_1 \bigcup \phi_2 ) \mid \exists\square \phi$

L'avantage de formaliser ces syntaxes minimales est d'être sur que pour toute formule CTL, notre algorithme d'évaluation possède une version de la formule dans sa syntaxe.

Sur ordinateur, on préféra utiliser un autre alphabet et ajouter du sucre syntaxique par dessus la syntaxe ENF : \
$\phi ::=$ \
$false \mid true \mid p\mid (!\phi )\mid (\phi\, \&\&\, \phi )\mid (\phi \| \phi )\mid (\phi $->$ \phi ) \mid$ \
$AX \phi \mid EX \phi \mid AF \phi \mid EF \phi \mid AG \phi \mid EG \phi \mid A[\phi U \phi ]\mid E[\phi  U\phi ]$


## Règles d'équivalences
---
### Lois de dualité
- $\forall \bigcirc \phi \equiv \lnot \exists \bigcirc \lnot \phi$
- $\exists \bigcirc \phi \equiv \lnot \forall \bigcirc \lnot \phi$
- $\forall \lozenge \phi \equiv \lnot \exists \square \lnot \phi$
- $\exists \lozenge \phi \equiv \lnot \forall \square \lnot \phi$
- $\forall(\phi \bigcup \psi) \equiv \lnot \exists ((\phi \land \lnot \psi) W (\lnot \phi \land \lnot \psi))$
### Lois d'expansion
- $\forall(\phi \bigcup \psi) \equiv \psi \lor (\phi \land \forall \bigcirc \forall(\phi \bigcup \psi))$ 
- $\forall \lozenge \phi \equiv \phi \lor \forall \bigcirc \forall \lozenge \phi$
- $\forall \square \phi \equiv \phi \land \forall\bigcirc \forall \square \phi$

- $\exists(\phi \bigcup \psi) \equiv \psi \lor (\phi \land \exists \bigcirc \exists(\phi \bigcup \psi))$
- $\exists \lozenge \phi \equiv \phi \lor \exists \bigcirc \exists \lozenge \phi$
- $\exists \square \phi \equiv \phi \land \exists \bigcirc \exists \square \phi$

### Distributivité
- $\forall \square(\phi \land \psi) \equiv \forall \square \phi \land \forall \square \psi$
- $\exists \lozenge (\phi \lor \psi) \equiv \exists \lozenge \phi \lor \exists \lozenge \psi$

## Expressivité
---
Le LTL est similaire au CTL mais se restreint à un unique chemins d'execution, il n'y a donc pas d'ambiguité sur le chemin considèré dans la construction de formule. Les quantificateurs $\forall$ et $\exists$ n'ont plus de raison d'être.

Une syntaxe minimale du LTL est la suivante : \
$\phi ::= \lnot( \phi \land \psi) \mid \square\phi \mid \lozenge\phi \mid [\phi_1 U \phi_2] \mid \bigcirc \phi$

Le théorème 6.18 de "principles of model checking" nous montre que pour toute formule CTL, si il existe un équivalent LTL, alors celui est nécessairement obtenu en supprimant les quantifieurs de chemins.

De plus il existe des formules LTL sans équivalent CTL :
-  $\lozenge \square a$
- $\lozenge(a\land\bigcirc a)$

Et evidemment il existe des formules CTL sans équivalent LTL :
- $\forall \lozenge \forall \square a$
- $\forall \lozenge (a \land \forall \bigcirc a)$
- $\forall \square \exists \lozenge a$

De manière général : (CTL $\bigcup$ LTL) $\subsetneq$ CTL* $\subsetneq$ $\mu$-calcule

# Algorithme d'évaluation CTL
## Principe général
---
Le principe générale derrière l'algorithme d'évaluation de formules CTL est de construire recursivement $Sat_f$ pour chaque sous formule $f$ de la formule évalué.

### Exemple :
On souhaite construire $Sat_f$ :\
f = $a \land (b \lor(AF \, c))$ \
f = $f_1 \land f_2\quad$  tq. $f_1 = a$ et $f_2 = b \lor(AF \, c)$ \
Pour ce faire on doit évaluer f en chaque état. Puisque l'on a décomposé $f$ en $f_1$, $f_2$, on peut se contenter d'évaluer d'abord $Sat_{f_1}, Sat_{f_2}$ puis de considérer $f_1, f_2$ comme des propositions atomiques. \
Remarque : puisque $f_1$ est déjà une proposition atomique, $Sat_{f_1}$ est déjà construit avant de lancer le processus. Pour $f_2$ on repetera le processus.

### Formellement :
```
actualisation_sat (formule, sat, kripke) :
    decomposition = formule.decomposition()

    pour chaque sous_formule dans decomposition :
        sat = actualisation_sat(sous_formule, sat, kripke)

    sat[formule] = formule.evaluation(sat, kripke)

    retourne sat
```
Remarque : Pour que l'algorithme ci-dessus fonctionne, il faut incorporer dans le sat initiale les valeurs des propositions atomique de la structure de kripke.

## Opérateurs CTL
---
### Opérateurs minimaux (ENF)
---
Les opérateurs booléens (dont l'ordre des argument n'importe pas) peuvent être evalués localement comme ceci :

```
evaluation_bool (opérateurs, sat, kripke) :
    sat_formule = sat_vide()

    pour chaque noeud dans kripke :
        valeurs_opérateurs = []
        pour chaque opérateur dans opérateurs :
            valeur_opérateur = noeud dans sat[opérateur]
            valeurs_opérateurs.ajouter(valeur_opérateur)
        
        si operation(valeurs_opérateurs) :
            sat_formule.ajouter(noeud)

    retourne sat_formule
```

### **Pour les opérateurs non-booléens (modalités) :**

- Remarque 1 : Dans la suite on appel graphe d'une propriété $\phi$,
la restriction de celui ci aux noeuds vérifiant la propriété. Si K est une structure de Kripke, on note son graphe de $\phi$ : $K_\phi$
- Remarque 2 : On appel dans la suite un stricte composant fortement connexe (SCFC), un composant fortement connexe qui boucle sur lui même.
- Remarque 3 : On appel co-graphe d'un graphe orienté son équivalent avec les flèches retournées. On note pour K une structure de Kripke, sa co-structure $K^{op}$.

#### Opérateur EX
```
evaluation_ex (opérateur, sat, kripke) :
    sat_formule = sat_vide()

    pour chaque noeud dans kripke :
        boucle:
        pour noeud_suivant dans kripke.suivant(noeud) :
            si noeud_suivant dans sat[opérateur] :
                sat_formule.ajouter(noeud)
                casser boucle

    retourne sat_formule
```

#### Opérateur EG
- Explication
    - Un noeud vérifie EG $\phi$ si un de ses chemins vérifie $G \phi$.
    - Un tel chemin dans une structure de Kripke finira nécessairement par boucler infiniement dans un composant fortement connexe.
    - Il suffit de trouver les composants fortement connexes vérifiant $\phi$ et de remonter les états qui ont accès à ses noeuds tant qu'ils vérifiant aussi $\phi$.
- Algorithme de EG $\phi$ :
    - Calculer $K_\phi$
    - Calculer les SCFC de $K_\phi$ que l'on appelera AX (pour axiome)
    - $Sat_\phi = Exploration(K_\phi^{op}, AX)$
        - Ici la fonction exploration marque tout les noeuds rencontré dans $K_\phi^{op}$ en partant de $AX$ compris.
```
evaluation_eg(opérateur, sat, kripke) :
    kripke_restriction = kripke.restriction(sat, opérateur)
    AX = kripke_restriction.calculer_scfc()
    sat_formule = cloture_transitive(kripke_restriction.opposé())
    
    retourne sat_formule
```

#### Opérateur EU
- Explication
    - Si un noeud vérifie $\phi$, alors tout les chemins qui mènent à lui en vérifiant $\psi$ vérifient $\psi U \phi$
    - En partant des noeuds vérifiant $\phi$, tout les noeuds vérifiant $\psi$ valideront aussi la formule $E[\psi U \phi]$
- Algorithme de $E[\phi U \psi]$ :
    - Créer un ensemble des noeuds déjà visités $visited$.
    - Marquer les noeuds de $K_\psi$ dans une liste $batch$
    - Tant que $batch$ n'est pas vide
        - On initialise une liste $next\_batch$ pour une prochaine itération.
        - Ajouter $batch$ à $visited$
        - Si un élément est dans $batch$
            - On sait qu'il doit être ajouté à $Sat_{E[\phi U \psi]}$
            - tout ses enfants dans $K^{op}$ qui vérifient $\phi$ doivent appartenir à $Sat_{E[\phi U \psi]}$ et doivent donc être mis dans $next\_batch$ si ils ne font pas partie de $visited$.
        - On remplace $batch$ par $next\_batch$

```
evaluation_EU(phi, psi, sat, kripke) :
    kripke_restriction = kripke.restriction(sat, phi)
    
    batch = kripke_restrictions.noeuds()
    visited = ensemble_vide()
    
    tant que batch non vide :
        next_batch = []
        pour tout noeud dans batch mais pas dans visited :
            ajouter noeud à visited
            pour tout parent dans kripke.parents(noeud) :
                si parent dans sat[phi] mais pas dans visited :
                    ajouter à next_batch parent
    
    retourne visited
```

### Sucre syntaxique
---
Sont considéré comme sucre syntaxique, les opérateurs définis comme composition d'opérateurs minimaux. Dans la partie **opérateurs supplémentaires**, nous avons montré que les opérateurs suivant sont du sucre syntaxique pour la syntaxique ENF :
- $AX, AG, AF, A[ \,\cdot\, U \,\cdot\, ], EF$
- Weak Until : $A/E[ \,\cdot\, W \,\cdot\, ]$
- Release : $A/E[ \,\cdot\, R \,\cdot\, ]$
- A[ $\cdot$ FROM $\cdot$ TO $\cdot$ ] et E[ $\cdot$ FROM $\cdot$ TO $\cdot$ ]
- EFG, AGF


## AFG, EGF
---
Il est suffisant de faire l'algorithme uniquement pour EFG puisqu'il est complémentaire à AFG: \
$AFG \phi = \lnot EGF \lnot \phi$

Explication $EGF \phi$ :
- Un noeud vérifie $EGF \phi$ si une des 2 conditions est remplie
    - le noeud est dans un SCFC ou un noeud vérifie $\phi$
    - le noeud possède un chemin vers un noeud qui vérifie $EGF \phi$
Algorithme $EGF \phi$ :
- On calcule tout les SCFC
- On place dans batch les SCFC dont un noeud vérifie $\phi$
- Tant que $batch$ est non vide :
    - On initialize le prochain $batch$
    - On ajoute à $Sat_{EGF \phi} les noeuds du batch (non visités)
    - Pour chaque noeud dans le batch, on met dans le prochain batch les parents du noeud (non visités)
    - On remplace le batch par le prochain

```
evaluation_egf(phi, sat, kripke) :
    batch = []
    scfc = kripke.calculer_scfc()
    
    pour composant dans scfc :
        si un noeud du composant appartient sat[phi] :
            batch.ajouter_tout(composant)           
    
    visited = ensemble_vide()
    tant que batch non vide :
        next_batch = []
        pour tout noeud dans batch mais pas dans visited :
            ajouter noeud à visited
            pour tout parent dans kripke.parents(noeud) :
                si parent pas dans visited :
                    ajouter à next_batch parent

    retourne visited
```

## E[ $\varphi$ $G$% $n$ ]
---
La formule $E[\phi \,G\%\, n]$ est vérifié pour le noeud dont il existe un chemin tel qu'une fois sur $n$ la propriété $\phi$ soit vérifiée.

Explication :
- Soit $A_s$ l'ensemble des noeuds à n transition de $s$ qui vérifient $\phi$
- On appel graphe d'accessibilité : $A_\phi := \bigcup \set{A_s \mid s \in K_\phi}$
- Pour obtenir les noeuds qui vérifient $E[\phi \, G\%\, n]$ il suffit de regarder la cloture transitive de $A_\phi^{op}$

Algorithme :
- On calcule le graphe d'accessibilité.
- $Sat_{E[\phi \,G\%\, n]} = cloture\_transitive(A_\phi^{op})$
    - l'algorithme est similaire à celui des autres parties
```
evaluation_egk(phi, n, sat, kripke):
    acc = kripke.access(phi, n, sat).opposé()
    AX = acc.calculer_scfc()
    sat_formule = cloture_transitive(AX, acc)

    retourne sat_formule

```

# Outil développé

## Parser CTL, LTL, SL $\to$ Représentation abstraite
---


## Sauvegarde / Lecture des structures de Kripke
---


## Evaluation des formules CTL classiques
---


## Enrichissement de la syntaxe
---

# Déroulement du projet



# Références
- Principle of Model Checking, 2008, Christel Baier et Joost-Pieter Katoen
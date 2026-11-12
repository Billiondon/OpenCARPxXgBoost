# La pente de restitution suffit-elle à prédire la rupture d'onde cardiaque ?

Banc d'essai en simulation 2D (openCARP) et analyse par machine learning explicable (XGBoost / SHAP).

---

## Le problème médical

Chaque année, 4 à 5 millions de personnes décèdent de mort subite cardiaque dans le monde. Dans la majorité des cas, ce décès est provoqué par une **fibrillation ventriculaire (FV)** : l'onde électrique qui synchronise les ventricules se brise en plusieurs tourbillons chaotiques, empêchant le cœur de pomper le sang.

Pour protéger les personnes à haut risque, les médecins implantent un **défibrillateur automatique (DAI)** capable de délivrer un choc électrique salvateur dès qu'une FV démarre. Mais la décision d'opérer repose aujourd'hui sur un critère mécanique à **faible sensibilité et faible spécificité** : la fraction d'éjection (FEVG), qui mesure la force globale de contraction du cœur. En pratique, beaucoup de morts subites frappent des personnes dont la FEVG était jugée normale, tandis qu'une large partie des patients équipés d'un DAI ne déclenchera jamais d'arythmie de toute sa vie.

Pour comprendre les mécanismes invisibles de cet accident électrique, les chercheurs ont cherché à mettre la cellule cardiaque en équations. En s'appuyant sur le théorème de **Nolasco et Dahlen (1968)**, ils ont identifié un point de bascule précis sur une cellule isolée : dès que la durée d'un battement réagit trop fortement au manque de repos (**pente de restitution > 1**), la cellule perd son équilibre et entre dans un cycle irrégulier d'alternances (un battement long, un battement court).

Pendant longtemps, on a cru que cette règle suffisait à prédire le chaos dans tout le cœur. Pourtant, chez les patients, ce critère présente les mêmes limites que la FEVG : des cœurs basculent dans la fibrillation alors que la règle dit qu'ils devraient rester calmes, et d'autres restent stables alors que leur pente dépasse 1.

---

## L'objectif de ce projet

Pourquoi la formule mathématique se trompe-t-elle ? Parce qu'elle décrit une cellule isolée dans un bocal, alors qu'un vrai cœur est un tissu connecté qui possède une vitesse de propagation, une hétérogénéité et une **mémoire** des battements précédents.

L'objectif de ce dépôt n'est pas de vendre un prédicteur miracle de l'arrêt cardiaque, mais de construire un **banc de test rigoureux sur ordinateur**. En couplant le simulateur électrophysiologique **openCARP** à un modèle **XGBoost + SHAP**, le projet répond à une question simple : qu'est-ce qui déclenche vraiment la rupture électrique dans un tissu cardiaque ? La simple pente de départ, ou la mémoire et les propriétés du terrain ?

L'idée n'est pas de montrer qu'on sait entraîner une IA, mais d'utiliser l'explicabilité (SHAP) comme une loupe statistique pour hiérarchiser le poids réel de chaque mécanisme physiologique.

---

## Comment se déroule l'expérience ?

Toutes les simulations s'appuient sur le modèle cellulaire ventriculaire humain de **Ten Tusscher et Panfilov (2006)** dans openCARP.

**Étape 1 : Mesurer la réaction simple (1D)**
On commence par prendre un simple fil virtuel de muscle cardiaque de 2 cm. On lui envoie une série de battements réguliers, suivie d'un seul battement prématuré (protocole S1-S2), puis on trace la durée du battement en fonction du temps de repos qui le précède. La question est simple : est-ce que la cellule surréagit au manque de repos ? Si la pente dépasse 1, elle surréagit. Si la pente est inférieure à 1, elle reste calme.

**Étape 2 : Mesurer la mémoire et la vitesse (1D dynamique)**
On garde exactement le même fil, mais cette fois on accélère le rythme petit à petit, en laissant le tissu s'adapter à chaque palier. En superposant cette nouvelle courbe avec celle de l'étape 1, on voit immédiatement s'il y a un décalage. Si les deux courbes ne se collent pas, c'est la preuve que le tissu a de la mémoire : il ne réagit pas seulement au dernier battement, il se souvient aussi des précédents. On mesure en même temps à quel point l'onde électrique ralentit quand le rythme s'emballe.

**Étape 3 : Le crash-test sur 200 plaques (2D)**
On passe ensuite en deux dimensions sur une plaque de muscle cardiaque de 5 x 5 cm, comme un petit drap virtuel. Plutôt que de se contenter d'un seul cas, on fabrique 200 plaques différentes en faisant varier les réglages des canaux ioniques (\(I_{Ks}\), \(I_{CaL}\)), la vitesse de propagation et l'hétérogénéité du terrain (en mélangeant des zones saines et des zones plus fragiles). Sur chaque plaque, on déclenche une spirale électrique et on l'observe pendant 2 secondes : soit elle tourne proprement d'un seul bloc (stable), soit elle se déchire en plusieurs morceaux chaotiques (rupture).

**Étape 4 : Le verdict avec XGBoost et SHAP**
À la fin, on rassemble les mesures de nos 200 plaques dans un tableau et on les confie à un algorithme XGBoost, qui apprend à prédire quelles plaques vont casser et lesquelles vont tenir. On applique ensuite la méthode SHAP pour ouvrir le capot du modèle et voir quels critères ont réellement pesé dans la décision : la pente de départ, la mémoire, la vitesse de l'onde ou les défauts du tissu ? C'est ce classement qui tranche objectivement entre la vieille règle de 1968 et le comportement réel du tissu.

---

## Cas témoins observés en 2D

Avant d'analyser les 200 plaques, quatre scénarios de référence permettent d'illustrer les contre-exemples de la littérature :

| Scénario | Réglage du tissu | Pente max (1D) | Fronts d'onde à 1500 ms | Verdict |
| :--- | :--- | :---: | :---: | :--- |
| **C1** | Normal | `< 1` | `1` | Tourbillon stable |
| **C2** | Courant \(I_{Ks}\) réduit | `> 1` | `X` | *À observer* |
| **C3** | Pente > 1 + CV constante | `> 1` | `1` | **Stabilité malgré pente > 1** |
| **C4** | Pente < 1 + Tissu coupé en 2 | `< 1` | `> 1` | **Rupture malgré pente < 1** |

---

## Limites du modèle (à garder en tête)

Pour garder une démarche scientifique honnête, voici ce que ce projet ne prétend pas faire :

* **Un tissu 2D simplifié :** La plaque simulée est plate et isotrope. Elle ne reproduit ni l'orientation des fibres musculaires en 3D, ni le réseau de Purkinje, ni l'anatomie d'un cœur entier.
* **Un travail purement numérique :** Les résultats sont obtenus *in silico*, sans validation expérimentale sur tissu vivant ni prise en compte du système nerveux ou du métabolisme.
* **Rupture d'onde ≠ FV clinique :** On mesure la fragmentation physique d'un front d'onde sur une grille, pas un arrêt cardiaque clinique dans toute sa complexité.
* **Aucune prédiction médicale :** Les suivis cliniques sur plusieurs années montrent que la pente de restitution ne prédit ni la mortalité ni les chocs de défibrillateur chez le patient. Ce dépôt est un outil de compréhension biophysique, pas un outil de diagnostic.

---

## Structure du code et lancement

```text
├── scripts/
│   ├── 01_restitution_S1S2.py       # Étape 1 : mesure de la pente stationnaire en 1D
│   ├── 02_restitution_dynamique.py  # Étape 2 : calcul de la mémoire et de la CV en 1D
│   └── 03_cohort_2D.py              # Étape 3 : génération et simulation des 200 plaques 2D
├── analysis/
│   ├── count_wavebreaks.py          # Détection automatique des ruptures d'onde à 1500 ms
│   └── train_xgboost_shap.py        # Entraînement XGBoost et graphiques d'importance SHAP
├── results/
│   ├── figures/                     # Courbes de restitution, mémoire et graphiques SHAP
│   └── videos/                      # Vidéos des scénarios témoins exportées depuis meshalyzer
└── README.md
```

### Exécution rapide

**Prérequis :** [openCARP](https://opencarp.org) (recommandé via Docker), `carputils`, `xgboost` et `shap`.

```bash
# 1. Mesurer la pente stationnaire et la mémoire sur le fil 1D
python3 scripts/01_restitution_S1S2.py
python3 scripts/02_restitution_dynamique.py

# 2. Lancer les 4 scénarios témoins ou la cohorte complète en 2D
python3 scripts/03_cohort_2D.py --mode reference
python3 scripts/03_cohort_2D.py --mode cohort --n_samples 200

# 3. Extraire les ruptures d'onde et lancer l'analyse XGBoost + SHAP
python3 analysis/count_wavebreaks.py
python3 analysis/train_xgboost_shap.py
```

---

## Références

* **Nolasco, J. B., & Dahlen, R. W. (1968).** *A graphic method for the study of alternation in cardiac action potentials.* Journal of Applied Physiology.
* **Ten Tusscher, K. H., & Panfilov, A. V. (2006).** *Alternans and spiral breakup in a human ventricular tissue model.* American Journal of Physiology.
* **Qu, Z., Garfinkel, A., Chen, P. S., & Weiss, J. N. (2000).** *Mechanisms of discordant alternans and induction of reentry in simulated cardiac tissue.* Circulation.



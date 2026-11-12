La pente de restitution suffit-elle à prédire la rupture d'onde cardiaque ?

Le problème médical:

Chaque année, 4 à 5 millions de personnes décèdent de mort subite cardiaque. Dans la majorité des cas, ces décès sont la conséquence de la fibrillation ventriculaire (FV).

Pour prévenir ce trouble chez les personnes à haut risque, les médecins implantent un défibrillateur automatique (DAI) pour délivrer un choc salvateur en quelques secondes dès qu'une FV démarre. Mais la décision d'implanter le DAI se base sur un critère de faible sensibilité et spécificité à la FV : la fraction d'éjection (FEVG). Ainsi, beaucoup de morts subites ont lieu alors que la FEVG était normale, et une bonne partie des patients portant un DAI ne déclenche jamais d'arythmie de leur vie.

Pour comprendre les mécanismes invisibles de l'arrêt cardiaque, une formalisation mathématique des courants électriques a cherché à observer comment la cellule cardiaque cause le trouble rythmique. En s'appuyant sur le modèle de Nolasco et Dahlen (1968), les chercheurs ont identifié un point de bascule précis : dès que la cellule devient trop sensible au raccourcissement de son temps de repos (pente > 1), elle perd sa stabilité et entre dans un cycle irrégulier d'alternances.

Cependant, ce critère présente en pratique les mêmes limites que le critère de référence : des cœurs tremblent alors que la règle dit qu'ils ne devraient pas, et des cœurs restent stables alors que la règle dit qu'ils devraient trembler.

L'objectif du projet:

Le projet répond donc à une question simple : qu'est-ce qui déclenche vraiment la rupture électrique ? La simple pente de départ, ou la mémoire et les caractéristiques du tissu ?

Pour cela, je combine une simulation in silico robuste (openCARP) à un modèle ML prédictif (XGBoost) avec SHAP. L'objectif n'est pas de démontrer une capacité à entraîner une IA, mais d'utiliser l'explicabilité (SHAP) pour hiérarchiser le poids réel de chaque mécanisme physiologique dans la cassure de l'onde.

Le protocole en 4 étapes

Étape 1 : Mesurer la pente de restitution (1D)
On prend un fil de cœur de 2 cm dans openCARP (modèle Ten Tusscher).
On lance un protocole S1-S2 : des battements réguliers, puis un battement prématuré.
On trace ensuite la durée du battement en fonction du temps de repos.

La pente répond à une question simple : est-ce que le cœur surréagit ?
* Pente > 1 : il surréagit.
* Pente < 1 : il reste calme.

Étape 2 : Mesurer la mémoire (1D dynamique)
On reprend le même fil, mais cette fois on accélère le rythme petit à petit.
On compare cette courbe avec celle de l'étape 1.

Si les deux courbes ne se superposent pas, c'est la mémoire : le cœur se souvient des battements d'avant, pas uniquement du dernier.
On mesure aussi la restitution de vitesse, c'est-à-dire à quel point l'onde ralentit quand ça va vite.

Étape 3 : Crash-test en 2D
On passe sur une plaque de 5x5 cm, comme un drap artificiel.
On fabrique 200 plaques différentes en changeant un peu les réglages, la vitesse et l'hétérogénéité du terrain.
Sur chaque plaque, on lance une spirale électrique et on observe pendant 2 secondes :
* Stable : la spirale tourne d'un seul bloc.
* Rupture : la spirale se casse en plusieurs morceaux.

Étape 4 : Verdict avec XGBoost + SHAP
On donne les 200 plaques à XGBoost.
Il apprend à prédire : cassé ou pas cassé ?
Ensuite, SHAP nous montre qui a pesé le plus dans la décision : la pente, la mémoire ou la vitesse ?
C'est ce classement qui tranche : on voit si la vieille règle de la pente > 1 suffit, ou si la mémoire et le terrain comptent plus.


Cas témoins observés en 2D

| Scénario | Réglage du tissu | Pente max (1D) | Nombre de fronts d'onde à 1500 ms | Verdict |
| :--- | :--- | :---: | :---: | :--- |
| **C1** | Normal | `< 1` | `1` | Tourbillon stable |
| **C2** | Courant \(I_{Ks}\) réduit | `> 1` | `X` | *À observer* |
| **C3** | Pente > 1 + CV constante | `> 1` | `1` | **Stabilité malgré pente > 1** |
| **C4** | Pente < 1 + Tissu coupé en 2 | `< 1` | `> 1` | **Rupture malgré pente < 1** |

---

Limites du modèle

Modèle 2D simplifié : plaque isotrope sans fibres 3D, sans réseau de Purkinje et sans anatomie de cœur entier.
Simulation uniquement : pas de validation expérimentale animale, uniquement in silico, sans métabolisme ni système nerveux.
Rupture d'onde ≠ FV clinique : on mesure la fragmentation d'un front d'onde sur ordinateur, pas une fibrillation ventriculaire clinique.
Pas de prédiction médicale : les études cliniques (comme le suivi sur 6 ans de 74 patients porteurs de DAI) montrent que la pente ne prédit ni la mortalité ni les chocs. Ce dépôt est un banc de test biophysique, pas un outil médical.

Structure du code et exécution

scripts/01_restitution_S1S2.py : Étape 1 (pente stationnaire 1D)
scripts/02_restitution_dynamique.py : Étape 2 (mémoire et restitution de vitesse 1D)
scripts/03_cohort_2D.py : Étape 3 (génération et simulation des 200 plaques 2D)
analysis/count_wavebreaks.py : Comptage des fronts d'onde à 1500 ms
analysis/train_xgboost_shap.py : Étape 4 (entraînement XGBoost et explication SHAP)
results/figures/ : Courbes de restitution, mémoire et graphiques SHAP
results/videos/ : Vidéos des scénarios C1-C4 exportées depuis meshalyzer

Prérequis : openCARP, carputils, xgboost, shap.

Commandes :
python3 scripts/01_restitution_S1S2.py
python3 scripts/02_restitution_dynamique.py
python3 scripts/03_cohort_2D.py --mode reference
python3 scripts/03_cohort_2D.py --mode cohort --n_samples 200
python3 analysis/count_wavebreaks.py
python3 analysis/train_xgboost_shap.py

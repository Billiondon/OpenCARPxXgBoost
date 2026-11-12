**Prédiction du risque d'arythmie ventriculaire**: pourquoi la pente de restitution échoue à anticiper la rupture d'onde cardiaque?
Chaque année 4 à 5 M de personnes décèdent de mort subite cardiaque. Dans la majorité des cas, ces décès sont la conséquence de la fibrillation ventriculaire. 
Pour prévenir ce trouble chez les personnes à haut risque, les médecins implantent un défibrillateur automatique pour délivrer un choc salvateur en quelques secondes dès qu'une FV démarre.
Mais la décision d'implanter le DAI se base sur un critere de faible sensibilite et specificite à la FV.(FEVG).
Ainsi, bcp de morts subites ont lieu alors que la FEVG était normale et une bonne partie de patients portant un DAI ne la déclenche jamais de leur vie.

Pour comprendre les mécanismes invisibles de l'arrêt cardiaque, une formalisation mathématique des courants électriques a cherché à observer comment la cellule cardiaque cause le trouble rythmique
En s'appuyant sur le modèle de Nolasco et Dahlen (1968), les chercheurs ont identifié un point de bascule précis. Dès que la cellule devient trop sensible au raccourcissement de son temps de repos (pente > 1), elle perd sa stabilité et entre dans un cycle irrégulier d'alternances.

Cependant, ce critère presente les memes limites que celui de réference: des cœurs basculent dans le chaos alors que la règle dit qu'ils ne devraient pas, et des cœurs restent stables alors que la règle dit qu'ils devraient trembler

Le projet est répond donc a une question simple: qu'est-ce qui déclenche vraiment la rupture électrique ? La simple pente de départ, ou la mémoire et les caractéristiques du tissu ?
Pour cela, je combine une simulation in silico robuste ( openCARB, 200 Simulations ) a un modele ML prédictif ( XGBOOST) avec SHAP.
L'objectif n'est pas de démontrer une capacité à entrainer une IA mais d'utiliser l'explicabilité (SHAP) pour hiérarchiser le poids réel de chaque mécanisme physiologique dans la cassure de l'onde.  

Le dépôt regroupe le code et les résultats de l'analyse prédictiva de la fibrillation cardiaque, combinant des simulations électrophysiologiques **openCARP** et des modèles de Machine Learning explicables (**XGBoost** et **SHAP**).

**Résultats Clés**
* **Performance du Modèle** : 100% de précision sur le jeu de test.
* **Paramètres Clés** : Domination nette de la **`pente_dyn`** et de la **`pente_S1_S2`** dans la prédiction de la rupture.

**Visualisations**
*Importance globale (XGBoost Gain) :
  ![Feature Importance](feature_importance.png)

**SHAP** :
  ![SHAP Summary](shap_summary.png)

**Stack Technique**
* Environnement : Conteneur Docker `openCARP` (Ubuntu)
* Bibliothèques : `Python`, `XGBoost`, `SHAP`, `Scikit-Learn`, `Matplotlib`, `Pandas`

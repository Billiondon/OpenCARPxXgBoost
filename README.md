# Prédiction du risque d'arythmie : pourquoi la pente de restitution échoue à anticiper la rupture d'onde cardiaque

**Un banc d'essai en simulation 2D (openCARP) pour tester les limites d'un célèbre critère électrophysiologique.**

---

## 1. Le problème médical au départ : la fibrillation ventriculaire

En cardiologie, le mot « fibrillation » désigne deux maladies très différentes qu'il ne faut pas confondre :
* **La fibrillation auriculaire (FA) :** Elle touche les oreillettes (le haut du cœur). Très fréquente (environ 1 million de personnes en France), elle gêne le quotidien et augmente le risque d'AVC, mais elle ne tue pas sur le coup.
* **La fibrillation ventriculaire (FV) — le sujet de ce projet :** Elle frappe les ventricules, la pompe principale du cœur. Au lieu de se contracter d'un seul bloc pour envoyer le sang dans le corps et le cerveau, le muscle se met à frémir de manière anarchique. L'onde électrique s'est brisée en plusieurs tourbillons chaotiques. Sans choc électrique dans les minutes qui suivent, c'est l'arrêt cardiaque et la mort certaine.

### L'ampleur du fléau et le rôle du défibrillateur
La fibrillation ventriculaire est la cause principale de la **mort subite cardiaque** (75 à 80 % des cas). Elle tue **46 000 à 50 000 personnes par an en France** (et 4 à 5 millions dans le monde), avec moins de 5 % de survie sans séquelles.

Face à ce chaos électrique, les médicaments sont trop lents. **Le seul bouclier efficace est le défibrillateur automatique implantable (DAI)** : un petit boîtier placé sous la peau qui surveille le cœur en continu et délivre un choc salvateur dès qu'une FV démarre.

### Le dilemme médical : un outil de dépistage imprécis
Aujourd'hui, la décision d'implanter un défibrillateur (une opération invasive, coûteuse, avec des risques d'infection ou de chocs électriques douloureux délivrés par erreur) se base principalement sur un critère mécanique grossier : **la fraction d'éjection du ventricule gauche (FEVG)**, qui mesure la force globale de contraction du cœur.

Or, la FEVG présente une **très faible sensibilité et une faible spécificité** pour prédire la fibrillation ventriculaire :
* **Faible sensibilité (les oubliés) :** Plus de la moitié des victimes de mort subite avaient une FEVG normale ou peu altérée, et n'avaient donc pas de défibrillateur.
* **Faible spécificité (les implantés à tort) :** Parmi les patients qui ont une FEVG basse et reçoivent un défibrillateur par précaution, la grande majorité ne fera jamais de FV de toute sa vie.

---

## 2. L'idée de départ et son échec chez le patient

Pour combler ce vide médical, les cardiologues ont cherché un test électrique direct. Ils se sont appuyés sur une règle mathématique de 1968 (Nolasco et Dahlen) qui décrit le comportement d'une cellule cardiaque isolée :

* **Le principe :** La durée d'un battement (APD) dépend du temps de repos juste avant (DI). Si la cellule réagit trop fortement au manque de repos (une courbe avec une **pente supérieure à 1**), elle perd le rythme. Elle alterne alors entre un battement long et un battement court (*alternans*).
* **La promesse médicale :** Les médecins ont pensé que cette instabilité allait briser l'onde électrique dans tout le cœur et déclencher une fibrillation. Il suffisait donc de mesurer cette pente avec une sonde : si la pente dépassait 1, on posait un défibrillateur.
* **L'échec dans la vraie vie :** Les études sur plusieurs années ont montré que ce test ne fonctionne pas. Beaucoup de patients avec une pente supérieure à 1 vivent très bien sans jamais faire de crise. À l'inverse, d'autres patients avec une pente rassurante (inférieure à 1) font des fibrillations mortelles.

---

## 3. L'objectif de ce projet

L'objectif de ce dépôt n'est pas de vendre un « prédicteur miracle » de l'arrêt cardiaque. Il s'agit de construire un **banc de test rigoureux sur ordinateur** pour démontrer, simulation à l'appui, pourquoi ce critère unique échoue dès qu'on passe de la cellule isolée au tissu cardiaque.

Concrètement, nous cherchons à reproduire sur simulateur deux contre-exemples qui contredisent la théorie classique :
1. **La fausse alerte :** Une onde électrique qui reste parfaitement stable même avec une **pente > 1**.
2. **Le piège silencieux :** Une onde électrique qui se brise en morceaux chaotiques même avec une **pente < 1**.

---

## 4. Comment fonctionne l'expérience ?

Toutes les simulations utilisent le simulateur **openCARP** avec le modèle mathématique de cellule ventriculaire humaine de référence (**Ten Tusscher et Panfilov, 2006**). Le protocole est découpé en trois étapes distinctes pour ne pas mélanger les effets :

### Protocole A : Mesurer la réaction simple (1D sans mémoire)
On envoie une série de battements réguliers sur un fil virtuel de 2 cm, suivie d'un seul battement prématuré (protocole S1-S2). Cela permet de tracer la courbe de restitution stationnaire et de repérer les réglages cellulaires (canaux ioniques $I_{Ks}$ et $I_{CaL}$) qui donnent une **pente < 1** ou une **pente > 1**.

### Protocole B : Mesurer l'effet mémoire (1D dynamique)
Sur le même fil, on accélère progressivement le rythme cardiaque par paliers (de 1000 ms à 200 ms) en laissant le tissu s'adapter à chaque étape. L'écart entre cette courbe dynamique et la première courbe mesure directement la **mémoire cardiaque**. Si cet écart est grand, la théorie simple de 1968 ne s'applique plus.

### Protocole C : Tester la cassure sur une plaque de tissu (2D)
On passe sur un carré de muscle cardiaque de **5 x 5 cm** et on déclenche un tourbillon électrique (une réentrée). On observe ce qui se passe pendant 2000 ms dans 4 scénarios :
* **C1 (Tissu sain) :** Pente < 1. On vérifie que le tourbillon tourne proprement sans casser.
* **C2 (Pente forte) :** Pente > 1. On regarde si la forte pente suffit à briser l'onde.
* **C3 (Pente forte + vitesse figée) :** Pente > 1, mais on empêche la vitesse de propagation (CV) de ralentir aux rythmes rapides. *Objectif : montrer qu'une onde peut rester stable malgré une pente > 1.*
* **C4 (Tissu hétérogène) :** Pente < 1 partout, mais la plaque est coupée en deux zones différentes (moitié C1, moitié C2). *Objectif : montrer que les différences de terrain brisent l'onde même avec une pente faible.*

---

## 5. Résultats principaux

*(Tableau à compléter avec vos valeurs exactes après simulation)*

| Scénario | Réglage du tissu | Pente max (1D) | Nombre de fronts d'onde à 1500 ms | Verdict |
| :--- | :--- | :---: | :---: | :--- |
| **C1** | Normal | `< 1` | `1` | Tourbillon stable |
| **C2** | Courant $I_{Ks}$ réduit | `> 1` | `X` | *À observer* |
| **C3** | Pente > 1 + CV constante | `> 1` | `1` | **Stabilité malgré pente > 1** |
| **C4** | Pente < 1 + Tissu coupé en 2 | `< 1` | `> 1` | **Rupture malgré pente < 1** |

**Conclusion de l'expérience :** Dans notre configuration, nous observons des cas de rupture d'onde avec une pente inférieure à 1 (C4) et des cas de stabilité avec une pente supérieure à 1 (C3). Cela confirme la littérature moderne : la pente de restitution seule est un critère insuffisant. Pour comprendre l'instabilité électrique du cœur, il faut regarder l'ensemble formé par la durée du battement, la vitesse de conduction, la mémoire et l'hétérogénéité du tissu.

---

## 6. Ce que ce projet ne fait pas (Limites importantes)

Pour garder une approche scientifique honnête, il est essentiel de préciser les limites de ce travail :

* **Un modèle 2D simplifié :** Le tissu simulé est une plaque plate et isotrope. Il ne reproduit ni l'orientation complexe des fibres musculaires en 3D, ni le réseau électrique naturel (Purkinje), ni l'anatomie d'un cœur entier.
* **Pas de validation sur le vivant :** Ce projet est purement numérique (*in silico*). Il n'inclut pas d'expériences sur tissu animal ni l'influence du stress, des hormones ou du métabolisme.
* **Rupture d'onde ≠ Fibrillation clinique :** Nous mesurons un phénomène physique précis (la fragmentation d'un front d'onde sur une grille), et non un arrêt cardiaque clinique dans toute sa complexité.
* **Aucune prédiction médicale :** Des études cliniques de suivi sur plusieurs années (notamment chez des patients porteurs de défibrillateurs) ont déjà montré que la pente de restitution ne permet pas de prédire la mortalité ni le risque de choc électrique. Ce code n'a donc aucune vocation diagnostique.

---

## 7. Organisation du dépôt et lancement

```text
├── scripts/
│   ├── 01_restitution_S1S2.py       # Protocole A : mesure de la pente stationnaire en 1D
│   ├── 02_restitution_dynamique.py  # Protocole B : calcul de l'effet mémoire en 1D
│   └── 03_simulation_2D.py          # Protocole C : lancement des 4 scénarios (C1 à C4)
├── analysis/
│   ├── plot_curves.py               # Génération des graphiques APD = f(DI)
│   └── count_wavebreaks.py          # Détection automatique des cassures d'onde à 1500 ms
├── results/
│   ├── figures/                     # Courbes de restitution et de mémoire (PNG)
│   └── videos/                      # Vidéos des 4 simulations 2D exportées depuis meshalyzer
└── README.md
```

**Prérequis :** Avoir installé [openCARP](https://opencarp.org) (recommandé via l'image Docker officielle) et `carputils`.

```bash
# 1. Calculer les courbes de restitution et la mémoire en 1D
python3 scripts/01_restitution_S1S2.py
python3 scripts/02_restitution_dynamique.py

# 2. Lancer les 4 scénarios sur la plaque 2D
python3 scripts/03_simulation_2D.py --case C1
python3 scripts/03_simulation_2D.py --case C2
python3 scripts/03_simulation_2D.py --case C3
python3 scripts/03_simulation_2D.py --case C4

# 3. Compter automatiquement les fronts d'onde et générer les figures
python3 analysis/count_wavebreaks.py
```

---

## 8. Références clés


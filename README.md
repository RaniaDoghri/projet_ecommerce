# Projet ecommerce — Nettoyage & Analyse

Pipeline de nettoyage de données de ventes avec `pandas`, suivi d'une analyse du chiffre d'affaires.

---

##  Structure du projet

```
projet-ventes/
├── README.md
├── .gitignore
├── data/
│   ├── ventes.csv          # Dataset brut (sale)
│   └── ventes_clean.csv    # Dataset nettoyé, prêt pour analyse
└── notebooks/
    ├── nettoyage_ventes.ipynb   # Pipeline de nettoyage
    └── analyse_ca.ipynb         # Analyse du chiffre d'affaires
```

---

##  Objectif

Partir d'un jeu de données de ventes contenant des incohérences réalistes (valeurs manquantes, formats mixtes, doublons, outliers), le nettoyer, puis calculer et analyser le chiffre d'affaires.

---

##  Description des données

Le fichier `ventes.csv` contient 100 transactions avec les colonnes suivantes :

| Colonne    | Type attendu | Description                         |
|------------|--------------|-------------------------------------|
| `ID`       | int          | Identifiant unique de la transaction |
| `Prix`     | float        | Prix unitaire en euros              |
| `Quantite` | int          | Nombre d'unités vendues             |
| `Remise`   | float        | Pourcentage de remise (0–100)       |

### Problèmes présents dans les données brutes

- **Valeurs manquantes** sur les trois colonnes numériques
- **Formats mixtes** : virgules décimales à la française (`50,37`), symbole `€`, symbole `%`
- **Valeurs textuelles** dans `Quantite` (`"cinq"`, `"deux"`)
- **Valeurs aberrantes** : prix négatifs, remises > 100%, quantités extrêmes
- **IDs dupliqués**

---

##  Règles de nettoyage appliquées

| Colonne    | Traitement |
|------------|-----------|
| `Prix`     | Suppression du `€`, virgule → point, conversion en float, prix négatifs → valeur absolue |
| `Quantite` | Conversion des nombres en lettres (`"cinq"` → `5`), conversion en int |
| `Remise`   | Suppression du `%`, conversion en float, valeurs hors [0, 100] → NaN |
| `ID`       | Suppression des doublons (première occurrence conservée) |

### Imputation des valeurs manquantes
- `Prix` et `Quantite` → **médiane** de la colonne
- `Remise` → **0** (pas de remise par défaut)

---

##  Démarrage rapide

### Prérequis

```bash
pip install pandas numpy jupyter
```

### Cloner le projet

```bash
git clone https://github.com/<ton-user>/projet-ventes.git
cd projet-ventes
```

### Charger le dataset propre

```python
import pandas as pd
df = pd.read_csv('data/ventes_clean.csv')
df.head()
```

---

##  Workflow

1. **Cellule 1** du notebook `nettoyage_ventes.ipynb` génère `ventes.csv` (dataset sale)
2. **Cellules 2 à 10** nettoient les données et exportent `ventes_clean.csv`
3. Le notebook `analyse_ca.ipynb` consomme `ventes_clean.csv` pour les calculs métier

---

##  Équipe & répartition

| Membre       | Responsabilité                              |
|--------------|---------------------------------------------|
| **Rania Doghri**    | Génération du dataset & pipeline de nettoyage |
| **[Coéquipier]** | Calcul du chiffre d'affaires & analyse      |

---

##  Conventions Git

- Travailler sur des branches dédiées : `feature/nettoyage`, `feature/analyse-ca`
- Messages de commit en français, format court : `ajout`, `fix`, `refacto`, `docs`
- Ouvrir une Pull Request avant de merger sur `main`

---

##  Notes techniques

- Encodage des fichiers CSV : **UTF-8**
- Séparateur : **virgule** (`,`)
- Python ≥ 3.10 recommandé

---

##  Licence

Projet PFA

# Projet Ventes Tunisie
## Generation et Nettoyage d'un Dataset Commercial

---

## Table des matieres

1. [Contexte du projet](#1-contexte-du-projet)
2. [Structure du dataset](#2-structure-du-dataset)
3. [Fourchettes de prix par sous-categorie](#3-fourchettes-de-prix-par-sous-categorie-tnd)
4. [Erreurs injectees dans le dataset brut](#4-erreurs-injectees-dans-le-dataset-brut)
5. [Notebook de generation](#5-notebook-de-generation--ventes_dataset_v2ipynb)
6. [Pipeline de nettoyage](#6-pipeline-de-nettoyage--pipeline_nettoyageipynb)
7. [Resultats du nettoyage](#7-resultats-du-nettoyage)
8. [Fichiers du projet](#8-fichiers-du-projet)
9. [Comment utiliser ce projet](#9-comment-utiliser-ce-projet)

---

## 1. Contexte du projet

Ce projet simule un cas reel de Data Engineering : a partir d'une specification metier, on genere un dataset commercial synthetique pour la Tunisie, on y injecte deliberement des erreurs typiques du terrain (valeurs manquantes, doublons, formats incoherents, outliers), puis on construit un pipeline de nettoyage complet et reproductible.

| Element | Detail |
|---|---|
| Objectif | Generer un dataset de ventes tunisiennes realiste avec erreurs, puis le nettoyer |
| Langage | Python 3 / Jupyter Notebook (.ipynb) |
| Librairies | pandas, numpy, random, datetime, re, openpyxl |
| Fichier brut | ventes_dirty_N.xlsx |
| Fichier propre | ventes_clean.xlsx |
| Notebook generation | ventes_dataset_v2.ipynb |
| Notebook nettoyage | pipeline_nettoyage.ipynb |

---

## 2. Structure du dataset

### 2.1 Colonnes

| Colonne | Type attendu | Description |
|---|---|---|
| ID_Vente | string | Identifiant unique de la vente (ex : VNT-0042) |
| Date_Vente | date | Date de la transaction |
| Region | string | Wilaya tunisienne |
| Categorie_Produit | string | Grande categorie du produit |
| Sous_categorie | string | Sous-categorie specifique |
| Prix_Unitaire | float | Prix en TND |
| Quantite | integer | Nombre d'unites vendues |
| Mode_Paiement | string | Moyen de paiement utilise |

### 2.2 Valeurs de reference

**Regions**

Tunis, Ariana, Sfax, Sousse, Monastir, Nabeul, Bizerte, Gabes

**Categories et sous-categories**

| Categorie | Sous-categories |
|---|---|
| Electronique | Telephones, Ordinateurs Portables, Tablettes, Ecouteurs, Accessoires Telephone |
| Mode et Vetements | Jeans, Hoodies, Chaussures, Robes, Vestes |
| Beaute et Cosmetique | Shampoing, Parfum, Skincare, Maquillage, Cremes |
| Maison et Cuisine | Lampes, Oreillers, Decoration, Ustensiles, Mixeurs |
| Sports et Loisirs | Equipement Fitness, Velos, Chaussures Sport, Raquettes, Balles |

**Modes de paiement**

Especes, Carte Bancaire, D17

---

## 3. Fourchettes de prix par sous-categorie (TND)

Les prix sont calibres par sous-categorie pour refleter le marche tunisien (sources : Jumia TN, Mytek, Tunisianet).

| Sous-categorie | Prix min (TND) | Prix max (TND) | Categorie parente |
|---|---|---|---|
| Accessoires Telephone | 5 | 90 | Electronique |
| Ecouteurs | 25 | 450 | Electronique |
| Tablettes | 400 | 2 200 | Electronique |
| Telephones | 350 | 3 500 | Electronique |
| Ordinateurs Portables | 1 200 | 6 000 | Electronique |
| Hoodies | 35 | 130 | Mode et Vetements |
| Jeans | 45 | 180 | Mode et Vetements |
| Robes | 40 | 200 | Mode et Vetements |
| Vestes | 80 | 280 | Mode et Vetements |
| Chaussures | 60 | 350 | Mode et Vetements |
| Cremes | 10 | 60 | Beaute et Cosmetique |
| Shampoing | 8 | 45 | Beaute et Cosmetique |
| Maquillage | 12 | 90 | Beaute et Cosmetique |
| Skincare | 20 | 130 | Beaute et Cosmetique |
| Parfum | 35 | 250 | Beaute et Cosmetique |
| Ustensiles | 8 | 95 | Maison et Cuisine |
| Oreillers | 20 | 80 | Maison et Cuisine |
| Lampes | 15 | 120 | Maison et Cuisine |
| Decoration | 10 | 150 | Maison et Cuisine |
| Mixeurs | 90 | 550 | Maison et Cuisine |
| Balles | 5 | 40 | Sports et Loisirs |
| Raquettes | 30 | 200 | Sports et Loisirs |
| Chaussures Sport | 60 | 280 | Sports et Loisirs |
| Equipement Fitness | 80 | 900 | Sports et Loisirs |
| Velos | 350 | 2 500 | Sports et Loisirs |

---

## 4. Erreurs injectees dans le dataset brut

Le dataset brut contient deliberement 7 types d'erreurs representatifs des problemes rencontres en production.

| Type d'erreur | Taux injecte | Exemple concret | Colonnes affectees |
|---|---|---|---|
| Valeurs manquantes | 3 - 4 % | NaN dans Mode_Paiement, Prix ou Categorie | Mode_Paiement, Prix_Unitaire, Categorie_Produit |
| Doublons sur ID | 3 % | VNT-0042 apparait deux fois | ID_Vente |
| Formats de date mixtes | 8 % | 24/04/2024 au lieu de 2024-04-24 | Date_Vente |
| Typos region | 6 % | 'tunis', 'SFAX', 'Sousse ' (espace parasite) | Region |
| Quantite invalide | 2 + 2 % | '3.0' comme string, valeur 0 ou -1 | Quantite |
| Prix outlier | 2 % | Prix 4x a 8x le max de la sous-categorie | Prix_Unitaire |
| Libelle categorie errone | 3 % | 'beaute et cosmetique', 'Mode & Vetements' | Categorie_Produit |

---

## 5. Notebook de generation  (ventes_dataset_v2.ipynb)

Le notebook est divise en 7 cellules independantes et reproductibles.

| Cellule | Titre | Role |
|---|---|---|
| 1 | Imports | Chargement des librairies (pandas, numpy, random, datetime) |
| 2 | Configuration | Parametres modifiables : N_ROWS, SEED, OUTPUT_PATH |
| 3 | Reference Data | Catalogue de 25 sous-categories avec prix min/max TND |
| 4 | Fonctions dirty | 7 fonctions d'injection d'erreurs, une par type de probleme |
| 5 | Generation | Boucle principale : valeurs propres puis injections |
| 6 | Rapport qualite | Verification statistique de chaque type d'erreur injecte |
| 7 | Export | Sauvegarde du fichier Excel brut |

### Parametre de taille

Pour changer la taille du dataset, modifier uniquement la variable `N_ROWS` dans la Cellule 2 :

```python
N_ROWS = 100    # petit jeu de test
N_ROWS = 200    # taille par defaut
N_ROWS = 500    # jeu moyen
N_ROWS = 1000   # grand jeu de production
```

### Fonctions d'injection d'erreurs (Cellule 4)

Chaque probleme est isole dans sa propre fonction. Il suffit de commenter un appel dans la Cellule 5 pour desactiver un type d'erreur specifique.

```
inject_missing_values()   --> NaN dans Mode_Paiement, Categorie, Prix
inject_duplicate_id()     --> Reutilisation d'un ID precedent
inject_date_format()      --> Format dd/mm/yyyy ou yyyy/mm/dd
inject_region_typo()      --> Casse incorrecte, espace parasite
inject_bad_quantity()     --> Valeur negative, zero, ou string float
inject_outlier_price()    --> Prix 4x a 8x le maximum de la sous-categorie
inject_category_typo()    --> Libelle avec casse ou caracterex incorrects
```

---

## 6. Pipeline de nettoyage  (pipeline_nettoyage.ipynb)

Le pipeline traite chaque type d'erreur dans un ordre logique, du plus structurel (doublons) au plus fin (valeurs manquantes residuelles).

| Cellule | Etape | Methode utilisee | Resultat observe |
|---|---|---|---|
| 2 | Chargement | read_excel avec dtype=str pour ne rien perdre | 200 lignes x 8 colonnes |
| 3 | Suppression des doublons | drop_duplicates sur ID_Vente, keep='first' | 3 lignes supprimees, 197 restantes |
| 4 | Standardisation des dates | Regex + strptime pour detecter les 3 formats | 0 NaN restants |
| 5 | Normalisation Region | strip().title() + validation contre ensemble valide | 0 NaN restants |
| 6 | Normalisation Categorie | Matching par mots-cles insensible a la casse | 7 NaN irrecuperables |
| 7 | Nettoyage Quantite | to_numeric + invalidation <= 0 + imputation mediane | 0 NaN, type int |
| 8 | Nettoyage Prix | Borne par sous-categorie (3x max) + mediane par groupe | 2 outliers invalides, 0 NaN |
| 9 | Valeurs manquantes restantes | Categorie depuis Sous_categorie + Mode par le mode | Minimum de NaN residuels |
| 10 | Typage final | datetime, float, int selon la semantique metier | Types corrects |
| 11 | Rapport de nettoyage | Statistiques descriptives completes | Vue d'ensemble finale |
| 12 | Export | to_excel vers ventes_clean.xlsx | Fichier propre sauvegarde |

### Strategies d'imputation

| Colonne | Strategie | Justification |
|---|---|---|
| Mode_Paiement | Mode (valeur la plus frequente) | Trois valeurs seulement, distribution connue |
| Prix_Unitaire | Mediane de la sous-categorie | Robuste aux outliers residuels, contextuelle |
| Quantite | Mediane globale | Distribution symetrique entre 1 et 10 |
| Categorie_Produit | Deduction depuis Sous_categorie via mapping | Information disponible dans la meme ligne |
| Sous_categorie | Conserve NaN | Irrecuperable si les deux colonnes sont nulles |
| Date_Vente | Conserve NaN | Aucune heuristique fiable pour imputer une date |

### Detection des outliers de prix (Cellule 8)

Un prix est considere comme une erreur si il depasse **3 fois le maximum de reference** de sa sous-categorie. Cette approche est plus fiable que la methode IQR globale car elle tient compte du contexte metier de chaque produit.

```python
# Exemple : Balles (max = 40 TND)
# Seuil d'outlier = 40 * 3 = 120 TND
# Un prix de 280 TND sur une Balle = outlier confirme

def flag_outlier_prix(row):
    sc, px = row['Sous_categorie'], row['Prix_Unitaire']
    if pd.isna(px) or pd.isna(sc) or sc not in PRIX_MAX_REF:
        return False
    return px > PRIX_MAX_REF[sc] * 3
```

---

## 7. Resultats du nettoyage

| Indicateur | Avant | Apres | Statut |
|---|---|---|---|
| Nombre de lignes | 200 | 197 | 3 doublons supprimes |
| Doublons sur ID_Vente | 3 | 0 | Corrige |
| Formats de date incoherents | ~16 | 0 | Corrige |
| Valeurs Region incorrectes | ~12 | 0 | Corrige |
| Libelles Categorie incorrects | ~6 | 0 | Corrige |
| Quantites invalides (<= 0 ou string) | ~6 | 0 | Corrige |
| Prix outliers | 2 | 0 | Corriges par mediane sous-categorie |
| NaN Mode_Paiement | ~8 | 0 | Impute par le mode |
| NaN Categorie (irrecuperables) | 7 | 7 | Irrecuperable (cat + sous-cat nulles) |
| Type Prix_Unitaire | string | float | Corrige |
| Type Quantite | string | int | Corrige |
| Type Date_Vente | string | date | Corrige |

---

## 8. Fichiers du projet

```
projet-ventes-tunisie/
|
|-- ventes_dataset_v2.ipynb       # Notebook de generation du dataset brut (7 cellules)
|-- pipeline_nettoyage.ipynb      # Pipeline de nettoyage complet (12 cellules)
|-- ventes_dirty_N.xlsx           # Dataset brut genere avec erreurs (N = nombre de lignes)
|-- ventes_clean.xlsx             # Dataset propre apres nettoyage
|-- README.md                     # Ce fichier de documentation
```

---

## 9. Comment utiliser ce projet

### Etape 1 - Generer le dataset brut

1. Ouvrir `ventes_dataset_v2.ipynb` dans Jupyter.
2. Dans la Cellule 2, definir `N_ROWS` selon le nombre de lignes voulu.
3. Executer toutes les cellules (Run All).
4. Le fichier `ventes_dirty_N.xlsx` est genere dans le meme repertoire.

### Etape 2 - Nettoyer le dataset

1. Ouvrir `pipeline_nettoyage.ipynb` dans Jupyter.
2. Dans la Cellule 2, verifier que `INPUT_PATH` correspond au fichier brut genere.
3. Executer toutes les cellules (Run All).
4. Le fichier `ventes_clean.xlsx` est sauvegarde dans le meme repertoire.

### Dependances Python

```bash
pip install pandas numpy openpyxl
```

Les librairies `re` et `datetime` font partie de la librairie standard Python et ne necessitent pas d'installation.

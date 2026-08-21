# Data Quality — Nolvéa Distribution

Cette documentation présente les principaux problèmes de qualité de données intégrés au cas d’étude Nolvéa Distribution ainsi que les traitements mis en place pour fiabiliser le reporting.

> **Projet portfolio fictif** — les données sont entièrement synthétiques et ont été construites pour reproduire des problématiques réalistes de reporting commercial B2B.

---

## 1. Périmètre des données

Le jeu de données couvre l’année 2025 et repose sur :

- 12 exports mensuels de ventes ;
- 37 879 lignes brutes ;
- deux formats ERP successifs ;
- plusieurs dimensions métier : clients, produits, commerciaux, canaux et calendrier.

Après les traitements de qualité et la déduplication, le modèle contient :

**37 864 lignes fiabilisées.**

---

## 2. Changement d’ERP en cours d’année

Le scénario prévoit une migration ERP au **1er juillet 2025** :

- janvier à juin : ancien système **GESCOM** ;
- juillet à décembre : nouveau système **SAGE X3**.

Les deux sources ne possèdent pas exactement le même schéma.

### Principales différences

| Ancien format | Nouveau format |
|---|---|
| `ChiffreAffaires` | `CA_Net` |
| `Cout` | `CoutAchat` |
| Date native Excel | Date au format texte `JJ/MM/AAAA` |
| Pas de `CodeEntrepot` | Ajout de `CodeEntrepot` |

À partir d’octobre, certains champs numériques tels que `PrixUnitaire` et `Remise` sont également fournis sous forme de texte avec une virgule comme séparateur décimal.

### Risque

Sans harmonisation, la consolidation annuelle peut générer :

- des colonnes incompatibles ;
- des valeurs numériques interprétées comme du texte ;
- des erreurs de calcul ;
- des ruptures lors du rafraîchissement.

### Traitement

La chaîne Power Query :

1. identifie les fichiers à consolider ;
2. harmonise les noms de colonnes ;
3. normalise les types ;
4. convertit explicitement les dates ;
5. traite les formats décimaux ;
6. restitue un schéma commun avant chargement dans le modèle.

---

## 3. Doublons de ventes

Le jeu de données brut contient **15 lignes dupliquées**.

### Impact

Sans traitement :

- CA brut : **16 458 774 €**
- CA après déduplication : **16 451 262 €**

Soit un écart de :

**7 513 €**

Même si l’écart reste limité par rapport au chiffre d’affaires total, il démontre qu’un contrôle de doublons est nécessaire avant toute analyse.

### Traitement

Les doublons sont identifiés à partir des attributs permettant de caractériser une ligne de vente, puis supprimés dans la chaîne de préparation.

Le nombre de lignes passe ainsi de :

**37 879 → 37 864**

---

## 4. Qualité des clés clients

Certaines clés clients présentent des différences de format susceptibles d’empêcher leur rapprochement avec la dimension client.

Exemples de problèmes possibles :

- espaces parasites ;
- différence de casse ;
- format hétérogène ;
- caractères inutiles.

### Impact

Avant normalisation :

**1 966 lignes**, soit environ **5,2 %** des ventes, présentent un risque de non-correspondance avec la dimension client.

Une mauvaise jointure peut provoquer :

- des clients non identifiés ;
- des segments manquants ;
- des analyses portefeuille erronées ;
- des agrégations incomplètes.

### Traitement

Les clés sont nettoyées avant la jointure :

- suppression des espaces inutiles ;
- normalisation du format ;
- harmonisation de la casse ;
- contrôles après rapprochement.

---

## 5. Harmonisation des canaux commerciaux

Les données sources contiennent plusieurs variantes de libellés représentant en réalité les mêmes canaux métier.

Le jeu source comporte **8 libellés différents**, ramenés à **3 canaux analytiques**.

Les catégories finales utilisées dans le dashboard sont notamment :

- Terrain ;
- Sédentaire (ADV) ;
- Web.

### Risque

Sans normalisation, deux écritures différentes d’un même canal seraient analysées comme deux catégories distinctes.

Cela fausserait :

- les répartitions de chiffre d’affaires ;
- les comparaisons par canal ;
- les filtres du dashboard.

### Traitement

Une règle de correspondance transforme les variantes de libellés en valeurs métier standardisées avant chargement dans `Dim_Canal`.

---

## 6. Contrôles de cohérence

La préparation ne se limite pas au nettoyage des données.

Des contrôles sont également utilisés pour vérifier la cohérence entre la source et le modèle.

Exemples :

- nombre de lignes avant / après transformation ;
- nombre de doublons supprimés ;
- contrôle des valeurs non rapprochées ;
- contrôle des catégories et canaux ;
- comparaison du chiffre d’affaires avant / après traitement ;
- vérification des types de données ;
- contrôle des valeurs nulles sur les champs structurants.

Une table technique `_Controles` est également présente dans le modèle Power BI afin de séparer les contrôles de qualité des mesures métier.

---

## 7. Résumé

| Contrôle | Avant traitement | Après traitement |
|---|---:|---:|
| Lignes de ventes | 37 879 | 37 864 |
| Doublons | 15 | 0 |
| Écart de CA lié aux doublons | 7 513 € | 0 € |
| Lignes à risque sur la clé client | 1 966 (≈5,2 %) | clés normalisées |
| Libellés de canaux | 8 | 3 |
| Formats ERP | 2 | 1 schéma analytique harmonisé |

---

## Ce que cette étape apporte au reporting

L’objectif n’est pas uniquement d’obtenir un fichier techniquement exploitable.

La préparation vise à garantir que les KPI affichés dans Power BI reposent sur :

**des données consolidées, cohérentes, rapprochables et reproductibles.**

Cette étape précède volontairement la modélisation et la visualisation : un dashboard ne peut être fiable que si les données qui l’alimentent le sont également.
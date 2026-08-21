# Sélection de mesures DAX

Cette section présente une sélection volontairement limitée de mesures utilisées dans le rapport Nolvéa Distribution.

L'objectif n'est pas de documenter l'intégralité du modèle, mais d'illustrer plusieurs problématiques DAX : intelligence temporelle, comparaison aux objectifs, calcul pondéré, ranking et règles métier.

---

## 1. Chiffre d'affaires du mois précédent

Cette mesure récupère le chiffre d'affaires du mois précédant la période courante.

```DAX
CA M-1 =
VAR _DateMax =
    MAX ( Dim_Date[Date] )
VAR _DebutMoisPrecedent =
    EOMONTH ( _DateMax, -2 ) + 1
VAR _FinMoisPrecedent =
    EOMONTH ( _DateMax, -1 )
RETURN
    CALCULATE (
        [CA],
        REMOVEFILTERS ( Dim_Date ),
        DATESBETWEEN (
            Dim_Date[Date],
            _DebutMoisPrecedent,
            _FinMoisPrecedent
        )
    )
```

**Objectif métier :** disposer d'une base de comparaison avec le mois précédent tout en conservant les autres filtres du rapport.

**Concepts DAX :** contexte de filtre, intelligence temporelle, `CALCULATE`, `REMOVEFILTERS`, `DATESBETWEEN`, `EOMONTH`.

---

## 2. Évolution du CA vs M-1

```DAX
Évolution CA vs M-1 % =
DIVIDE (
    [CA] - [CA M-1],
    [CA M-1]
)
```

**Objectif métier :** mesurer la progression ou le recul du chiffre d'affaires par rapport au mois précédent.

L'utilisation de `DIVIDE` permet également de gérer proprement les situations où la valeur du mois précédent est nulle.

---

## 3. Écart du CA par rapport à l'objectif

```DAX
Écart CA vs objectif % =
DIVIDE (
    [CA] - [Objectif CA],
    [Objectif CA]
)
```

La version en valeur absolue utilisée dans le modèle est :

```DAX
Écart CA vs objectif =
[CA] - [Objectif CA]
```

**Objectif métier :** mesurer immédiatement le niveau de surperformance ou de sous-performance par rapport à l'objectif commercial.

---

## 4. Taux de remise pondéré

Le calcul ne repose pas sur une moyenne simple des taux de remise ligne par ligne. La remise est évaluée par rapport au chiffre d'affaires brut avant remise.

Mesure support :

```DAX
CA brut avant remise =
SUMX (
    Fait_Ventes,
    Fait_Ventes[Quantite] * Fait_Ventes[PrixUnitaire]
)
```

Puis :

```DAX
Taux de remise pondéré =
DIVIDE (
    [CA brut avant remise] - [CA],
    [CA brut avant remise]
)
```

**Objectif métier :** obtenir un taux de remise représentatif du poids économique réel des ventes.

Cette approche évite qu'une petite commande et une commande à fort chiffre d'affaires aient artificiellement le même poids dans le calcul.

**Concepts DAX :** itération avec `SUMX`, mesure intermédiaire et calcul de ratio pondéré.

---

## 5. Rang commercial

Le ranking est calculé sur l'ensemble des commerciaux tout en conservant le contexte analytique du rapport, par exemple la période ou le canal sélectionné.

```DAX
Rang commercial =
VAR _CACommercial =
    [CA]

VAR _TableCommerciaux =
    CALCULATETABLE (
        ADDCOLUMNS (
            VALUES ( Dim_Commercial[CommercialID] ),
            "@CA", [CA]
        ),
        REMOVEFILTERS ( Dim_Commercial )
    )

RETURN
    RANKX (
        _TableCommerciaux,
        [@CA],
        _CACommercial,
        DESC,
        DENSE
    )
```

La version affichée dans le dashboard présente le rang sous la forme `8/11` :

```DAX
Rang commercial affiché =
VAR _Rang =
    [Rang commercial]

VAR _NbCommerciaux =
    COUNTROWS (
        CALCULATETABLE (
            VALUES ( Dim_Commercial[CommercialID] ),
            REMOVEFILTERS ( Dim_Commercial )
        )
    )

RETURN
    FORMAT ( _Rang, "0" )
        & "/"
        & FORMAT ( _NbCommerciaux, "0" )
```

**Objectif métier :** positionner un commercial par rapport à l'ensemble de l'équipe, même lorsqu'un commercial est sélectionné individuellement dans la page.

**Concepts DAX :** tables virtuelles, `ADDCOLUMNS`, `RANKX`, gestion explicite du contexte de filtre.

---

## 6. Produit à surveiller

Cette règle identifie les produits ayant un poids économique significatif mais une rentabilité inférieure au niveau de référence.

```DAX
Produit à surveiller =
VAR _CA = [CA]
VAR _CAMoyen = [CA moyen par produit]
VAR _Taux = [Taux de marge]
VAR _TauxRef = [Taux marge référence produits]
RETURN
    IF (
        _CA >= _CAMoyen
            && _Taux < _TauxRef,
        1,
        0
    )
```

**Logique métier :**

un produit est marqué comme étant à surveiller lorsque :

- son chiffre d'affaires est supérieur ou égal au CA moyen par produit ;
- son taux de marge est inférieur au taux de marge de référence.

Cette mesure ne cherche pas à déterminer automatiquement la cause de la faible rentabilité. Elle sert à constituer une population de produits à investiguer : prix de vente, coût d'achat, remises ou mix commercial.

**Concepts DAX :** variables, règle métier multicritère et mesure utilisée comme filtre analytique.

---

## Pourquoi cette sélection ?

Ces mesures illustrent plusieurs dimensions du modèle analytique :

| Problématique | Exemple |
|---|---|
| Intelligence temporelle | `CA M-1` |
| Analyse de variation | `Évolution CA vs M-1 %` |
| Pilotage des objectifs | `Écart CA vs objectif %` |
| Calcul pondéré | `Taux de remise pondéré` |
| Positionnement relatif | `Rang commercial affiché` |
| Détection métier | `Produit à surveiller` |

Le modèle complet contient d'autres mesures de chiffre d'affaires, marge, volume, objectifs, cumul et analyse produits. Cette sélection se concentre volontairement sur les calculs les plus représentatifs du raisonnement analytique mis en œuvre.
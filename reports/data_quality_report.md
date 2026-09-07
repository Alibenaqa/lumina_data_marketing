# Data quality report — Lumina & Co CRM export

**Fichiers analysés :** `customers.csv` (50 295 lignes), `transactions.csv`
(1 613 433 lignes), période du 2022-01-07 au 2026-06-30.

Ce rapport a été rédigé **avant toute transformation des données**. Il est le
pendant autonome de l'Étape 2 du notebook
[`notebooks/01_eda_cleaning.ipynb`](../notebooks/01_eda_cleaning.ipynb).

## 1. Près d'un quart des transactions ne peuvent être rattachées à aucun client

367 328 lignes de `transactions.csv` (22,8 %) ont un `customer_id` manquant,
pour un total de 11,5 M€ de chiffre d'affaires brut.

**Impact :** ce volume ne peut pas alimenter la segmentation RFM par client —
il doit être traité à part dans les analyses globales (chiffre d'affaires,
saisonnalité) et exclu des analyses par client.

## 2. `transactions.csv` est tronqué au niveau des lignes de facture — ce n'est pas `customers.csv` qui est gonflé

Première hypothèse testée et invalidée : `total_spent` n'est pas surestimé,
c'est le nombre de lignes de facture conservées par client dans
`transactions.csv` qui est incomplet. Preuve : en séparant les 4 736 clients
(9,6 % de la base) dont `total_spent` correspond exactement (± 3 %) au
chiffre d'affaires recalculé depuis le ledger, des 44 576 clients (90,4 %)
pour qui il ne correspond pas, le nombre de lignes par facture diffère
radicalement — 21,6 lignes/facture en moyenne (médiane 17,0) pour le premier
groupe, contre 2,6 (médiane 2,3) pour le second, un rapport (~8x) quasiment
identique au ratio médian d'écart de valeur observé (7,1x). Prix unitaires et
quantités sont statistiquement identiques entre les deux groupes (~20,1 € et
~1,55 en moyenne dans les deux cas) : ce sont les mêmes lignes, mais moins
nombreuses. Les factures existent bien (`n_orders` correspond exactement au
nombre de factures du ledger pour 74,9 % des clients, ± 10 % pour 79,4 %) ; la
troncature touche surtout les factures anciennes (moins de 1 % de clients
cohérents parmi ceux acquis en 2022-2023, contre 10-15 % parmi ceux acquis en
2024-2026) et épargne le mix produit (répartition par catégorie quasi
identique entre les deux groupes).

**Impact :** `customers.csv` (`total_spent`, `avg_basket`, `n_orders`,
`recency_days`, `tenure_days`) est la source de vérité pour la valeur client
et alimente directement le RFM du Jour 2, sans recalcul depuis
`transactions.csv`. Ce dernier reste fiable pour tout ce qui ne dépend pas du
nombre de lignes par facture : catégories, prix unitaires, dates, géographie,
et — validé indépendamment sur le sous-groupe à lignes complètes pour la
période récente (plateau, pics de juin) — la forme de la saisonnalité. Tout
montant absolu tiré de `transactions.csv` (CA mensuel, CA par catégorie en €)
reste une estimation basse.

## 3. La base est fortement biaisée géographiquement

45 904 clients sur 50 295 (91,3 %) sont en France ; le deuxième pays,
l'Allemagne, n'en compte que 918 (1,8 %).

**Impact :** toute segmentation ou tout comportement moyen calculé sur
l'ensemble de la base reflétera essentiellement le comportement français. Les
segments internationaux seront statistiquement fragiles (quelques centaines
d'individus) et ne doivent pas être généralisés.

## 4. Le chiffre d'affaires brut est pollué par des lignes qui ne sont pas des ventes

34 691 lignes ont une quantité négative (dont 19 494 rattachées à 8 292
factures d'avoir commençant par "C" — des retours), 10 897 lignes ont un prix
unitaire à 0 € et 177 un prix négatif, et 7 465 lignes appartiennent à la
catégorie "Frais" (frais de port, ajustements manuels, remises commerciales,
dons caritatifs, échantillons). 445 lignes sont des doublons stricts.

**Impact :** sans traitement, le chiffre d'affaires, le panier moyen et la
répartition par catégorie seraient surestimés ou faussés par des lignes qui
ne représentent pas des achats de produits.

## 5. Les données zero-party sont rares et ne signalent pas la valeur client

Les taux de remplissage vont de 19,8 % (`urban_density`) à 42,7 %
(`age_bracket`). Le montant dépensé moyen est quasiment identique entre
clients ayant renseigné une préférence et les autres (écart < 1 %).

**Impact :** ces colonnes ne peuvent pas servir seules à prioriser les
clients à forte valeur ; elles pourront enrichir le RFM du Jour 2 mais pas le
remplacer.

**Incohérence mineure à signaler :** la marque est présentée comme fondée il
y a 3 ans (donc en 2023), mais 1 502 transactions sont datées de 2022 — à
mentionner au CMO sans bloquer l'analyse.

---

*Traitement détaillé de chacun de ces points : voir Étape 3 du notebook.*

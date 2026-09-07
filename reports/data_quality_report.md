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

## 2. Les agrégats de `customers.csv` ne correspondent pas au grand livre transactionnel

`total_spent` est cohérent en interne avec `avg_basket`
(`avg_basket = total_spent / n_orders` exactement), mais la somme de
`total_spent` sur toute la base (169,7 M€) est plus de 4 fois supérieure au
chiffre d'affaires recalculé depuis `transactions.csv` (~40 M€, hors avoirs,
frais et lignes sans client). Client par client, le ratio médian est
d'environ 7. `n_orders`, en revanche, reste globalement cohérent avec le
nombre de factures observées dans `transactions.csv`.

**Impact :** on ne peut pas faire confiance à `total_spent` ni à
`avg_basket` de `customers.csv` pour la composante Monétaire d'un RFM — ces
métriques doivent être recalculées depuis `transactions.csv`.

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

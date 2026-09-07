# Lumina & Co — Data Marketing & Analytics (fil rouge DIA2)

Projet fil rouge du cours *Data Marketing et Analytics* (DIA2, Hetic). Sur 5
jours, on répond à trois questions posées par le CMO de Lumina & Co, marque
française de cosmétiques DTC fondée il y a 3 ans, en forte croissance jusqu'en
2024 puis en stagnation depuis :

1. « On envoie les mêmes emails à tous nos clients. C'est un problème ? »
   (Jours 1-2 — nettoyage, EDA, segmentation RFM)
2. « On ne sait pas quel canal fonctionne vraiment. » (Jour 3 — KPIs,
   attribution)
3. « Je veux des recommandations basées sur des tests. » (Jour 4 — A/B
   testing)

## Jour 1 — Nettoyage & EDA

Le notebook [`notebooks/01_eda_cleaning.ipynb`](notebooks/01_eda_cleaning.ipynb)
charge `customers.csv` et `transactions.csv`, documente un data quality
report, nettoie les données (avoirs, prix nuls/négatifs, codes non-produits,
`customer_id` manquants, doublons) et conduit une EDA orientée marketing
(saisonnalité, Pareto, géographie, catégories, zero-party data). Il se termine
par 3 à 5 hypothèses marketing qui seront testées en segmentation au Jour 2.

Le rapport de qualité des données autonome se trouve dans
[`reports/data_quality_report.md`](reports/data_quality_report.md).

## Structure du projet

```
data/
  raw/          fichiers sources, non modifiés
  processed/    données nettoyées (parquet)
notebooks/      notebooks Jupyter, un par jour de cours
src/            fonctions réutilisables (à venir)
outputs/figures/  graphiques exportés par les notebooks
reports/        livrables markdown autonomes
```

## Données

`data/raw/` contient `customers.csv`, `transactions.csv`, `touchpoints.csv`
et `campaigns.csv`. Ces fichiers **ne sont pas versionnés** (le plus gros
dépasse 100 Mo) : `data/` est exclu via `.gitignore`. Pour reproduire
l'analyse, placez les 4 CSV fournis par le data engineer dans `data/raw/`.

Le Jour 1 n'utilise que `customers.csv` et `transactions.csv`.
`touchpoints.csv` et `campaigns.csv` servent aux Jours 3 et 4.

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/01_eda_cleaning.ipynb
```

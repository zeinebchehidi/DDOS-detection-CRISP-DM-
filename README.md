# Détection d'attaques DDoS sur CIC-DDoS2019

Projet de fouille de données : préparation, visualisation et analyse
exploratoire du dataset CIC-DDoS2019, puis modélisation.

## Installation
1. `git clone https://github.com/zeinebchehidi/ddos-detection-cic-ddos2019.git`
2. `python -m venv .venv` puis activer l'environnement
3. `pip install -r requirements.txt`

## Données (non incluses dans le dépôt)
Télécharger les fichiers CSV sur https://www.unb.ca/cic/datasets/ddos-2019.html
et les placer dans `data/raw/`.

## Ordre d'exécution
1. `notebooks/01_exploration.ipynb` (préparation, visualisation, EDA)

## Structure
- `notebooks/` : notebooks d'analyse
- `data/raw/` : CSV originaux (ignoré par Git)
- `data/processed/` : fichiers Parquet nettoyés (ignoré par Git)
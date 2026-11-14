# Détection d'attaques DDoS sur CIC-DDoS2019

Projet de fouille et d'analyse de données appliqué à la cybersécurité : préparation, visualisation et analyse exploratoire du dataset **CIC-DDoS2019**, puis sélection d'attributs et détection d'attaques DDoS par approche hybride (supervisée et non supervisée).

**Auteure :** Zeineb Chehidi (Master en fouille et analyse de données)

---

## 1. Contexte

Une attaque par déni de service distribué (DDoS) inonde un service de requêtes pour le rendre indisponible aux utilisateurs légitimes. Les systèmes de détection d'intrusion (IDS) à base de signatures reconnaissent bien les attaques déjà documentées, mais échouent face aux variantes nouvelles. Les modèles de machine learning supervisés détectent bien les attaques vues à l'entraînement, mais généralisent mal à des attaques inconnues.

Ce projet se situe entre la **cybersécurité** (détection d'intrusion réseau) et la **fouille de données** (nettoyage, visualisation, sélection d'attributs, classification, détection d'anomalies, gestion du déséquilibre des classes).

## 2. Problématique

> Évaluer la capacité de modèles supervisés et non supervisés à détecter des attaques DDoS connues et inconnues sur CIC-DDoS2019, et étudier une approche hybride combinant les deux.

## 3. Objectifs

- Distinguer le trafic normal (`BENIGN`) du trafic malveillant.
- Identifier le type d'attaque (classification multi-classe).
- Déterminer les attributs réseau les plus discriminants.
- Mesurer la chute de performance d'un modèle supervisé face à des familles d'attaques jamais vues.
- Évaluer si la détection d'anomalies et une approche hybride compensent cette limite.
- Vérifier la généralisation sur un second dataset de flux (CIDDS-Coburg).

## 4. Le dataset : CIC-DDoS2019

- **Source :** Canadian Institute for Cybersecurity, Université du Nouveau-Brunswick : https://www.unb.ca/cic/datasets/ddos-2019.html
- **Principe :** trafic généré dans un banc d'essai contrôlé, converti en **flux réseau** avec CICFlowMeter. Chaque ligne décrit un flux (durée, nombre de paquets et d'octets, débits, intervalles entre paquets, drapeaux TCP, etc.).
- **Format utilisé :** fichiers CSV (les PCAP ne sont pas utilisés). Ici, 16 fichiers CSV issus de deux jours de capture.
- **Colonne cible :** `Label` (`BENIGN` ou le nom de l'attaque).
- **Classes observées dans l'échantillon (18) :** `BENIGN` et 17 types d'attaque : `DrDoS_DNS`, `DrDoS_LDAP`, `DrDoS_MSSQL`, `DrDoS_NetBIOS`, `DrDoS_NTP`, `DrDoS_SNMP`, `DrDoS_SSDP`, `DrDoS_UDP`, `LDAP`, `MSSQL`, `NetBIOS`, `Portmap`, `Syn`, `TFTP`, `UDP`, `UDP-lag`, `WebDDoS`.

**Particularités importantes**

- Chaque CSV contient presque un seul type d'attaque et très peu de trafic bénin (exemple : 253 553 lignes `DrDoS_DNS` pour 168 `BENIGN` dans le premier fichier).
- Le dataset est très déséquilibré : `TFTP` domine (environ 160 000 lignes dans l'échantillon) alors que `WebDDoS` compte environ 11 lignes et `Portmap` environ 180.
- Les données ne sont **pas incluses** dans ce dépôt (trop lourdes). Il faut les télécharger depuis la source officielle.

**Datasets complémentaires prévus**

- **CIDDS-Coburg** : validation croisée pour tester la généralisation.
- **NSL-KDD** : baseline légère, utile pour valider le pipeline.

## 5. Méthodologie en bref

1. **Échantillonnage stratégique :** lecture par morceaux (`chunksize`), conservation de **100 % du trafic bénin** et de **2 % des attaques**, conversion en `float32`.
2. **Nettoyage :** suppression des colonnes d'identification (Flow ID, IP, ports, timestamp) pour éviter les fuites de données, traitement des valeurs infinies et manquantes, des doublons et des colonnes constantes.
3. **Traçabilité :** le fichier source et le jour d'origine de chaque ligne sont conservés à part (`sample_meta.parquet`) pour le protocole d'attaques inconnues, puis retirés des attributs avant tout entraînement.
4. **Évaluation prévue :** F1 macro, PR-AUC et matrice de confusion (jamais l'accuracy seule), rééchantillonnage appliqué uniquement à l'entraînement, explicabilité avec SHAP.

## 6. Tâches et avancement

### Phase 1 : Cadrage et acquisition des données (terminée)
- [x] Choisir la problématique et le dataset
- [x] Télécharger et extraire les fichiers CSV
- [x] Créer l'arborescence du projet
- [x] Installer les bibliothèques

### Phase 2 : Préparation des données (notebook 01, en cours)

**2.1 Data preprocessing (terminée)**
- [x] Charger les fichiers par morceaux avec échantillonnage
- [x] Conserver le fichier et le jour d'origine de chaque ligne
- [x] Supprimer les colonnes d'identification
- [x] Traiter infinis, valeurs manquantes, doublons, colonnes constantes
- [x] Sauvegarder en Parquet : **419 244 lignes, 68 colonnes** (67 attributs + `Label`), dont 15,6 % de `BENIGN`

**2.2 Data viz (presque terminée)**
- [x] Distribution des classes (échelle logarithmique)
- [x] Comparaison bénin / attaque sur des attributs clés (durée, paquets, débit, taille des paquets)
- [ ] Répartition des attaques par fichier d'origine (le jour n'est pas encore correctement renseigné, voir « Points ouverts »)

**2.3 EDA, Exploratory Data Analysis (à faire)**
- [ ] Statistiques descriptives, asymétrie, valeurs aberrantes
- [ ] Matrice de corrélation et attributs redondants (corrélation supérieure à 0,95)
- [ ] Attributs qui distinguent le bénin des attaques et les familles d'attaques entre elles
- [ ] ACP : variance expliquée et projection en 2D

### Phase 3 : Sélection d'attributs (notebook 02, à faire)
- [ ] Information mutuelle
- [ ] Importance par Random Forest
- [ ] RFE
- [ ] Comparaison des sous-ensembles retenus et liste réduite d'attributs

### Phases suivantes (prévues, selon le périmètre final du projet)

**Phase 4 : Modèles supervisés (notebook 03)**
- [ ] Découpage entraînement / test
- [ ] Gestion du déséquilibre (pondération, SMOTE, sous-échantillonnage, sur l'entraînement uniquement)
- [ ] Random Forest et XGBoost
- [ ] Évaluation : F1 macro, PR-AUC, matrice de confusion

**Phase 5 : Détection d'anomalies (notebook 04)**
- [ ] Entraînement sur le trafic bénin uniquement
- [ ] Isolation Forest et autoencodeur
- [ ] Choix du seuil, courbes ROC et PR

**Phase 6 : Attaques inconnues et approche hybride (notebook 05)**
- [ ] Entraîner sur certaines familles d'attaques, tester sur des familles non vues
- [ ] Quantifier la chute de performance du modèle supervisé
- [ ] Approche hybride : le supervisé classe, la détection d'anomalies rattrape l'inconnu
- [ ] Validation croisée sur CIDDS-Coburg

**Phase 7 : Évaluation, explicabilité et rédaction (notebook 06)**
- [ ] Tableau comparatif final
- [ ] Explicabilité avec SHAP
- [ ] Analyse des limites et rédaction du rapport

## 7. Premiers constats (phase de visualisation)

- **Taille de l'échantillon :** 419 244 flux, 18 classes.
- **Déséquilibre marqué :** `TFTP` domine, `WebDDoS` (environ 11 lignes) et `Portmap` sont trop rares pour être apprises par un modèle. Ces classes devront être regroupées ou exclues des expériences multi-classes.
- **Attributs discriminants :** la taille moyenne des paquets avant (`Fwd Packet Length Mean`) et le débit (`Flow Bytes/s`) séparent nettement le bénin des attaques ; le nombre de paquets avant (`Total Fwd Packets`) discrimine peu.
- **Régularité du trafic d'attaque :** les attaques forment des pics très étroits, signe de trafic généré par des outils. Les modèles risquent donc d'obtenir des scores très élevés en laboratoire, ce qui justifie le test sur attaques inconnues et la validation croisée.

## 8. Limites connues

- La proportion de 15,6 % de `BENIGN` vient de l'échantillonnage (tout le bénin, 2 % des attaques) : elle ne reflète pas le dataset complet.
- Les jeux CIC présentent des problèmes d'étiquetage et des artefacts liés à CICFlowMeter documentés dans la littérature.
- Un seul fichier par type d'attaque donne un risque de sur-apprentissage sur les particularités de la capture.

## 9. Points ouverts

- **Nombre de fichiers :** 16 fichiers CSV ont été chargés, alors que les deux jours de capture en contiennent peut-être davantage. Des fichiers de même nom ont pu être écrasés lors de la copie dans un seul dossier. À vérifier.
- **Colonne `day` :** elle contient le nom du dossier (`raw`) et non le jour de capture. Elle sera corrigée en lisant directement les dossiers des deux jours.

## 10. Structure du projet

```
Projet_DDoS/
├── notebooks/
│   └── 01_exploration.ipynb   # préparation, visualisation, EDA
├── data/                      # ignoré par Git
│   ├── raw/                   # CSV originaux
│   └── processed/             # sample_clean.parquet, sample_meta.parquet
├── requirements.txt
├── .gitignore
└── README.md
```

## 11. Installation et utilisation

```bash
git clone https://github.com/zeinebchehidi/ddos-detection-cic-ddos2019.git
cd ddos-detection-cic-ddos2019

python -m venv .venv
.venv\Scripts\activate        # Windows (Linux/Mac : source .venv/bin/activate)
pip install -r requirements.txt
```

1. Télécharger les CSV sur la page officielle du dataset et les placer dans `data/raw/`.
2. Ouvrir `notebooks/01_exploration.ipynb` dans VS Code ou Jupyter, choisir le noyau du `.venv`.
3. Exécuter les cellules **dans l'ordre**. Le chargement des fichiers peut prendre de 10 à 30 minutes : il est conseillé de disposer d'au moins 8 Go de RAM libre. Les données nettoyées sont ensuite sauvegardées en Parquet pour ne pas refaire ce chargement.

## 12. Technologies

Python 3.10+, pandas, numpy, pyarrow, matplotlib, seaborn, scikit-learn, Jupyter. Prévus : imbalanced-learn, xgboost, shap.

## 13. Référence

Sharafaldin, I., Lashkari, A. H., Hakak, S., Ghorbani, A. A. : *Developing Realistic Distributed Denial of Service (DDoS) Attack Dataset and Taxonomy*, IEEE ICCST, 2019 (référence à vérifier avant citation dans le rapport).
# NetThreatAI
# NetThreatAI — ML-Powered Network Threat Detection

> Système de détection de menaces réseau en temps réel basé sur un ensemble de modèles Machine Learning (Random Forest + Isolation Forest + Autoencoder), déployable sur Kali Linux.

---

## Table des matières

- [Vue d'ensemble](#vue-densemble)
- [Architecture](#architecture)
- [Stack technique](#stack-technique)
- [Installation](#installation)
- [Dataset](#dataset)
- [Utilisation](#utilisation)
- [Modules](#modules)
- [Résultats attendus](#résultats-attendus)
- [Roadmap](#roadmap)
- [Avertissement légal](#avertissement-légal)

---

## Vue d'ensemble

NetThreatAI est un projet de cybersécurité avancé qui combine plusieurs algorithmes de Machine Learning pour détecter automatiquement des comportements malveillants dans le trafic réseau.

Le système fonctionne en deux modes :
- **Mode entraînement** : apprentissage sur le dataset CICIDS2017 (trafic labélisé normal vs attaques)
- **Mode temps réel** : capture du trafic live, extraction de features, inférence et alerte

### Ce que le projet détecte

| Type d'attaque | Méthode ML |
|---|---|
| DoS / DDoS | Random Forest (supervisé) |
| Port scanning | Random Forest + Isolation Forest |
| Anomalies inconnues | Autoencoder (non supervisé) |
| Comportements rares | Isolation Forest |

---

## Architecture

```
netthreatai/
├── capture/
│   └── sniffer.py          # Capture trafic réseau via Scapy
├── features/
│   ├── explore.py          # Exploration et stats du dataset
│   └── extractor.py        # Feature engineering des flows réseau
├── models/
│   ├── train.py            # Entraînement Random Forest + Isolation Forest
│   ├── autoencoder.py      # Autoencoder Keras pour anomalies
│   └── export_onnx.py      # Export modèles au format ONNX
├── inference/
│   └── realtime.py         # Pipeline d'inférence temps réel
├── data/
│   ├── Monday.csv          # Dataset CICIDS2017 (à télécharger)
│   └── download_dataset.sh # Script de téléchargement
├── dashboard/
│   └── app.py              # Dashboard Flask avec alertes live
├── venv/                   # Environnement virtuel Python
└── notes.md                # Journal de bord du projet
```

---

## Stack technique

| Composant | Technologie |
|---|---|
| OS | Kali Linux |
| Langage | Python 3.13 |
| Capture réseau | Scapy, PyShark |
| Feature engineering | Pandas, NumPy, Scikit-learn |
| Modèles ML | Random Forest, Isolation Forest (sklearn) |
| Deep Learning | Keras / TensorFlow (Autoencoder) |
| Inférence rapide | ONNX Runtime |
| Dashboard | Flask + Chart.js |
| Dataset | CICIDS2017 (University of New Brunswick) |

---

## Installation

### Prérequis

- Kali Linux (à jour)
- Python 3.10+
- 4 GB RAM minimum
- 5 GB d'espace disque libre

### Setup

```bash
# 1. Cloner / créer le projet
mkdir -p ~/netthreatai && cd ~/netthreatai

# 2. Environnement virtuel
python3 -m venv venv
source venv/bin/activate

# 3. Dépendances
pip install pandas numpy scikit-learn scapy pyshark \
            joblib matplotlib seaborn flask onnxruntime

# 4. (Optionnel) Deep Learning
pip install tensorflow tf2onnx
```

---

## Dataset

Ce projet utilise **CICIDS2017** (Canadian Institute for Cybersecurity Intrusion Detection System 2017).

### Téléchargement

```bash
cd ~/netthreatai/data

# Trafic normal (Monday)
wget -O Monday.csv "https://archive.org/download/cicids2017/Monday-WorkingHours.pcap_ISCX.csv"

# Trafic avec attaques (Tuesday — DoS, Hulk, etc.)
wget -O Tuesday.csv "https://archive.org/download/cicids2017/Tuesday-WorkingHours.pcap_ISCX.csv"
```

### Composition du dataset

| Fichier | Contenu | Volume |
|---|---|---|
| Monday | Trafic normal uniquement | ~530k flows |
| Tuesday | DoS Hulk, DoS GoldenEye, DoS Slowloris | ~445k flows |
| Wednesday | Heartbleed, DoS Slowhttptest | ~692k flows |
| Thursday | Web attacks (XSS, SQLi, Brute Force) | ~170k flows |
| Friday | Botnet, Port Scan, DDoS | ~288k flows |

---

## Utilisation

### 1. Explorer le dataset

```bash
cd ~/netthreatai/features
python3 explore.py
```

### 2. Entraîner les modèles

```bash
cd ~/netthreatai/models
python3 train.py
```

### 3. Capture et inférence temps réel

```bash
# Requiert les droits root pour capturer le trafic
sudo ~/netthreatai/venv/bin/python3 ~/netthreatai/inference/realtime.py
```

### 4. Dashboard

```bash
cd ~/netthreatai/dashboard
python3 app.py
# Accessible sur http://localhost:5000
```

---

## Modules

### `sniffer.py` — Capture réseau

Capture le trafic sur l'interface réseau et extrait les features brutes des paquets IP/TCP en temps réel.

**Features extraites :** IP source/destination, port source/destination, longueur du paquet, flags TCP, TTL, timestamp.

### `extractor.py` — Feature engineering

Transforme les paquets bruts en vecteurs de features compatibles avec les modèles ML.

**Normalisation :** StandardScaler, encodage des flags TCP en entier, agrégation par flow.

### `train.py` — Entraînement

Entraîne deux modèles complémentaires :
- **Random Forest** (supervisé) : classification binaire normal/malveillant
- **Isolation Forest** (non supervisé) : détection d'anomalies sans labels

**Métriques évaluées :** Accuracy, Precision, Recall, F1-Score, Matrice de confusion.

### `autoencoder.py` — Détection d'anomalies

Réseau de neurones entraîné uniquement sur le trafic normal. Une erreur de reconstruction élevée signale une anomalie.

**Architecture :** Input → Dense(32, ReLU) → Dense(16, ReLU) → Dense(32, ReLU) → Output

### `realtime.py` — Pipeline temps réel

Orchestre la capture, l'extraction de features et l'inférence en continu. Génère des alertes lorsqu'une menace est détectée.

### `app.py` — Dashboard Flask

Interface web affichant les alertes en temps réel, les statistiques de trafic et les prédictions des modèles.

---

## Résultats attendus

Après entraînement sur CICIDS2017 :

| Modèle | Accuracy | F1 (attaques) |
|---|---|---|
| Random Forest | ~98% | ~97% |
| Isolation Forest | ~85% | ~80% |
| Ensemble (vote) | ~99% | ~98% |

> Ces chiffres sont indicatifs et dépendent des fichiers du dataset utilisés.

---

## Roadmap

- [x] Structure du projet
- [x] Installation de l'environnement
- [ ] Téléchargement et exploration du dataset
- [ ] Nettoyage et feature engineering
- [ ] Entraînement Random Forest + Isolation Forest
- [ ] Autoencoder Keras
- [ ] Export ONNX
- [ ] Pipeline temps réel
- [ ] Dashboard Flask
- [ ] Tests d'évasion adversariale

---

## Avertissement 

Ce projet est développé **exclusivement à des fins éducatives et de recherche en cybersécurité**.

- Utiliser uniquement sur des réseaux et systèmes vous appartenant ou pour lesquels vous disposez d'une autorisation écrite explicite.
- Ne jamais déployer sur un réseau de production sans autorisation.
- L'auteur décline toute responsabilité en cas d'utilisation illégale ou malveillante.

---

## Ressources

- [CICIDS2017 — University of New Brunswick](https://www.unb.ca/cic/datasets/ids-2017.html)
- [Scikit-learn Documentation](https://scikit-learn.org/stable/)
- [Scapy Documentation](https://scapy.readthedocs.io/)
- [ONNX Runtime](https://onnxruntime.ai/)
- [Projet d'inspiration — CarterPerez-dev](https://github.com/CarterPerez-dev/Cybersecurity-Projects)

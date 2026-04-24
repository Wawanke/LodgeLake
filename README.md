# 🏕️ LodgeLake — Pipeline de données touristiques

> **Scraping → ETL → Data Lake → MongoDB → PostgreSQL → API REST → Dashboard BI**

[![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green?logo=fastapi)](https://fastapi.tiangolo.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-NoSQL-brightgreen?logo=mongodb)](https://www.mongodb.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-DW-blue?logo=postgresql)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/Licence-Open%20Licence-orange)](https://www.data.gouv.fr/fr/datasets/hebergements-touristiques-classes-en-france/)

---

## 📖 Description

**LodgeLake** est un pipeline de données complet centré sur les **hébergements touristiques classés en France** (source officielle : Atout France / data.gouv.fr).

Le projet couvre l'intégralité de la chaîne data :

- Scraping des données CSV depuis data.gouv.fr
- Nettoyage et transformation (ETL)
- Stockage Data Lake (CSV / JSON)
- Stockage MongoDB (collections RAW + CLEAN)
- Data Warehouse PostgreSQL (schéma analytique)
- Exposition via une API REST FastAPI
- Dashboard BI connecté à PostgreSQL (Power BI)
- Interface web frontend (HTML/CSS/JS)
- Visualisations interactives avec Streamlit

---

## 🗂️ Structure du projet

```
LodgeLake/
├── app/                # Logique applicative (ETL, ingestion, pipeline)
├── data/
│   ├── raw/            # Données brutes CSV (séparateur ;)
│   ├── clean/          # Données nettoyées CSV
│   └── lake/           # Export JSON (Data Lake)
├── frontend/           # Interface web (HTML / CSS / JS)
├── images/             # Captures Power BI et ressources visuelles
├── tests/              # Tests unitaires
├── visualisation/      # Dashboards Streamlit
├── main.py             # Point d'entrée principal
├── Dockerfile          # Image Docker de l'application
├── docker-compose.yml  # Orchestration des services
└── requirements.txt    # Dépendances Python
```

---

## 🏗️ Architecture

```
CSV Atout France (data.gouv.fr)
        │
        ▼
    Scraping
        │
        ▼
    Cleaning
(normalisation, typage, extraction département/étoiles)
        │
        ├──────────────────────────────────┐
        ▼                                  ▼
MongoDB (RAW + CLEAN)              Data Lake (CSV / JSON)
        │
        ▼
  PostgreSQL (DW)
  Schéma analytique
        │
        ├───────────────┐
        ▼               ▼
   API REST         Power BI
  (FastAPI)          (BI)
```

---

## 🛠️ Technologies

| Couche | Technologie |
|---|---|
| Langage | Python 3.11 |
| Transformation | Pandas |
| Base NoSQL | MongoDB |
| Data Warehouse | PostgreSQL |
| API REST | FastAPI |
| Visualisation | Streamlit, Power BI |
| Frontend | HTML / CSS / JavaScript |
| Conteneurisation | Docker / Docker Compose |

---

## 📦 Source des données

| Champ | Détail |
|---|---|
| **Jeu de données** | Hébergements touristiques classés en France |
| **Producteur** | Atout France – Agence de développement touristique |
| **Licence** | Licence Ouverte / Open Licence |
| **URL directe** | [hebergements_classes.csv](https://data.classement.atout-france.fr/static/exportHebergementsClasses/hebergements_classes.csv) |
| **Page data.gouv.fr** | [Lien officiel](https://www.data.gouv.fr/datasets/hebergements-touristiques-classes-en-france) |

Types d'hébergements couverts : Hôtel de tourisme, Camping, Village de vacances, Résidence de tourisme, Parc résidentiel de loisirs, Auberge collective.

---

## 🚀 Installation & Lancement

### 1. Cloner le dépôt

```bash
git clone https://github.com/Wawanke/LodgeLake.git
cd LodgeLake
```

### 2. Lancer les services Docker

```bash
docker compose down -v
docker compose up --build
```

### 3. Vérifier les containers actifs

```bash
docker ps
```

Containers attendus :

| Container | Rôle |
|---|---|
| `api` | API REST FastAPI |
| `postgres_dw` | Data Warehouse PostgreSQL |
| `mongo_db` | Stockage MongoDB |
| `app_data` | Pipeline ETL principal |

### 4. Accéder au site web LodgeLake

Puis naviguer vers : [http://localhost:8000/LodgeLake](http://localhost:8000/LodgeLake)

### 5. Accéder au Dashboard Streamlit

Puis naviguer vers : [http://localhost:8000/dashboard](http://localhost:8000/dashboard)

---

## ⚙️ Pipeline automatique

Le pipeline se déclenche **automatiquement au démarrage** des containers et exécute les étapes suivantes :

1. Téléchargement du CSV depuis Atout France (data.gouv.fr)
2. Sauvegarde RAW → `data/raw/hebergements_raw.csv`
3. Nettoyage et transformation des données
4. Insertion dans MongoDB (collection RAW)
5. Insertion dans MongoDB (collection CLEAN)
6. Export Data Lake JSON → `data/lake/hebergements.json`
7. Chargement dans PostgreSQL (DW)

---

## 🗄️ Base de données PostgreSQL

### Table principale

```sql
hebergements (
    id                  SERIAL PRIMARY KEY,
    nom                 TEXT,
    type_hebergement    TEXT,
    classement          TEXT,
    nb_etoiles          FLOAT,
    adresse             TEXT,
    code_postal         TEXT,
    commune             TEXT,
    departement         TEXT,
    site_internet       TEXT,
    capacite            FLOAT,
    nb_chambres         FLOAT,
    nb_emplacements     FLOAT,
    date_classement     TEXT,
    classement_proroge  TEXT
)
```

### Tables de dimension

```sql
dim_type_hebergement (id SERIAL PRIMARY KEY, type_hebergement TEXT UNIQUE)
dim_departement      (id SERIAL PRIMARY KEY, departement TEXT UNIQUE)
```

### Vérifier les données PostgreSQL

```bash
docker exec -it postgres_dw psql -U postgres
\c dw
SELECT * FROM hebergements LIMIT 5;
SELECT type_hebergement, COUNT(*) FROM hebergements GROUP BY type_hebergement;
```

---

## 🌐 API REST (FastAPI)

### Lancer l'API

```bash
docker compose up --build api
```

### Documentation interactive

[http://localhost:8000/docs](http://localhost:8000/docs)

### Endpoints disponibles

| Méthode | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Statut de l'API |
| `GET` | `/hebergements` | Liste filtrée et paginée |
| `GET` | `/hebergements/{id}` | Détail par ID |
| `GET` | `/stats` | Statistiques globales |
| `GET` | `/stats/type` | Stats par type d'hébergement |
| `GET` | `/stats/departement` | Stats par département (top 20) |
| `GET` | `/stats/classement` | Stats par classement |
| `GET` | `/top` | Top hébergements par capacité |
| `GET` | `/recherche?q=` | Recherche par nom d'établissement |

### Paramètres de filtrage (`/hebergements`)

| Paramètre | Type | Exemple |
|---|---|---|
| `limit` | int | `10` |
| `offset` | int | `0` |
| `type_hebergement` | string | `HÔTEL DE TOURISME` |
| `departement` | string | `75`, `13`, `69` |
| `nb_etoiles` | float | `3`, `4`, `5` |
| `commune` | string | `PARIS`, `LYON` |

---

## 📊 Streamlit

### Lancer le tableau de bord

```bash
docker compose up --build streamlit
```

Accès : [http://localhost:8501](http://localhost:8501)

---

## 🧹 Qualité des données

Traitements ETL appliqués :

- Suppression des doublons
- Suppression des lignes sans nom ni commune
- Remplacement des valeurs `"-"` par `null`
- Renommage des colonnes (suppression accents, majuscules, parenthèses)
- Conversion des colonnes numériques (`capacite`, `nb_chambres`, `nb_emplacements`)
- Extraction du département depuis le code postal (`str[:2]`)
- Extraction du nombre d'étoiles en valeur numérique depuis la colonne `classement`

---

## 📈 Power BI

Connexion directe à PostgreSQL :

| Paramètre | Valeur |
|---|---|
| Serveur | `localhost` |
| Port | `5432` |
| Base | `dw` |
| Utilisateur | `postgres` |
| Mot de passe | `postgres` |

Visuels disponibles :

- Répartition des hébergements par type
- Nombre d'hébergements par département
- Moyenne des étoiles par type
- Capacité totale par région
- Top établissements par capacité

---

## ⚠️ Limites connues

- Pas de coordonnées GPS dans le dataset source (pas de carte géographique)
- Pas de streaming de données en temps réel
- Couverture de tests unitaires limitée

---

## 🔭 Améliorations possibles

- Tests Pytest complets
- Orchestration avec Apache Airflow
- CI/CD via GitHub Actions
- Enrichissement avec un second dataset (ex : fréquentation touristique)

---

## ✅ Ce que démontre ce projet

- Ingestion de données open data officielles (data.gouv.fr)
- Transformation ETL complète et traçable
- Stockage multi-systèmes (MongoDB + PostgreSQL + Data Lake)
- Exposition API REST avec filtres, pagination et statistiques
- Exploitation BI via Power BI et Streamlit
- Conteneurisation complète avec Docker Compose

---

## 📄 Licence

Données source sous **Licence Ouverte / Open Licence** — Atout France / data.gouv.fr.

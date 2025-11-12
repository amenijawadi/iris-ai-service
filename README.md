#  Iris AI Service - MLOps avec Docker, Docker Compose et Monitoring

##  Description du Projet

Ce projet démontre la containerisation complète d'un service d'intelligence artificielle pour la classification des fleurs Iris. Il comprend :
- **API Backend** : FastAPI pour les prédictions ML
- **Frontend** : Interface React pour l'interaction utilisateur
- **Monitoring** : Prometheus pour la collecte de métriques et Grafana pour la visualisation

##  Architecture

```
iris-ai-service/
├── api/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       └── main.py
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   └── src/
├── monitoring/
│   └── prometheus.yml
├── docker-compose.yml
└── README.md
```

##  Technologies Utilisées

- **Backend** : Python 3.11, FastAPI, Uvicorn
- **Frontend** : React, Node.js 20, Nginx
- **Containerisation** : Docker, Docker Compose
- **Monitoring** : Prometheus, Grafana
- **ML** : Scikit-learn (modèle Iris)

## Services et Ports

| Service    | Port  | Description                          |
|------------|-------|--------------------------------------|
| API        | 8000  | API FastAPI avec endpoints ML        |
| Frontend   | 5174  | Interface utilisateur React          |
| Prometheus | 9090  | Collecte et stockage des métriques   |
| Grafana    | 3000  | Dashboards de visualisation          |

## 🔧 Prérequis

- Docker version 20.10+
- Docker Compose version 2.0+
- Git
- 4 Go de RAM minimum

##  Installation et Démarrage

### 1. Cloner le projet

```bash
git clone git@gitlab.com:<votre_utilisateur>/iris-ai-service.git
cd iris-ai-service
```

### 2. Configuration des fichiers

Vérifie que les fichiers suivants sont présents dans ton projet :

api/Dockerfile

frontend/Dockerfile

docker-compose.yml

monitoring/prometheus.yml

📁 api/Dockerfile

# Dockerfile du backend FastAPI
FROM python:3.11-slim

# Empêche Python de bufferiser les sorties (logs visibles en direct)
ENV PYTHONUNBUFFERED=1

# Définir le répertoire de travail
WORKDIR /app

# Copier et installer les dépendances
COPY requirements.txt /app/requirements.txt
RUN pip install --no-cache-dir -r /app/requirements.txt

# Copier le code source
COPY app /app/app

# Exposer le port 8000
EXPOSE 8000

# Démarrer le serveur FastAPI avec uvicorn
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]





📁 frontend/Dockerfile

# Étape 1 : build du projet React/Vite
FROM node:20-alpine AS build
WORKDIR /app

# Installer les dépendances
COPY package*.json ./
RUN npm install

# Copier le code et construire le build de production
COPY . .
RUN npm run build

# Étape 2 : servir l’application avec Nginx
FROM nginx:alpine

# Copier le fichier de configuration Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copier le build Vite dans le dossier web de Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Exposer le port HTTP
EXPOSE 80

# Lancer Nginx au premier plan
CMD ["nginx", "-g", "daemon off;"]








🧱 docker-compose.yml

services:
  api:
    build: ./api
    container_name: iris_api
    ports:
      - "8000:8000"
    networks:
      - monitoring_net

  frontend:
    build: ./frontend
    container_name: iris_frontend
    depends_on:
      - api
    ports:
      - "5174:80"
    networks:
      - monitoring_net

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
    ports:
      - "9090:9090"
    networks:
      - monitoring_net

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
    ports:
      - "3000:3000"
    networks:
      - monitoring_net

networks:
  monitoring_net:
    driver: bridge






⚙️ monitoring/prometheus.yml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: "iris-api"
    static_configs:
      - targets: ["api:8000"]
    metrics_path: /metrics

  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]




### 3. Lancer tous les services

```bash
# Construire et démarrer tous les conteneurs
docker compose up --build

# Ou en arrière-plan
docker compose up --build -d
```

### 4. Vérifier que tout fonctionne

```bash
# Vérifier les conteneurs en cours d'exécution
docker compose ps

# Vérifier les logs
docker compose logs -f
```

## 🧪 Test des Services Individuels

### Test de l'API

```bash
# Build de l'image
cd api
docker build -t iris-api:dev .

# Lancer le conteneur
docker run -d -p 8000:8000 --name iris-api iris-api:dev

# Tester le healthcheck
curl http://localhost:8000/health

# Accéder à la documentation Swagger
# Ouvrir dans le navigateur : http://localhost:8000/docs
```

### Test du Frontend

```bash
# Build de l'image
cd frontend
docker build -t iris-frontend:dev .

# Lancer le conteneur
docker run -d -p 5174:80 --name iris-frontend iris-frontend:dev

# Accéder à l'interface
# Ouvrir dans le navigateur : http://localhost:5174
```

## 📊 Configuration du Monitoring

### Prometheus

Prometheus collecte automatiquement les métriques de l'API sur `http://api:8000/metrics`.

**Accès** : http://localhost:9090

**Fichier de configuration** : `monitoring/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'iris-api'
    static_configs:
      - targets: ['api:8000']
```

### Grafana

**Accès** : http://localhost:3000

**Identifiants par défaut** :
- Username : `admin`
- Password : `admin`

#### Configuration initiale de Grafana :

1. Connectez-vous à http://localhost:3000
2. Allez dans **Configuration** > **Data Sources**
3. Ajoutez Prometheus avec l'URL : `http://prometheus:9090`
4. Importez un dashboard pour le monitoring (ID recommandé : 1860 pour Node Exporter)

## 🔍 Endpoints de l'API

### Health Check
```bash
GET http://localhost:8000/health
```

### Documentation Interactive
```bash
GET http://localhost:8000/docs
```

### Prédiction Iris
```bash
POST http://localhost:8000/predict
Content-Type: application/json

{
  "sepal_length": 5.1,
  "sepal_width": 3.5,
  "petal_length": 1.4,
  "petal_width": 0.2
}
```







---


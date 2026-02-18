# SDLE Deployment — CD Pipeline

## Description
Repository de **déploiement continu (CD)** de l'application SDLE. Ce repo orchestre le déploiement de tous les services via Docker Compose.

## Architecture de l'application

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│  PostgreSQL   │
│  (Next.js)   │     │ (Spring Boot)│     │     16        │
│  Port: 3000  │     │  Port: 8081  │     │  Port: 5432   │
└──────────────┘     └──────────────┘     └──────────────┘
                                                │
                                          ┌─────▼────────┐
                                          │   pgAdmin     │
                                          │  Port: 5050   │
                                          └──────────────┘
```

## Services déployés

| Service | Image | Port |
|---------|-------|------|
| Frontend | `yadex34/sdle-frontend:latest` | 3000 |
| Backend | `yadex34/sdle-backend:latest` | 8081 |
| PostgreSQL | `postgres:16-alpine` | 5432 |
| pgAdmin | `dpage/pgadmin4:latest` | 5050 |

## Pipeline CD (Jenkins)

```
Pipeline Jenkins :
├── Checkout du code de déploiement
├── Pull des dernières images depuis Docker Hub
├── Arrêt des services existants
├── Déploiement avec Docker Compose
└── Health Check de tous les services
```

## URLs de l'application
- **Frontend** : http://localhost:3000
- **Backend API** : http://localhost:8081
- **pgAdmin** : http://localhost:5050

## Lancer manuellement
```bash
# Déployer
docker compose up -d

# Voir les logs
docker compose logs -f

# Arrêter
docker compose down

# Mise à jour (sans interruption)
docker compose pull
docker compose up -d --force-recreate
```

## Fichiers
| Fichier | Description |
|---------|-------------|
| `docker-compose.yml` | Orchestration des services |
| `Jenkinsfile` | Pipeline CD Jenkins |
| `.env` | Variables d'environnement |

## Prérequis
- Docker & Docker Compose installés
- Images poussées sur Docker Hub (`yadex34/sdle-backend`, `yadex34/sdle-frontend`)
- Jenkins avec credentials Docker Hub configurés

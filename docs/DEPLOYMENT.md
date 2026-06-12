# 🚀 Guide de Déploiement - Sakado

Ce document décrit les procédures de déploiement pour tous les environnements.

## 📋 Table des matières

1. [Environnements](#environnements)
2. [Prérequis](#prérequis)
3. [Déploiement Local](#déploiement-local)
4. [Déploiement Staging](#déploiement-staging)
5. [Déploiement Production](#déploiement-production)
6. [Monitoring](#monitoring)
7. [Rollback](#rollback)
8. [Troubleshooting](#troubleshooting)

## 🌍 Environnements

| Env | URL | Auto-Deploy | Branche |
|-----|-----|-------------|---------|
| Development | localhost:3000 | ❌ Manuel | `develop` |
| Staging | staging.sakado.digital | ✅ Auto PR | `staging` |
| Production | sakado.digital | ✅ Auto Merge | `main` |

## ✅ Prérequis

### Localement
- Docker & Docker Compose >= 2.0
- Node.js >= 18.x
- npm >= 9.x
- Git

### Serveurs
- Ubuntu 22.04 LTS (recommandé)
- Docker & Docker Compose
- 2GB RAM minimum
- 10GB stockage libre

## 🏠 Déploiement Local

### Installation rapide

```bash
# Cloner le projet
git clone https://github.com/SakadoDigital/Sakado.git
cd Sakado

# Copier la config d'exemple
cp .env.example .env

# Démarrer tous les services
docker-compose -f docker-compose.dev.yml up -d

# Vérifier que tout fonctionne
docker-compose -f docker-compose.dev.yml ps
```

### Accès
- 🌐 Frontend: http://localhost:3000
- 🔌 API: http://localhost:5000
- 🗄️ DB: localhost:5432
- 💾 Redis: localhost:6379

### Arrêter
```bash
docker-compose -f docker-compose.dev.yml down

# Avec suppression des volumes (réinitialiser DB)
docker-compose -f docker-compose.dev.yml down -v
```

## 🎯 Déploiement Staging

### Configuration du serveur

```bash
# SSH sur le serveur staging
ssh user@staging.sakado.digital

# Cloner le projet
git clone https://github.com/SakadoDigital/Sakado.git
cd Sakado

# Checkout branche staging
git checkout staging

# Copier la config
cp .env.example .env

# Éditer .env avec les valeurs staging
nano .env
```

### Configuration .env pour Staging

```env
APP_ENV=staging
APP_URL=https://staging.sakado.digital
API_URL=https://staging.sakado.digital/api

NODE_ENV=staging
DB_HOST=postgres
DB_NAME=sakado_staging
DB_USER=sakado_staging_user
DB_PASSWORD=STRONG_PASSWORD_HERE

JWT_SECRET=staging_jwt_secret_key
```

### Déployer

```bash
# Démarrer les services
docker-compose up -d

# Vérifier les logs
docker-compose logs -f

# Migrations base de données
docker-compose exec server npm run migrate:latest

# Vérifier la santé
curl https://staging.sakado.digital/health
```

### CI/CD Auto-Deploy

Lors d'une PR vers `staging`:
1. Tests automatiques s'exécutent
2. Si succès, build Docker
3. Push image Docker Registry
4. Deploy auto sur staging
5. Tests d'intégration

## 🚀 Déploiement Production

### ⚠️ Checklist pré-déploiement

- [ ] Tests locaux passent
- [ ] Revue de code complétée
- [ ] PR fusionnée sur `main`
- [ ] Version bumpée (package.json)
- [ ] CHANGELOG.md mis à jour
- [ ] Pas de secrets en dur dans le code
- [ ] Database migrations testées
- [ ] Monitoring configuré

### Configuration du serveur Production

```bash
# SSH sur le serveur prod
ssh user@prod.sakado.digital

# Cloner le projet
git clone https://github.com/SakadoDigital/Sakado.git
cd Sakado

# Checkout main
git checkout main

# Configuration
cp .env.example .env
nano .env
```

### Configuration .env Production

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://sakado.digital
API_URL=https://sakado.digital/api

NODE_ENV=production
SERVER_PORT=5000

DB_HOST=postgres
DB_NAME=sakado_prod
DB_USER=sakado_prod_user
DB_PASSWORD=VERY_STRONG_PASSWORD_HERE

JWT_SECRET=VERY_STRONG_JWT_SECRET_KEY
JWT_EXPIRE=24h

# Emails
SMTP_HOST=smtp.sendgrid.net
SMTP_USER=apikey
SMTP_PASS=SG.xxxx...

# Logs
LOG_LEVEL=warn
LOG_FORMAT=json
```

### Déploiement

```bash
# Mise à jour code
git pull origin main

# Démarrer services
docker-compose up -d

# Vérifier les logs
docker-compose logs -f

# Migrations
docker-compose exec server npm run migrate:latest

# Vérifier santé
curl https://sakado.digital/health
```

### Post-déploiement

```bash
# Santé check
curl https://sakado.digital/api/health

# Logs
docker-compose logs --tail=100

# Performance
docker stats

# Database
docker-compose exec postgres psql -U sakado_prod_user sakado_prod
```

## 📊 Monitoring

### Docker Health Check

```bash
# Statut des services
docker-compose ps

# Logs temps réel
docker-compose logs -f

# Logs filtrés
docker-compose logs -f server
docker-compose logs --tail=50 client
```

### Système

```bash
# Utilisation ressources
docker stats

# Disque
df -h

# Processus
ps aux | grep docker
```

### Application

```bash
# Health check
curl https://sakado.digital/api/health

# Logs applicatifs
docker-compose exec server tail -f logs/app.log
```

## ⏮️ Rollback

### Rollback rapide

```bash
# Retourner au commit précédent
git revert HEAD

# Rebuild et redémarrer
docker-compose up -d --build

# Migrations revert si nécessaire
docker-compose exec server npm run migrate:rollback
```

### Rollback complet avec backup

```bash
# Backup base actuelle
docker-compose exec postgres pg_dump -U sakado_prod_user sakado_prod > backup_$(date +%s).sql

# Restore backup précédent
docker-compose exec postgres psql -U sakado_prod_user sakado_prod < backup_TIMESTAMP.sql

# Redémarrer services
docker-compose restart
```

## 🔧 Troubleshooting

### Service ne démarre pas

```bash
# Vérifier les logs
docker-compose logs server

# Vérifier les ports
sudo lsof -i :5000
sudo lsof -i :3000

# Redémarrer
docker-compose restart

# Rebuild
docker-compose up -d --build
```

### Erreur base de données

```bash
# Vérifier la connexion
docker-compose exec postgres psql -U sakado_prod_user -h postgres sakado_prod

# Vérifier les migrations
docker-compose exec server npm run migrate:status

# Reset base (⚠️ développement seulement)
docker-compose down -v
docker-compose up -d
```

### Erreur mémoire

```bash
# Augmenter limites Docker
# Éditer docker-compose.yml:

services:
  server:
    mem_limit: 1g
    memswap_limit: 1g
```

### Problèmes réseau/CORS

```bash
# Vérifier CORS config
docker-compose exec server env | grep CORS

# Vérifier logs CORS
docker-compose logs server | grep -i cors
```

## 📝 Commandes utiles

```bash
# Démarrer
docker-compose up -d

# Arrêter
docker-compose stop

# Redémarrer
docker-compose restart

# Supprimer tout
docker-compose down -v

# Logs
docker-compose logs -f --tail=50

# Exec commande
docker-compose exec server npm run seed

# Exec bash
docker-compose exec server sh
```

## 🆘 Support

- 📧 DevOps: devops@sakado.digital
- 📚 Docs: [GitHub Docs](https://github.com/SakadoDigital/Sakado/docs/)
- 🐛 Issues: [GitHub Issues](https://github.com/SakadoDigital/Sakado/issues)

---

**Dernière mise à jour**: 2026-06-12

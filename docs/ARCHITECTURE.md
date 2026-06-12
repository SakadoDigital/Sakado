# 🏗️ Architecture - Sakado

## Vue d'ensemble

Sakado utilise une architecture moderne **microservices-friendly** avec React frontend et Node.js backend, déployée via Docker.

```
┌─────────────────────────────────────────────────────────┐
│                    Internet                              │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS
       ┌───────────────▼──────────────┐
       │   Nginx Reverse Proxy        │
       │   (Load Balancer)            │
       └───────┬───────────────┬──────┘
               │               │
     ┌─────────▼────┐   ┌──────▼─────────┐
     │   Client     │   │    Server      │
     │  (React)     │   │  (Express.js)  │
     │  Port 3000   │   │   Port 5000    │
     └─────────┬────┘   └────────┬───────┘
               │                 │
               │    ┌────────────┴───────────┐
               │    │                        │
         ┌─────▼────────┐         ┌──────────▼────────┐
         │  PostgreSQL  │         │  Redis Cache      │
         │  Port 5432   │         │  Port 6379        │
         └──────────────┘         └───────────────────┘
```

## 🔌 Composants

### Frontend (Client)
- **Framework**: React 18
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **HTTP**: Axios
- **State Management**: Context API / Redux
- **Port**: 3000

**Structure:**
```
client/
├── public/
├── src/
│   ├── components/      # Composants réutilisables
│   ├── pages/          # Pages principales
│   ├── hooks/          # Hooks personnalisés
│   ├── services/       # Appels API
│   ├── store/          # State management
│   ├── styles/         # Styles globaux
│   ├── types/          # Types TypeScript
│   └── App.tsx
└── package.json
```

### Backend (Server)
- **Runtime**: Node.js 18+
- **Framework**: Express.js
- **Language**: JavaScript/TypeScript
- **Database**: PostgreSQL
- **Cache**: Redis
- **Auth**: JWT
- **Port**: 5000

**Structure:**
```
server/
├── src/
│   ├── routes/         # Définition des routes
│   ├── controllers/     # Logique métier
│   ├── models/         # Schémas et validations
│   ├── middleware/     # Middleware Express
│   ├── services/       # Services métier
│   ├── utils/          # Fonctions utilitaires
│   ├── config/         # Configuration
│   ├── database/       # Setup DB
│   └── app.js          # App Express
├── tests/              # Tests unitaires/intégration
├── migrations/         # DB migrations
├── Dockerfile
└── package.json
```

### Base de Données
- **Type**: PostgreSQL 15
- **Stockage**: Volume Docker persistant
- **Port**: 5432
- **Migrations**: Knex.js

**Schéma principal:**
```
Tables:
├── users              # Utilisateurs
├── services           # Services proposés
├── projects           # Projets clients
├── invoices           # Factures
├── payments           # Paiements
└── audit_logs         # Logs d'audit
```

### Cache
- **Type**: Redis 7
- **Usage**: Sessions, Cache DB, Queues
- **Port**: 6379

## 🔐 Sécurité

### Authentification
- JWT avec expiration 24h
- Refresh tokens stockés en DB
- Hachage bcrypt pour passwords

### Autorisation
- RBAC (Role-Based Access Control)
- Rôles: admin, manager, user, guest
- Permissions par endpoint

### Communication
- HTTPS/TLS en production
- CORS configuré
- Rate limiting
- Input validation/sanitization

### Données
- Connexion SSL PostgreSQL
- Encryption at rest (optionnel)
- Audit logging
- GDPR compliant

## 📦 Déploiement

### Conteneurisation
Chaque service a son Dockerfile:
- **Client**: Multi-stage build, nginx
- **Server**: Node.js slim
- **PostgreSQL**: Image officielle
- **Redis**: Image officielle

### Orchestration
- **Development**: docker-compose.dev.yml
- **Production**: docker-compose.yml
- **Optionnel**: Kubernetes ready

### CI/CD
GitHub Actions:
1. Tests (unit + intégration)
2. Linting & Format check
3. Build Docker
4. Push Registry
5. Deploy (staging/prod)

## 🔄 Flux API

### Request/Response
```
Request Flow:
Client → HTTP → Nginx → Express → Controller → Service → DB

Response Flow:
DB → Service → Controller → Serializer → Express → HTTP → Client
```

### Error Handling
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [...]
  }
}
```

## 📊 Scalabilité

### Horizontal Scaling
```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    └────┬─────────┬──┘
                         │         │
        ┌────────────────┘         └───────────────┐
        │                                           │
   ┌────▼───────┐                           ┌──────▼────┐
   │  Server 1  │                           │  Server 2 │
   │ (Port 5001)│                           │ (Port 5002)
   └────────────┘                           └───────────┘
        │                                           │
        └────────────────┬──────────────────────────┘
                         │
                  ┌──────▼─────────┐
                  │  PostgreSQL    │
                  │  (partagée)    │
                  └────────────────┘
```

### Database
- Connection pooling
- Réplication (optionnel)
- Backups automatiques

## 🧪 Testing

### Types
- **Unit**: Logique métier
- **Integration**: API endpoints
- **E2E**: Workflows complets
- **Load**: Performance

### Commandes
```bash
npm run test              # Unit tests
npm run test:integration # Integration tests
npm run test:e2e         # E2E tests
npm run test:coverage    # Coverage report
```

## 📈 Monitoring

### Logs
- Application logs → stdout → Docker
- Format JSON pour parsing
- ELK Stack optionnel

### Métriques
- CPU, Mémoire, Disque
- Requêtes HTTP, latence
- Database performance

### Alertes
- Health checks
- Error tracking (Sentry)
- Uptime monitoring

## 🔄 CI/CD Pipeline

```
                          GitHub
                             │
                ┌────────────┴────────────┐
                │                         │
              Push                    Pull Request
                │                         │
        ┌───────▼────────┐       ┌───────▼────────┐
        │   Build & Test │       │   Build & Test │
        │    (on 'main')  │       │   (on PR)      │
        └───────┬────────┘       └───────┬────────┘
                │                        │
        ┌───────▼────────┐       ┌───────▼────────┐
        │ Build Docker   │       │  Staging Auto  │
        │ Push Registry  │       │     Deploy     │
        └───────┬────────┘       └────────────────┘
                │
        ┌───────▼────────┐
        │ Prod Auto      │
        │ Deploy on      │
        │ merge to main  │
        └────────────────┘
```

## 🚀 Prochaines améliorations

- [ ] Kubernetes support
- [ ] GraphQL API
- [ ] WebSocket real-time
- [ ] Microservices split
- [ ] Message Queue (RabbitMQ)
- [ ] Advanced monitoring (Prometheus)
- [ ] Multi-region deployment

---

**Dernière mise à jour**: 2026-06-12

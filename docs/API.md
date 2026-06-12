# 📖 Documentation API - Sakado

## Vue d'ensemble

L'API Sakado est une API RESTful construite avec Express.js et PostgreSQL.

**Base URL**: `https://api.sakado.digital` (production)  
**Version**: 1.0.0  
**Format**: JSON  
**Authentification**: JWT Bearer Token

---

## 🔐 Authentification

### Obtenir un token

```bash
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Réponse (200 OK):**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 86400,
    "user": {
      "id": "user-123",
      "email": "user@example.com",
      "role": "user"
    }
  }
}
```

### Utiliser le token

```bash
GET /api/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Rafraîchir le token

```bash
POST /api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## 👥 Endpoints Utilisateurs

### List Users

```bash
GET /api/users?page=1&limit=10&role=admin
Authorization: Bearer <token>
```

**Query Parameters:**
- `page` (int): Numéro de page (défaut: 1)
- `limit` (int): Nombre d'items par page (défaut: 10, max: 100)
- `role` (string): Filtrer par rôle (admin, manager, user)
- `search` (string): Rechercher par email/nom

**Réponse (200 OK):**
```json
{
  "success": true,
  "data": {
    "users": [...],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 45,
      "pages": 5
    }
  }
}
```

### Get User

```bash
GET /api/users/:id
Authorization: Bearer <token>
```

### Create User

```bash
POST /api/users
Authorization: Bearer <token> (admin only)
Content-Type: application/json

{
  "email": "newuser@example.com",
  "password": "SecurePassword123!",
  "firstName": "John",
  "lastName": "Doe",
  "role": "user"
}
```

### Update User

```bash
PATCH /api/users/:id
Authorization: Bearer <token>
Content-Type: application/json

{
  "firstName": "Jane",
  "role": "manager"
}
```

### Delete User

```bash
DELETE /api/users/:id
Authorization: Bearer <token> (admin only)
```

---

## 🛠️ Gestion des Erreurs

### Format d'erreur standard

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

### Codes d'erreur courants

| Code | HTTP | Description |
|------|------|-------------|
| `VALIDATION_ERROR` | 400 | Données invalides |
| `UNAUTHORIZED` | 401 | Token manquant/invalide |
| `FORBIDDEN` | 403 | Accès refusé |
| `NOT_FOUND` | 404 | Ressource non trouvée |
| `CONFLICT` | 409 | Conflit (ex: email existe déjà) |
| `RATE_LIMIT` | 429 | Trop de requêtes |
| `SERVER_ERROR` | 500 | Erreur serveur |

---

## 📊 Pagination

Toutes les listes utilisent la pagination:

```json
{
  "success": true,
  "data": {
    "items": [...],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 245,
      "pages": 25,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  }
}
```

---

## 🔄 Rate Limiting

L'API implémente le rate limiting:

```
100 requêtes par 15 minutes par IP
1000 requêtes par 15 minutes par utilisateur authentifié
```

**Headers de réponse:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1702345678
```

---

## 📝 Rôles et Permissions

| Rôle | Permissions |
|------|------------|
| `admin` | Accès complet |
| `manager` | Gestion des utilisateurs et rapports |
| `user` | Accès aux données personnelles |
| `guest` | Accès lecture seule |

---

## 🧪 Testing avec cURL

```bash
# Login
curl -X POST https://api.sakado.digital/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'

# Get current user
curl -X GET https://api.sakado.digital/api/users/me \
  -H "Authorization: Bearer YOUR_TOKEN"

# List users
curl -X GET "https://api.sakado.digital/api/users?page=1&limit=10" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## 📚 Ressources

- [GitHub Repo](https://github.com/SakadoDigital/Sakado)
- [Guide Déploiement](DEPLOYMENT.md)
- [Architecture](ARCHITECTURE.md)
- [Support](https://github.com/SakadoDigital/Sakado/discussions)

---

**Dernière mise à jour**: 2026-06-12

# Git Conventional Commits Configuration

## Format des commits

Utilisez le format suivant pour tous les commits:

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Types autorisés

- **feat**: Une nouvelle fonctionnalité
- **fix**: Une correction de bug
- **docs**: Changements de documentation
- **style**: Changements qui n'affectent pas le code (formatting, missing semicolons, etc)
- **refactor**: Refactorisation du code sans changement de fonctionnalité
- **perf**: Amélioration de performance
- **test**: Ajout ou modification de tests
- **chore**: Changements de build, dépendances, scripts, etc
- **ci**: Changements de configuration CI/CD

## Exemples

### Exemple 1: Nouvelle fonctionnalité
```
feat(auth): ajouter authentification JWT

Ajoute un système d'authentification JWT complèt avec:
- Login/Logout
- Token refresh
- Token expiration
- RBAC

Closes #123
```

### Exemple 2: Bug fix
```
fix(database): corriger connexion pooling

La connexion PostgreSQL se fermait prématurément.
Augmentation du timeout et amélioration de la gestion d'erreur.

Fixes #456
```

### Exemple 3: Documentation
```
docs: améliorer guide de déploiement

Ajout des étapes manquantes pour la configuration SSL.
Clarification des variables d'environnement requises.
```

### Exemple 4: Refactoring
```
refactor(api): simplifier validations

Extrait les validations communes dans un middleware
pour réduire la duplication de code.
```

## Portée (Scope)

Utilisez une portée qui décrit le sous-système affecté:

- auth
- api
- database
- ui
- config
- docker
- tests
- ci
- docs

## Corps du message

- Expliquez **pourquoi** pas juste le quoi
- Une ligne vide avant le corps
- Envelopper à 72 caractères
- Utilisez l'impératif: "ajouter" pas "ajouté" ou "ajoute"

## Pied de page

Incluez des références aux issues:

```
Closes #123
Fixes #456
Relates to #789
Breaking change: description
```

## Bonnes pratiques

✅ DO:
- Commits atomiques et focalisés
- Messages clairs et descriptifs
- Une feature = une branche
- Rebase avant PR

❌ DON'T:
- Mix multiple features dans un commit
- Messages vagues ("fix stuff", "update code")
- Commits directs sur main
- Push de commits non-testés

## Configuration Git

```bash
# Voir le dernier commit
git log -1

# Amender le dernier commit
git commit --amend

# Réécrire l'historique
git rebase -i HEAD~3
```

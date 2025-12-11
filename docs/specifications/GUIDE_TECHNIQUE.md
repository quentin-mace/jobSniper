# Guide Technique - JobSniper

## 1. Stack Technique

### Choix Technologiques MVP

| Composant | Technologie | Version | Justification |
|-----------|-------------|---------|---------------|
| **Backend** | Symfony | 8.0 | Framework PHP moderne et complet |
| **Langage** | PHP | 8.4+ | Déjà en place, rapide pour MVP |
| **ORM** | Doctrine | 3.x | Intégré à Symfony, productif |
| **Base de données** | PostgreSQL | 16+ | Robuste, performant, open source |
| **Serveur web** | FrankenPHP | latest | Moderne, performant, HTTP/3 |
| **Templating** | Twig | 3.x | Intégré Symfony, sûr |
| **Assets** | Asset Mapper | - | Pas de build JS, simplicité |
| **Interactivité** | HTMX | 2.x | SPA-like sans complexité JS |
| **JS léger** | Stimulus | 3.x | Composants riches ponctuels |
| **CSS** | Tailwind CSS | 3.x | Rapide à développer, moderne |
| **UI Components** | DaisyUI | 4.x | Composants Tailwind prêts à l'emploi |
| **Emails** | Symfony Mailer | - | Intégré, fiable |

### Évolution Future (V2.0)

**Microservices Rust** (optionnel si besoin de performance) :
- Service de statistiques lourdes
- Worker de traitement batch
- Justification : Performance, apprentissage Rust

**Décision :** Reporter après validation du besoin réel.

---

## 2. Architecture Simplifiée MVP

### 2.1 Schéma d'Architecture

```
┌─────────────────────────────────────────┐
│           Navigateur Web                │
│  ┌───────────────────────────────────┐  │
│  │ HTMX + Stimulus                   │  │
│  │ Tailwind CSS + DaisyUI            │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
                   │
                   │ HTTP/AJAX
                   ▼
┌─────────────────────────────────────────┐
│         FrankenPHP + Symfony 8          │
│  ┌───────────────────────────────────┐  │
│  │  Controllers                      │  │
│  │  ├─ DashboardController           │  │
│  │  ├─ JobApplicationController      │  │
│  │  └─ SecurityController            │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │  Services                         │  │
│  │  ├─ JobApplicationService         │  │
│  │  ├─ ReminderService               │  │
│  │  └─ StatisticsService             │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │  Entities & Repositories          │  │
│  │  ├─ User                          │  │
│  │  ├─ JobApplication                │  │
│  │  └─ UserPreference                │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
                   │
                   │ Doctrine ORM
                   ▼
┌─────────────────────────────────────────┐
│          PostgreSQL 16                  │
│  Tables: user, job_application,         │
│          user_preference                │
└─────────────────────────────────────────┘
```

### 2.2 Flux de Données

**Exemple : Changement de statut avec HTMX**

```
1. User clique sur select statut
   ↓
2. HTMX envoie POST /applications/{id}/status
   ↓
3. Symfony Controller traite
   ↓
4. Service met à jour BDD
   ↓
5. Twig rend le nouveau badge
   ↓
6. HTMX remplace uniquement le badge dans le DOM
```

**Avantage :** Expérience fluide sans rechargement, code simple côté serveur.

---

## 3. Modèle de Données

### 3.1 Schéma Entité-Relation

```
┌─────────────────┐           ┌──────────────────────┐
│      User       │           │   JobApplication     │
├─────────────────┤           ├──────────────────────┤
│ id (PK)         │───────────│ id (PK)              │
│ email           │     1:N   │ user_id (FK)         │
│ password        │           │ company_name         │
│ first_name      │           │ job_title            │
│ last_name       │           │ applied_at           │
│ created_at      │           │ status (enum)        │
│ updated_at      │           │ url                  │
└─────────────────┘           │ contact_email        │
        │                     │ contact_phone        │
        │ 1:1                 │ comment              │
        │                     │ is_archived          │
        ▼                     │ created_at           │
┌─────────────────┐           │ updated_at           │
│ UserPreference  │           └──────────────────────┘
├─────────────────┤
│ id (PK)         │
│ user_id (FK)    │
│ reminder_days   │
│ timezone        │
│ locale          │
└─────────────────┘
```

### 3.2 Entités Principales

#### User
- **Champs clés :** `email` (unique), `password` (hashé), `firstName`, `lastName`
- **Relations :** 
  - 1:N avec `JobApplication`
  - 1:1 avec `UserPreference`
- **Implémente :** `UserInterface`, `PasswordAuthenticatedUserInterface` (Symfony Security)

#### JobApplication
- **Champs obligatoires :** `companyName`, `jobTitle`, `appliedAt`, `status`
- **Champs optionnels :** `url`, `contactEmail`, `contactPhone`, `comment`
- **Soft delete :** `isArchived` (boolean)
- **Relations :** N:1 avec `User`

#### UserPreference
- **Champs :** `reminderDays` (integer, défaut: 7), `timezone`, `locale`
- **Relations :** 1:1 avec `User`
- **Création automatique :** À l'inscription de l'utilisateur

### 3.3 Enum des Statuts (MVP)

```php
enum JobApplicationStatus: string
{
    case DRAFT = 'draft';
    case SENT = 'sent';
    case NEEDS_FOLLOW_UP = 'needs_follow_up';
    case IN_PROGRESS = 'in_progress';
    case OFFER_RECEIVED = 'offer_received';
    case ACCEPTED = 'accepted';
    case REJECTED = 'rejected';
    
    public function label(): string
    {
        return match($this) {
            self::DRAFT => 'Brouillon',
            self::SENT => 'Envoyée',
            self::NEEDS_FOLLOW_UP => 'À relancer',
            self::IN_PROGRESS => 'En cours',
            self::OFFER_RECEIVED => 'Offre reçue',
            self::ACCEPTED => 'Acceptée',
            self::REJECTED => 'Refusée',
        };
    }
    
    public function color(): string
    {
        return match($this) {
            self::DRAFT => 'gray',
            self::SENT => 'blue',
            self::NEEDS_FOLLOW_UP => 'amber',
            self::IN_PROGRESS => 'purple',
            self::OFFER_RECEIVED => 'emerald',
            self::ACCEPTED => 'green',
            self::REJECTED => 'red',
        };
    }
}
```

---

## 4. Routes Principales

### 4.1 Routes Web

| Méthode | Route | Controller::Action | Description |
|---------|-------|-------------------|-------------|
| GET | `/` | Home::index | Landing page |
| GET/POST | `/register` | Security::register | Inscription |
| GET/POST | `/login` | Security::login | Connexion |
| GET | `/logout` | Security::logout | Déconnexion |
| GET | `/dashboard` | Dashboard::index | Tableau de bord |
| GET | `/applications` | JobApplication::index | Liste (tableau) |
| GET | `/applications/new` | JobApplication::new | Formulaire création |
| POST | `/applications` | JobApplication::create | Sauvegarde nouvelle |
| GET | `/applications/{id}` | JobApplication::show | Détail |
| GET | `/applications/{id}/edit` | JobApplication::edit | Formulaire édition |
| PUT | `/applications/{id}` | JobApplication::update | Sauvegarde modification |
| POST | `/applications/{id}/archive` | JobApplication::archive | Archiver |
| POST | `/applications/{id}/unarchive` | JobApplication::unarchive | Restaurer |
| POST | `/applications/{id}/status` | JobApplication::updateStatus | Changer statut (HTMX) |
| GET/POST | `/settings` | Settings::index | Paramètres utilisateur |

### 4.2 Interactions HTMX

**Exemple 1 : Changement de statut**
```html
<select 
  hx-post="/applications/123/status" 
  hx-target="#status-badge-123"
  hx-swap="outerHTML"
  name="status">
  <option value="sent">Envoyée</option>
  <option value="in_progress">En cours</option>
</select>
```

**Exemple 2 : Filtres sans rechargement**
```html
<form 
  hx-get="/applications" 
  hx-target="#applications-list"
  hx-trigger="change"
  hx-push-url="true">
  <input type="search" name="q" placeholder="Rechercher...">
  <select name="status">...</select>
</form>
```

---

## 5. Structure des Répertoires

```
src/
├── Controller/
│   ├── DashboardController.php       # Tableau de bord
│   ├── JobApplicationController.php  # CRUD candidatures
│   ├── SecurityController.php        # Auth (login, register)
│   └── SettingsController.php        # Paramètres utilisateur
│
├── Entity/
│   ├── User.php
│   ├── JobApplication.php
│   └── UserPreference.php
│
├── Repository/
│   ├── UserRepository.php
│   └── JobApplicationRepository.php
│
├── Form/
│   ├── RegistrationType.php
│   ├── JobApplicationType.php
│   └── UserPreferenceType.php
│
├── Service/
│   ├── JobApplicationService.php     # Logique métier candidatures
│   ├── ReminderService.php           # Calcul des rappels
│   ├── StatisticsService.php         # Calcul des statistiques
│   └── MailerService.php             # Envoi emails (V1.1)
│
├── Enum/
│   └── JobApplicationStatus.php
│
├── Security/
│   └── Voter/
│       └── JobApplicationVoter.php   # Contrôle d'accès
│
└── Command/
    └── ProcessRemindersCommand.php   # Commande cron

assets/
├── app.js                            # Point d'entrée JS
├── controllers/                      # Stimulus controllers
│   ├── status_controller.js
│   └── filter_controller.js
└── styles/
    └── app.css                       # Tailwind CSS

templates/
├── base.html.twig
├── dashboard/
│   └── index.html.twig
├── job_application/
│   ├── index.html.twig               # Liste
│   ├── show.html.twig                # Détail
│   ├── _form.html.twig               # Formulaire
│   └── _status_badge.html.twig       # Badge (pour HTMX swap)
├── security/
│   ├── login.html.twig
│   └── register.html.twig
└── settings/
    └── index.html.twig
```

---

## 6. Sécurité

### 6.1 Authentification
- **Composant :** Symfony Security Bundle
- **Hash mot de passe :** Bcrypt (auto)
- **Remember me :** Token 30 jours
- **Reset password :** SymfonyCasts ResetPasswordBundle (V1.1)

### 6.2 Autorisation
- **Voters :** Vérification propriété des ressources
- **Règle :** Un user ne voit que ses candidatures

```php
// Exemple dans Controller
#[IsGranted('ROLE_USER')]
public function show(JobApplication $application): Response
{
    $this->denyAccessUnlessGranted('view', $application);
    // ...
}

// Voter
class JobApplicationVoter extends Voter
{
    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();
        return $subject->getUser() === $user;
    }
}
```

### 6.3 Protection CSRF
- Automatique sur tous les formulaires Symfony
- Token CSRF validé côté serveur

### 6.4 Validation
- Annotations Symfony Validator sur les entités
- Validation côté serveur (never trust client)
- Sanitization des inputs (Twig échappe par défaut)

---

## 7. Jobs Asynchrones

### 7.1 Commande de Traitement des Rappels

**Fichier :** `src/Command/ProcessRemindersCommand.php`

**Rôle :**
- Parcourir les candidatures au statut "Envoyée"
- Calculer celles dépassant le délai configuré
- Mettre à jour leur statut vers "À relancer"

**Exécution :** Toutes les heures via cron

```bash
# Cron configuration
0 * * * * cd /var/www/jobsniper && php bin/console app:process-reminders
```

**Logique simplifiée :**
```php
// Pseudo-code
$applications = $repository->findBy(['status' => 'sent']);
foreach ($applications as $app) {
    $daysSince = now()->diffInDays($app->getAppliedAt());
    $reminderDays = $app->getUser()->getPreference()->getReminderDays();
    
    if ($daysSince >= $reminderDays) {
        $app->setStatus('needs_follow_up');
        $entityManager->flush();
    }
}
```

### 7.2 Envoi d'Emails (V1.1)

**Composant :** Symfony Mailer + Twig templates
**Configuration :** SMTP ou service tiers (SendGrid, Mailgun)
**File d'attente :** Symfony Messenger (si volume élevé)

---

## 8. Performance et Optimisation

### 8.1 Indexation Base de Données

```sql
-- Index sur colonnes fréquemment requêtées
CREATE INDEX idx_job_application_user_status 
ON job_application(user_id, status);

CREATE INDEX idx_job_application_applied_at 
ON job_application(applied_at);
```

### 8.2 Requêtes Optimisées

- Doctrine QueryBuilder avec jointures explicites
- Pagination native Doctrine
- Lazy loading désactivé si non nécessaire
- Méthodes de repository optimisées

### 8.3 Cache (V1.2)

- HTTP Cache Symfony pour pages statiques
- Cache applicatif pour statistiques (Symfony Cache)
- OPcache PHP activé en production

### 8.4 Assets

- Asset Mapper compile et minifie en production
- CDN pour Tailwind CSS + DaisyUI (MVP) puis build local (V1.1)
- Images optimisées et lazy loading

---

## 9. Environnement de Développement

### 9.1 Docker Compose

```yaml
services:
  php:
    image: dunglas/frankenphp
    volumes:
      - .:/app
    ports:
      - "8080:80"
      - "443:443"
    environment:
      - DATABASE_URL=postgresql://app:app@database:5432/jobsniper
    depends_on:
      - database

  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: jobsniper
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  mailer:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Interface web

volumes:
  db_data:
```

### 9.2 Commandes Utiles

```bash
# Démarrage
docker compose up -d

# Installation dépendances
docker compose exec php composer install

# Création BDD
docker compose exec php bin/console doctrine:database:create
docker compose exec php bin/console doctrine:migrations:migrate

# Tests
docker compose exec php bin/phpunit

# Qualité code
docker compose exec php vendor/bin/php-cs-fixer fix
docker compose exec php bin/console lint:twig templates/
```

---

## 10. Tests

### 10.1 Structure

```
tests/
├── Unit/
│   ├── Service/
│   │   ├── ReminderServiceTest.php
│   │   └── StatisticsServiceTest.php
│   └── Enum/
│       └── JobApplicationStatusTest.php
│
├── Functional/
│   ├── Controller/
│   │   └── JobApplicationControllerTest.php
│   └── Command/
│       └── ProcessRemindersCommandTest.php
│
└── Integration/
    └── Repository/
        └── JobApplicationRepositoryTest.php
```

### 10.2 Couverture Minimale MVP

- **Controllers :** Tests fonctionnels des routes principales
- **Services :** Tests unitaires de la logique métier
- **Commands :** Tests de la commande de rappels
- **Objectif :** 70% de couverture minimum

---

## 11. Déploiement

### 11.1 Checklist Production

- [ ] Variables d'environnement configurées
- [ ] APP_ENV=prod
- [ ] Secrets Symfony configurés
- [ ] Base de données créée et migrée
- [ ] Assets compilés (asset-map:compile)
- [ ] Cron configuré pour rappels
- [ ] Logs rotatifs configurés
- [ ] HTTPS activé
- [ ] Backups automatiques BDD

### 11.2 Hébergement Suggéré

**Option 1 : VPS classique** (DigitalOcean, Hetzner)
- Docker + Docker Compose
- Reverse proxy Caddy
- PostgreSQL managé ou conteneurisé

**Option 2 : Platform.sh / Symfony Cloud**
- Déploiement Git automatique
- BDD et services managés
- Scaling automatique

---

## 12. Migration Future vers Rust (V2.0)

**Si nécessaire, architecture évoluée :**

```
Navigateur → Caddy Reverse Proxy
                ├─ /app/* → Symfony (UI, Auth, CRUD)
                └─ /api/stats/* → Rust (Calculs lourds)
                        ↓
                   PostgreSQL (partagé)
```

**Justification migration :**
- Statistiques complexes > 10k candidatures
- Temps de calcul PHP > 2 secondes
- Apprentissage Rust en contexte réel

**Pour l'instant : PHP suffit amplement.**

---

## 13. Ressources et Documentation

### Documentation Officielle
- [Symfony 8.0](https://symfony.com/doc/current/index.html)
- [HTMX](https://htmx.org/docs/)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [DaisyUI](https://daisyui.com/)
- [PostgreSQL](https://www.postgresql.org/docs/)

### Tutoriels Symfony
- [SymfonyCasts](https://symfonycasts.com/)
- [Symfony Best Practices](https://symfony.com/doc/current/best_practices.html)

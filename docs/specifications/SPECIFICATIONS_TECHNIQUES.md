# Spécifications Techniques - JobSniper

## 1. Stack Technique

### 1.1 Backend

| Composant | Technologie | Version |
|-----------|-------------|---------|
| Langage | PHP | >= 8.4 |
| Framework | Symfony | 8.0.* |
| ORM | Doctrine | 3.x |
| Base de données | PostgreSQL | 16+ |
| Serveur web | FrankenPHP | - |

### 1.2 Frontend

| Composant | Technologie | Justification |
|-----------|-------------|---------------|
| Templating | Twig | Intégré à Symfony |
| Assets | Asset Mapper | Moderne, sans Node.js |
| JavaScript | Stimulus | Réactivité légère |
| Navigation | Turbo | SPA-like sans JS custom |
| CSS | Tailwind CSS | Utilitaire, rapide à développer |

### 1.3 Infrastructure

| Composant | Technologie |
|-----------|-------------|
| Conteneurisation | Docker + Docker Compose |
| CI/CD | GitHub Actions (à définir) |
| Hébergement | À définir (VPS, Platform.sh, etc.) |

---

## 2. Architecture

### 2.1 Architecture Globale

```
┌─────────────────────────────────────────────────────────┐
│                      Navigateur                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Turbo Drive + Stimulus Controllers              │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    Symfony 8.0                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Controllers │  │   Services   │  │  Repositories │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │    Forms     │  │   Entities   │  │  Messenger   │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    PostgreSQL                            │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Structure des Répertoires

```
src/
├── Controller/
│   ├── Admin/                    # Controllers admin (futur)
│   ├── JobApplicationController.php # CRUD candidatures
│   ├── DashboardController.php   # Tableau de bord
│   ├── ReminderController.php    # Gestion rappels
│   └── SecurityController.php    # Auth (login, register, etc.)
├── Entity/
│   ├── User.php
│   ├── JobApplication.php           # Candidature
│   ├── JobApplicationStatus.php     # Historique des statuts
│   ├── Reminder.php              # Rappels
│   └── UserPreference.php        # Préférences utilisateur
├── Repository/
│   ├── UserRepository.php
│   ├── JobApplicationRepository.php
│   └── ReminderRepository.php
├── Form/
│   ├── JobApplicationType.php
│   ├── ReminderType.php
│   ├── RegistrationType.php
│   └── UserPreferenceType.php
├── Service/
│   ├── JobApplicationService.php    # Logique métier candidatures
│   ├── FollowUpService.php       # Gestion des relances automatiques
│   ├── StatisticsService.php     # Calcul des statistiques
│   └── MailerService.php         # Envoi emails de relance
├── Command/
│   ├── ProcessFollowUpRemindersCommand.php  # Cron pour relances
│   └── UpdateStaleJobApplicationsCommand.php   # Mise à jour statuts inactifs
├── EventSubscriber/
│   └── JobApplicationStatusSubscriber.php  # Écoute changements statut
├── Twig/
│   └── Components/               # Twig Components
│       ├── JobApplicationCard.php
│       ├── StatusBadge.php
│       └── ReminderAlert.php
└── Enum/
    ├── JobApplicationStatusEnum.php
    └── ReminderTypeEnum.php
```

---

## 3. Modèle de Données

### 3.1 Diagramme Entité-Relation

```
┌─────────────┐       ┌──────────────────┐       ┌─────────────────┐
│    User     │       │  JobApplication  │       │    Reminder     │
├─────────────┤       ├──────────────────┤       ├─────────────────┤
│ id (PK)     │──────<│ id (PK)          │>──────│ id (PK)         │
│ email       │       │ user_id (FK)     │       │ job_application_id  │
│ password    │       │ company_name     │       │ type            │
│ first_name  │       │ job_title        │       │ scheduled_at    │
│ last_name   │       │ applied_at       │       │ is_done         │
│ created_at  │       │ status           │       │ notes           │
│ updated_at  │       │ url              │       │ created_at      │
│             │       │ contact_email    │       └─────────────────┘
│             │       │ contact_phone    │
│             │       │ comment          │       ┌──────────────────┐
│             │       │ is_archived      │       │JobApplicationStatus │
│             │       │ created_at       │       ├──────────────────┤
│             │       │ updated_at       │       │ id (PK)          │
│             │       └──────────────────┘       │ job_application_id   │
│             │                                  │ status           │
│             │                                  │ notes            │
│             │                                  │ changed_at       │
│             │                                  └──────────────────┘
│             │
│             │                                  ┌─────────────────┐
│             │                                  │ UserPreference  │
│             │                                  ├─────────────────┤
│             │                                  │ id (PK)         │
│             │                                  │ user_id (FK)    │
└─────────────┴─────────────────────────────────>│ reminder_days   │
                                                 │ auto_archive    │
                                                 │ timezone        │
                                                 │ locale          │
                                                 └─────────────────┘
```

### 3.2 Définition des Entités

#### User
```php
#[ORM\Entity(repositoryClass: UserRepository::class)]
#[ORM\Table(name: '`user`')]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180, unique: true)]
    private ?string $email = null;

    #[ORM\Column]
    private array $roles = [];

    #[ORM\Column]
    private ?string $password = null;

    #[ORM\Column(length: 100)]
    private ?string $firstName = null;

    #[ORM\Column(length: 100)]
    private ?string $lastName = null;

    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    #[ORM\OneToMany(targetEntity: JobApplication::class, mappedBy: 'user', orphanRemoval: true)]
    private Collection $jobApplications;

    #[ORM\OneToOne(targetEntity: UserPreference::class, mappedBy: 'user', cascade: ['persist', 'remove'])]
    private ?UserPreference $preference = null;
}
```

#### Application (Candidature)
```php
#[ORM\Entity(repositoryClass: JobApplicationRepository::class)]
#[ORM\HasLifecycleCallbacks]
class JobApplication
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\ManyToOne(inversedBy: 'applications')]
    #[ORM\JoinColumn(nullable: false)]
    private ?User $user = null;

    #[ORM\Column(length: 255)]
    private ?string $companyName = null;

    #[ORM\Column(length: 255)]
    private ?string $jobTitle = null;

    #[ORM\Column(type: Types::DATE_MUTABLE)]
    private ?\DateTimeInterface $appliedAt = null;

    #[ORM\Column(type: Types::STRING, enumType: JobApplicationStatusEnum::class)]
    private ApplicationStatusEnum $status = JobApplicationStatusEnum::DRAFT;

    #[ORM\Column(length: 500, nullable: true)]
    private ?string $url = null;

    #[ORM\Column(length: 180, nullable: true)]
    private ?string $contactEmail = null;

    #[ORM\Column(length: 30, nullable: true)]
    private ?string $contactPhone = null;

    #[ORM\Column(type: Types::TEXT, nullable: true)]
    private ?string $comment = null;

    #[ORM\Column]
    private bool $isArchived = false;

    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    #[ORM\OneToMany(targetEntity: JobApplicationStatus::class, mappedBy: 'application', cascade: ['persist'], orphanRemoval: true)]
    #[ORM\OrderBy(['changedAt' => 'DESC'])]
    private Collection $statusHistory;

    #[ORM\OneToMany(targetEntity: Reminder::class, mappedBy: 'application', orphanRemoval: true)]
    private Collection $reminders;
}
```

### 3.3 Enums

```php
enum JobApplicationStatusEnum: string
{
    case DRAFT = 'draft';
    case SENT = 'sent';
    case NEEDS_FOLLOW_UP = 'needs_follow_up';   // À relancer (pas d'email recruteur)
    case FOLLOWED_UP = 'followed_up';           // Relancée (email envoyé auto)
    case VIEWED = 'viewed';
    case IN_PROGRESS = 'in_progress';
    case INTERVIEW_SCHEDULED = 'interview_scheduled';
    case INTERVIEW_DONE = 'interview_done';
    case OFFER_RECEIVED = 'offer_received';
    case ACCEPTED = 'accepted';
    case REJECTED_BY_ME = 'rejected_by_me';
    case REJECTED_BY_COMPANY = 'rejected_by_company';
    case NO_RESPONSE = 'no_response';
    case ARCHIVED = 'archived';

    public function label(): string
    {
        return match($this) {
            self::DRAFT => 'Brouillon',
            self::SENT => 'Envoyée',
            self::NEEDS_FOLLOW_UP => 'À relancer',
            self::FOLLOWED_UP => 'Relancée',
            self::VIEWED => 'Vue',
            self::IN_PROGRESS => 'En cours',
            self::INTERVIEW_SCHEDULED => 'Entretien planifié',
            self::INTERVIEW_DONE => 'Entretien passé',
            self::OFFER_RECEIVED => 'Offre reçue',
            self::ACCEPTED => 'Acceptée',
            self::REJECTED_BY_ME => 'Refusée (par moi)',
            self::REJECTED_BY_COMPANY => 'Refusée (entreprise)',
            self::NO_RESPONSE => 'Sans réponse',
            self::ARCHIVED => 'Archivée',
        };
    }

    public function color(): string
    {
        return match($this) {
            self::DRAFT => 'gray',
            self::SENT => 'blue',
            self::NEEDS_FOLLOW_UP => 'amber',
            self::FOLLOWED_UP => 'cyan',
            self::VIEWED => 'indigo',
            self::IN_PROGRESS => 'yellow',
            self::INTERVIEW_SCHEDULED => 'purple',
            self::INTERVIEW_DONE => 'pink',
            self::OFFER_RECEIVED => 'emerald',
            self::ACCEPTED => 'green',
            self::REJECTED_BY_ME => 'orange',
            self::REJECTED_BY_COMPANY => 'red',
            self::NO_RESPONSE => 'slate',
            self::ARCHIVED => 'zinc',
        };
    }
}

enum ReminderTypeEnum: string
{
    case FOLLOW_UP = 'follow_up';      // Relance uniquement
}
```

---

## 4. Routes et API

### 4.1 Routes Web

| Route | Méthode | Controller | Description |
|-------|---------|------------|-------------|
| `/` | GET | HomeController::index | Page d'accueil |
| `/register` | GET/POST | SecurityController::register | Inscription |
| `/login` | GET/POST | SecurityController::login | Connexion |
| `/logout` | GET | SecurityController::logout | Déconnexion |
| `/dashboard` | GET | DashboardController::index | Tableau de bord |
| `/applications` | GET | ApplicationController::index | Liste candidatures |
| `/applications/new` | GET/POST | ApplicationController::new | Nouvelle candidature |
| `/applications/{id}` | GET | ApplicationController::show | Détail candidature |
| `/applications/{id}/edit` | GET/POST | ApplicationController::edit | Modifier candidature |
| `/applications/{id}/archive` | POST | ApplicationController::archive | Archiver candidature (soft delete) |
| `/applications/{id}/unarchive` | POST | ApplicationController::unarchive | Désarchiver candidature |
| `/applications/{id}/status` | POST | ApplicationController::changeStatus | Changer statut |
| `/reminders` | GET | ReminderController::index | Liste candidatures à relancer |
| `/settings` | GET/POST | SettingsController::index | Paramètres |

### 4.2 Turbo Frames

Pour une expérience SPA-like, certaines parties seront chargées via Turbo Frames :

- `application-list` : Liste des candidatures (filtrable sans rechargement)
- `application-status` : Badge de statut (changement rapide)
- `reminders-panel` : Panneau des rappels
- `stats-widget` : Widgets de statistiques

---

## 5. Sécurité

### 5.1 Authentification

- Utilisation du Security Bundle de Symfony
- Hash des mots de passe avec bcrypt/argon2
- Protection CSRF sur tous les formulaires
- Rate limiting sur login (à implémenter)

### 5.2 Autorisation

- Voters Symfony pour vérifier l'accès aux ressources
- Un utilisateur ne peut voir/modifier que ses propres candidatures

```php
#[IsGranted('ROLE_USER')]
#[IsGranted('APPLICATION_VIEW', subject: 'application')]
public function show(Application $application): Response
```

### 5.3 Validation

- Validation côté serveur avec Symfony Validator
- Contraintes sur les entités (Assert attributes)
- Sanitization des inputs

---

## 6. Commandes Console

### 6.1 Traitement des Rappels de Relance

```bash
# Exécuter toutes les heures via cron
php bin/console app:process-follow-up-reminders
```

Cette commande :
- Récupère les candidatures au statut "Envoyée" dont le délai de relance est dépassé
- **Si l'email du recruteur est renseigné** :
  - Envoie un email de relance automatique au recruteur
  - Passe le statut de la candidature à "Relancée" (FOLLOWED_UP)
  - Historise l'envoi de l'email
- **Si l'email du recruteur n'est pas renseigné** :
  - Passe le statut de la candidature à "À relancer" (NEEDS_FOLLOW_UP)
  - L'utilisateur devra effectuer la relance manuellement

### 6.2 Mise à jour des statuts

```bash
# Exécuter quotidiennement
php bin/console app:update-stale-applications
```

Cette commande :
- Identifie les candidatures sans activité depuis X jours
- Les passe en statut "Sans réponse"

---

## 7. Tests

### 7.1 Structure des Tests

```
tests/
├── Unit/
│   ├── Entity/
│   ├── Service/
│   └── Enum/
├── Integration/
│   ├── Repository/
│   └── Service/
└── Functional/
    ├── Controller/
    └── Command/
```

### 7.2 Outils de Test

- PHPUnit 12.x (déjà installé)
- Symfony BrowserKit pour tests fonctionnels
- Fixtures avec Foundry ou DoctrineFixturesBundle

---

## 8. Environnement de Développement

### 8.1 Docker Compose

Le fichier `compose.yaml` existant sera complété pour inclure :

```yaml
services:
  php:
    image: dunglas/frankenphp
    ports:
      - "8080:80"
      - "443:443"
    volumes:
      - .:/app
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
      - "1025:1025"
      - "8025:8025"

volumes:
  db_data:
```

### 8.2 Commandes de Développement

```bash
# Démarrer l'environnement
docker compose up -d

# Créer la base de données
php bin/console doctrine:database:create
php bin/console doctrine:migrations:migrate

# Charger les fixtures (développement)
php bin/console doctrine:fixtures:load

# Lancer les tests
php bin/phpunit

# Vérifier le code
php bin/console lint:twig templates/
php bin/console lint:yaml config/
```

---

## 9. Performance

### 9.1 Optimisations Prévues

- Cache HTTP avec Symfony Cache
- Index sur les colonnes fréquemment requêtées (user_id, status, applied_at)
- Pagination des listes
- Lazy loading des relations Doctrine

### 9.2 Monitoring

- Symfony Profiler en développement
- Logs structurés avec Monolog

---

## 10. Prochaines Étapes

1. **Setup initial**
   - Configuration Docker complète
   - Installation Tailwind CSS via Asset Mapper
   - Configuration base de données

2. **Entités et migrations**
   - Création des entités Doctrine
   - Génération des migrations
   - Configuration des fixtures

3. **Authentification**
   - Mise en place Security Bundle
   - Formulaires inscription/connexion
   - Remember me token

4. **CRUD Candidatures**
   - Controller, Forms, Templates
   - Filtres et recherche
   - Changement de statut

5. **Dashboard et statistiques**
   - Vue d'ensemble
   - Widgets statistiques
   - Graphiques (Chart.js via Stimulus)

6. **Système de relances automatiques**
   - Service de relance (FollowUpService)
   - Commande console de traitement des relances
   - Envoi d'emails automatiques si email recruteur présent
   - Changement de statut automatique si pas d'email

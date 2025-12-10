# Spécifications Fonctionnelles - JobSniper

## 1. Présentation du Projet

**JobSniper** est une application web de suivi de candidatures destinée aux chercheurs d'emploi. Elle permet de centraliser, organiser et suivre l'ensemble des candidatures effectuées durant une recherche d'emploi.

### 1.1 Objectifs

- Centraliser toutes les candidatures en un seul endroit
- Suivre l'avancement de chaque candidature
- Ne jamais oublier de relancer grâce aux rappels configurables
- Avoir une vue d'ensemble claire de sa recherche d'emploi
- Analyser ses statistiques de candidature

---

## 2. Utilisateurs Cibles

- Chercheurs d'emploi actifs
- Personnes en reconversion professionnelle
- Étudiants en recherche de stage/alternance
- Freelances à la recherche de missions

---

## 3. Fonctionnalités

### 3.1 Gestion des Utilisateurs

#### F1 - Inscription
- Création de compte avec email et mot de passe
- Validation de l'email (optionnel en V1)
- Informations du profil : nom, prénom, email

#### F2 - Connexion / Déconnexion
- Authentification par email/mot de passe
- Option "Se souvenir de moi"
- Réinitialisation du mot de passe

#### F3 - Gestion du Profil
- Modification des informations personnelles
- Changement de mot de passe
- Suppression du compte

---

### 3.2 Gestion des Candidatures

#### F4 - Création d'une Candidature
Champs obligatoires :
- Nom de l'entreprise
- Intitulé du poste
- Date de candidature

Champs optionnels :
- URL (site du recruteur / offre d'emploi)
- Email du contact
- Téléphone du contact
- Commentaire

#### F5 - Modification d'une Candidature
- Modification de tous les champs
- Historique des modifications (optionnel en V1)

#### F6 - Archivage d'une Candidature
- Pas de suppression définitive des candidatures (soft delete uniquement)
- Archivage avec confirmation : la candidature passe au statut "Archivée"
- Les candidatures archivées ne sont pas affichées par défaut dans les listes
- Possibilité de filtrer pour voir les candidatures archivées
- Possibilité de désarchiver une candidature (restauration)

#### F7 - Liste des Candidatures
- Affichage sous forme de tableau ou kanban
- Par défaut, les candidatures archivées ne sont pas affichées
- Tri par date, entreprise, statut
- Filtres par statut, période, type de contrat
- Filtre pour inclure/afficher les candidatures archivées
- Recherche textuelle

---

### 3.3 Suivi des Statuts

#### F8 - Gestion des Statuts
Statuts prédéfinis :
1. **Brouillon** - Candidature en préparation
2. **Envoyée** - Candidature transmise
3. **À relancer** - Rappel déclenché, relance manuelle nécessaire (pas d'email recruteur)
4. **Relancée** - Email de relance envoyé automatiquement au recruteur
5. **Vue** - L'entreprise a consulté la candidature
6. **En cours** - Processus de recrutement actif
7. **Entretien planifié** - Un entretien est programmé
8. **Entretien passé** - En attente de retour après entretien
9. **Offre reçue** - Proposition d'embauche reçue
10. **Acceptée** - Offre acceptée
11. **Refusée (par moi)** - J'ai décliné l'offre
12. **Refusée (par l'entreprise)** - Candidature rejetée
13. **Sans réponse** - Aucun retour après X jours
14. **Archivée** - Candidature classée

#### F9 - Changement de Statut
- Changement rapide depuis la liste
- Historique des changements de statut avec date
- Notes associées au changement de statut

---

### 3.4 Système de Rappels

#### F10 - Rappels de Relance
- Rappel automatique configurable X jours après l'envoi d'une candidature sans changement de statut
- Fréquence par défaut configurable dans les préférences utilisateur (défaut : 7 jours)
- Possibilité de personnaliser la fréquence par candidature

#### F11 - Comportement des Rappels
Deux comportements selon la présence d'une adresse email du recruteur :

**Si l'email du recruteur est renseigné :**
- Envoi automatique d'un email de relance au recruteur
- L'email utilise un modèle prédéfini personnalisable
- Le statut de la candidature passe à "Relancée"
- Historisation de l'envoi (date, destinataire)

**Si l'email du recruteur n'est pas renseigné :**
- Changement automatique du statut de la candidature vers "À relancer"
- L'utilisateur voit la candidature mise en avant dans son dashboard
- L'utilisateur doit effectuer la relance manuellement puis mettre à jour le statut

#### F12 - Affichage des Rappels
- Badge indiquant le nombre de candidatures à relancer sur le dashboard
- Liste des candidatures nécessitant une action dans une section dédiée

---

### 3.5 Tableau de Bord

#### F13 - Dashboard Principal
- Statistiques globales :
  - Nombre total de candidatures
  - Répartition par statut
  - Taux de réponse
  - Taux de conversion (candidatures → entretiens)
- Candidatures récentes
- Prochains rappels
- Graphique d'évolution des candidatures dans le temps

---

### 3.6 Paramètres

#### F14 - Préférences Utilisateur
- Fréquence de rappel par défaut
- Préférences de notification
- Fuseau horaire
- Langue de l'interface (FR par défaut)

#### F15 - Personnalisation des Statuts (V2)
- Ajout de statuts personnalisés
- Modification des couleurs des statuts
- Réorganisation de l'ordre des statuts

---

## 4. Règles de Gestion

### RG1 - Rappels Automatiques de Relance
- Un rappel est déclenché automatiquement X jours après l'envoi d'une candidature si aucun changement de statut n'intervient
- La durée X est configurable (défaut : 7 jours)
- **Si l'email du recruteur est renseigné** : un email de relance est envoyé automatiquement et le statut passe à "Relancée"
- **Si l'email du recruteur n'est pas renseigné** : le statut passe à "À relancer" pour signaler à l'utilisateur qu'une action manuelle est requise

### RG2 - Statut Sans Réponse
- Une candidature passe automatiquement en "Sans réponse" après Y jours sans activité
- La durée Y est configurable (défaut: 30 jours)

### RG3 - Archivage
- Les candidatures terminées (acceptée, refusée) peuvent être archivées
- Les candidatures archivées ne génèrent plus de rappels

### RG4 - Statistiques
- Les candidatures archivées sont incluses dans les statistiques
- Les brouillons ne sont pas comptés dans les statistiques principales

---

## 5. Interfaces Utilisateur

### 5.1 Pages Principales

1. **Page d'accueil / Landing** - Présentation de l'application
2. **Inscription / Connexion** - Formulaires d'authentification
3. **Dashboard** - Vue d'ensemble avec statistiques et raccourcis
4. **Liste des candidatures** - Tableau/Kanban des candidatures
5. **Détail candidature** - Fiche complète d'une candidature
6. **Formulaire candidature** - Création/édition de candidature
7. **Paramètres** - Configuration du compte

### 5.2 Composants Réutilisables

- Carte de candidature (pour vue kanban)
- Ligne de candidature (pour vue tableau)
- Badge de statut (coloré selon le statut)
- Indicateur de rappel
- Formulaire de changement de statut rapide

---

## 6. Priorités de Développement

### MVP (Version 1.0)
- [ ] Authentification (inscription, connexion, déconnexion)
- [ ] CRUD candidatures
- [ ] Gestion des statuts
- [ ] Liste des candidatures avec filtres
- [ ] Dashboard basique
- [ ] Rappels simples (affichage dans l'app)

### Version 1.1
- [ ] Notifications email
- [ ] Vue Kanban
- [ ] Statistiques avancées

### Version 2.0
- [ ] Personnalisation des statuts
- [ ] Import/Export des données
- [ ] Mode sombre
- [ ] Application mobile (PWA)

---

## 7. Glossaire

| Terme | Définition |
|-------|------------|
| Candidature | Ensemble des informations relatives à une postulation à une offre d'emploi |
| Rappel | Notification programmée pour inciter à une action (relance, etc.) |
| Statut | État actuel d'une candidature dans le processus de recrutement |
| Relance | Action de recontacter une entreprise pour avoir des nouvelles |

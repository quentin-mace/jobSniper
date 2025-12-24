# Prompts Maquettes - JobSniper

---

## 🎯 Context Global

Crée les maquettes pour JobSniper, une application web de suivi de candidatures. Le design doit être moderne et épuré, inspiré de Notion et Linear, avec une interface simple et rapide. L'application utilise 7 statuts représentés par des badges colorés : Brouillon en gris, Envoyée en bleu, À relancer en orange, Relancé en jaune, Offre reçue en vert clair, Acceptée en vert foncé et Refusée en rouge. La navigation principale comprend les sections Dashboard, Candidatures, Rappels et Paramètres.

---

## 📊 Prompt 1 : Dashboard

Crée la page d'accueil qui donne une vue d'ensemble de l'activité. En haut, affiche trois cartes de statistiques présentant le nombre total de candidatures, les candidatures à relancer avec un badge urgent, et le taux de réponse. Intègre ensuite un graphique simple montrant la répartition des candidatures par statut. Plus bas, affiche une liste des candidatures urgentes nécessitant une action immédiate sous forme de tableau épuré. Place un bouton CTA bien visible pour créer une nouvelle candidature. Termine avec une section activités récentes montrant les dernières modifications. Le design doit utiliser des cards pour les stats et rester responsive sur mobile.

---

## 📋 Prompt 2 : Page Candidatures

Crée la page principale qui liste toutes les candidatures. L'en-tête affiche le titre avec le compteur total et un bouton pour créer une nouvelle candidature. Juste en dessous, intègre une barre de filtres horizontale avec une recherche par entreprise ou poste, un filtre de statut en multi-sélection affichant le nombre de candidatures par catégorie, un filtre de date proposant aujourd'hui, cette semaine, ce mois ou une période personnalisée, et un menu de tri par date récente, alphabétique ou priorité.

Le tableau principal comporte sept colonnes : une checkbox pour la sélection multiple, l'entreprise en gras et cliquable, le poste, le statut sous forme de badge coloré cliquable pour modification rapide, la date d'envoi, la prochaine action avec un badge URGENT si la date est dépassée, et un menu kebab avec les actions voir, modifier, archiver et supprimer. Les lignes s'illuminent au survol et les candidatures urgentes ont une bordure colorée.

Inclus un état vide avec le message "Aucune candidature" et un CTA invitant à créer la première. Sur mobile, transforme le tableau en cartes verticales avec les filtres accessibles via un drawer et un bouton flottant pour ajouter. Affiche environ dix candidatures variées avec différents statuts et quelques rappels urgents pour le réalisme.

---

## 👤 Prompt 3 : Page Compte

Crée la page de gestion du compte utilisateur organisée en sections distinctes. En haut, affiche le profil avec un avatar circulaire modifiable au survol, le nom complet, l'email, la date d'inscription et un bouton pour modifier ces informations. 

La section suivante présente les préférences de notifications avec des toggles pour activer ou désactiver les rappels par email, le résumé hebdomadaire des statistiques, et les alertes de réponse d'entreprise. Chaque toggle inclut une courte description.

Ensuite, affiche les paramètres de candidature permettant de définir la durée par défaut avant relance, le statut par défaut à la création, et l'archivage automatique des candidatures refusées. Ajoute une section statistiques personnelles montrant le total de candidatures, le taux de réponse moyen, la candidature la plus ancienne et le temps moyen de réponse, présentés en cartes avec icônes.

En bas, place les actions du compte avec les boutons pour exporter les données, accéder à l'aide, se déconnecter et supprimer le compte en rouge avec confirmation. Le design utilise une colonne centrée avec largeur maximale confortable, des sections bien séparées et reste responsive sur mobile.

---

## 📝 Pages Suivantes

- [ ] Formulaire ajout/modification candidature
- [ ] Détail candidature
- [ ] Page rappels
- [ ] Login/Register

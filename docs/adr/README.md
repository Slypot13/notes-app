# ADR-0001 : Stockage des notes

- **Date :** 2026-11-13
- **Statut :** accepté

## Contexte

L'API doit enregistrer les notes entre les redémarrages. Le projet est une petite application pédagogique avec une API Express, sans besoin de gérer beaucoup de données ou plusieurs utilisateurs en même temps.

## Décision

Les notes sont stockées dans le fichier `api/notes.json`. L'API lit le fichier pour récupérer les notes et le réécrit après une création ou une suppression.

## Options étudiées

### Option 1 : fichier JSON (choisie)

- **Positif :** simple à mettre en place, aucun serveur ni dépendance supplémentaire, facile à lire.
- **Négatif :** chaque modification réécrit tout le fichier ; les accès simultanés et les gros volumes sont mal gérés.

### Option 2 : base de données SQLite

- **Positif :** recherches et modifications plus fiables, mieux adaptée à davantage de données.
- **Négatif :** demande une dépendance et une configuration supplémentaires pour ce petit projet.

Le fichier JSON est choisi car il répond aux besoins actuels du projet et reste le plus simple à utiliser.

## Conséquences

- Le projet reste facile à lancer et à maintenir.
- Le fichier JSON convient à un petit volume de notes.
- Chaque modification réécrit le fichier entier ; ce stockage n'est donc pas adapté à une forte utilisation simultanée ou à un grand volume de données.
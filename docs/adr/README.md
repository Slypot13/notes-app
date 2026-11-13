# ADR-0001 : Stockage des notes

- **Date :** 2026-11-13
- **Statut :** accepté

## Contexte

L'API doit enregistrer les notes entre les redémarrages. Le projet est une petite application pédagogique avec une API Express, sans besoin de gérer beaucoup de données ou plusieurs utilisateurs en même temps.

## Décision

Les notes sont stockées dans le fichier `api/notes.json`. L'API lit le fichier pour récupérer les notes et le réécrit après une création ou une suppression.

## Justification

Ce choix est simple à comprendre et à mettre en place. Il ne demande ni serveur de base de données ni dépendance supplémentaire. Le fichier est facile à consulter et à sauvegarder.

Une base de données comme SQLite serait plus adaptée si le projet devait grandir, gérer des accès simultanés ou effectuer des recherches plus complexes.

## Conséquences

- Le projet reste facile à lancer et à maintenir.
- Le fichier JSON convient à un petit volume de notes.
- Chaque modification réécrit le fichier entier ; ce stockage n'est donc pas adapté à une forte utilisation simultanée ou à un grand volume de données.
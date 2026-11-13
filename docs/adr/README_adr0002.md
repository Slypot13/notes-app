# ADR-0002 — Stratégie de pagination de `GET /notes`

- **Statut :** accepté
- **Date :** à compléter
- **Contexte :** l'API expose une liste de notes et le front doit pouvoir la parcourir sans charger toutes les notes à terme.

## Contexte et options étudiées

La route `GET /notes` peut fournir les notes de plusieurs façons :

1. **Pagination par numéro de page** : le client transmet un numéro de page et l'API renvoie une tranche de résultats.
2. **Pagination par curseur** : le client transmet un curseur associé au dernier élément reçu. Cette approche est adaptée aux grands volumes et aux listes qui changent souvent, mais demande une API et un front plus complexes.
3. **Pas de pagination côté API** : l'API renvoie toutes les notes. C'est simple pour le front au début, mais le volume de données et le temps de chargement augmentent avec le nombre de notes.

## Décision

Nous retenons une **pagination par numéro de page**, avec des pages de **10 notes**. Le paramètre `page` est un entier commençant à zéro : `page=0` désigne la première page, `page=1` la deuxième, etc. Si le paramètre est absent, l'API doit renvoyer la première page.

Cette solution est suffisante pour le volume actuel et permet au front d'implémenter simplement des commandes « précédent » et « suivant ». Le curseur pourra être réévalué si le volume ou la fréquence de modification des notes rend la pagination par pages inadaptée. Renvoyer toutes les notes n'est pas retenu afin d'éviter des réponses non bornées.

## Contrat API

```http
GET /notes?page=0
```

La réponse reste un tableau JSON contenant au maximum 10 notes. Le contrat ne fournit pas de métadonnées sur le nombre total de notes ni sur l'existence d'une page suivante. Une page sans résultat est donc représentée par un tableau vide.

Le paramètre `page` doit être un entier positif ou nul. L'API doit utiliser `0` par défaut et rejeter ou traiter explicitement les valeurs invalides ; elle ne doit pas les convertir silencieusement en une autre page.

## Implications pour le front

- Le front conserve le numéro de page courant, initialisé à `0`, et le passe à chaque requête (`/notes?page=${page}`).
- « Suivant » incrémente le numéro de page ; « précédent » le décrémente sans descendre sous `0`.
- Le front peut désactiver « précédent » sur la première page. Sans métadonnées, il ne connaît pas à l'avance la dernière page : il peut désactiver « suivant » lorsque la réponse contient moins de 10 notes, ou lorsque la page demandée est vide. Une page finale exactement pleine peut nécessiter un clic supplémentaire pour constater la fin.
- Lors d'un changement de page, le front remplace la liste affichée par la réponse de cette page et indique l'état de chargement et les erreurs réseau.
- Toute opération qui modifie l'ensemble des notes affichées (par exemple une recherche ou une création) doit ramener la pagination à `page=0` ou recharger la page courante de manière cohérente.

## État actuel et écart à corriger

L'implémentation actuelle dans `api/server.js` découpe bien la liste par tranches de 10, mais son calcul n'est pas conforme à cette décision : `Number(req.query.page) || 1` transforme `page=0` en `1`, et l'absence de paramètre sélectionne aussi la tranche d'index 10 à 19. Cela contredit l'exemple `page=0` du README principal. Il faut aligner l'API sur le contrat ci-dessus avant de considérer la pagination comme correctement prise en charge.

Le front actuel appelle `/notes` sans paramètre et ne propose pas de navigation entre pages. Tant que l'API n'est pas corrigée, cet appel peut afficher la deuxième page plutôt que la première.

## Conséquences

**Avantages**

- Mise en œuvre simple, avec une réponse de taille bornée.
- Contrat facile à utiliser et à tester depuis le front.

**Limites**

- Les insertions ou suppressions pendant la navigation peuvent décaler les éléments entre les pages.
- L'absence de métadonnées empêche de connaître précisément le nombre de pages et d'afficher une pagination numérotée complète.
- La navigation vers une page éloignée nécessite de connaître son numéro.
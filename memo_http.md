# Mémo HTTP pour concevoir une API REST

## Méthodes
| Méthode | Usage | Sûre | Idempotente | Corps de requête |
|---|---|---|---|---|
| GET | lire une ressource ou une collection | oui | oui | non |
| HEAD | comme GET, sans le corps | oui | oui | non |
| POST | créer une ressource dans une collection, ou lancer un traitement | non | **non** | oui |
| PUT | remplacer entièrement une ressource | non | oui | oui |
| PATCH | modifier partiellement une ressource | non | non (en général) | oui |
| DELETE | supprimer une ressource | non | oui | non |
| OPTIONS | découvrir les méthodes autorisées (pré-vol CORS) | oui | oui | non |

- **Sûre** : ne modifie pas l'état du serveur.
- **Idempotente** : l'envoyer une fois ou dix fois produit le même état final (pas forcément la même réponse : un second DELETE peut répondre 404).

## Codes de statut les plus utiles
| Code | Quand |
|---|---|
| 200 OK | lecture ou modification réussie, avec un corps |
| 201 Created | création réussie : **toujours** avec l'en-tête `Location` |
| 204 No Content | succès sans corps (DELETE) |
| 304 Not Modified | la version en cache du client est encore valable (`If-None-Match`) |
| 400 Bad Request | requête illisible (JSON mal formé, paramètre incohérent) |
| 401 Unauthorized | pas authentifié (jeton absent, invalide, expiré) |
| 403 Forbidden | authentifié mais pas autorisé |
| 404 Not Found | ressource inexistante (ou que l'on ne veut pas révéler) |
| 405 Method Not Allowed | méthode non prévue sur cette URI (avec `Allow`) |
| 406 Not Acceptable | aucun format demandé dans `Accept` n'est disponible |
| 409 Conflict | conflit avec l'état de la ressource (complet, déjà utilisé, doublon) |
| 412 Precondition Failed | `If-Match` ne correspond plus à la version courante |
| 415 Unsupported Media Type | le `Content-Type` envoyé n'est pas accepté |
| 422 Unprocessable Content | JSON valide mais contenu contraire aux règles (validation) |
| 428 Precondition Required | le serveur exige un `If-Match` |
| 429 Too Many Requests | limitation de débit (avec `Retry-After`) |
| 500 Internal Server Error | bug côté serveur : ne jamais renvoyer de trace au client |
| 502 / 503 / 504 | une dépendance est en panne / service indisponible / délai dépassé |

## En-têtes à connaître
`Content-Type`, `Accept`, `Location`, `ETag`, `If-None-Match`, `If-Match`, `Cache-Control`, `Authorization`, `WWW-Authenticate`, `Retry-After`, `Link`, `Allow`.

## Erreurs : RFC 9457 « Problem Details »
```http
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
  "type": "https://api.orbitale.test/problems/not-found",
  "title": "Ressource introuvable",
  "status": 404,
  "detail": "Événement introuvable",
  "instance": "/v1/events/6f1c2b8e-4f3a-4e8b-9a51-2d7c0f9e1a23"
}
```

## Nommage
- Des **noms au pluriel** : `/v1/events`, jamais `/v1/getEvents` ni `/v1/createEvent`.
- La hiérarchie exprime l'appartenance : `/v1/orders/{orderId}/tickets`.
- Les filtres, le tri et la pagination passent en paramètres de requête : `/v1/events?status=published&sort=-startsAt`.
- La version est visible et stable : `/v1/...`.

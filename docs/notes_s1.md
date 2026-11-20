# Notes TP Séance 1 — Fondamentaux REST

## Étape 1 — Anatomie d'un échange HTTP

- **Ligne de requête :** `GET /repos/nodejs/node HTTP/1.1`
- **Ligne de statut :** `HTTP/1.1 200 OK`
- **Version HTTP utilisée :** `HTTP/1.1`

**Quatre en-têtes de réponse et leur rôle :**
- `Content-Type: application/json; charset=utf-8` : indique au client que le corps de la réponse est au format JSON encodé en UTF-8.
- `Cache-Control: public, max-age=60, s-maxage=60` : précise que la réponse peut être mise en cache pendant 60 secondes (par le client et les caches intermédiaires).
- `ETag: W/"2ba9501cc90ef8b590d118bb95fd19903378c6b8afcdd385812689ba9e404ee0"` : empreinte (identifiant de version) de la ressource, utile pour vérifier plus tard si elle a changé.
- `Content-Length: 6618` : taille du corps de la réponse en octets.

**`X-RateLimit-Remaining` :**
Indique le nombre de requêtes qu'il nous reste avant d'atteindre la limite horaire imposée par GitHub (ici `56` restantes sur `60` autorisées sans authentification). Quand il tombe à `0`, GitHub bloque les appels suivants (erreur `403 Forbidden` / rate limit exceeded) jusqu'au timestamp indiqué dans `X-RateLimit-Reset`.

---

## Étape 2 — Le cache conditionnel

- **Statut obtenu :** `304` (*Not Modified*).
- **Pourquoi le corps est-il vide ?** L'ETag envoyé dans `If-None-Match` correspond toujours à la version actuelle côté serveur. GitHub répond donc `304` sans renvoyer le JSON pour économiser de la bande passante (et côté `curl`, l'option `-o NUL` ignore de toute façon le corps).
- **Intérêt pour une appli mobile :** évite de retélécharger des données inchangées, ce qui économise la data (4G/5G), la batterie et accélère l'affichage.
- **Intérêt pour le quota GitHub :** les requêtes conditionnelles qui renvoient un `304` ne sont pas décomptées du `X-RateLimit-Remaining`.

---

## Étape 3 — Les erreurs

| API | Requête | Statut | Format du corps d'erreur | Information utile pour corriger |
|---|---|---|---|---|
| GitHub | `GET /repos/nodejs/ce-depot-n-existe-pas` | `404` | JSON (`message`, `documentation_url`, `status`) | Indique que le dépôt n'existe pas (`Not Found`) + lien vers la doc |
| GitHub | `DELETE /repos/nodejs/node` | `401` | JSON (`message`, `documentation_url`, `status`) | Indique qu'il faut s'authentifier (`Requires authentication`) + lien vers la doc |
| geo.api.gouv.fr | `GET /communes?codePostal=abc` | `200` | Tableau JSON vide (`[]`) | Aucune info d'erreur, on croit juste qu'il n'y a aucun résultat |

- **Erreur qui ne respecte pas la sémantique HTTP :** `geo.api.gouv.fr` renvoie un `200 OK` avec `[]` alors que le paramètre `codePostal=abc` est clairement invalide (un code postal est composé de chiffres). L'API devrait renvoyer une erreur client (`400 Bad Request` ou `422 Unprocessable Content`).

**Réécriture de l'erreur `geo.api.gouv.fr` au format RFC 9457 :**
```json
{
  "type": "https://geo.api.gouv.fr/problems/invalid-parameter",
  "title": "Paramètre de requête invalide",
  "status": 422,
  "detail": "Le paramètre 'codePostal' doit contenir exactement 5 chiffres (reçu : 'abc').",
  "instance": "/communes?codePostal=abc"
}
```

---

## Étape 4 — Collections et pagination

- **Filtrage et sélection sur `geo.api.gouv.fr` :** on filtre avec des paramètres de requête dans l'URL (`?nom=Lyon`) et on choisit les champs retournés avec le paramètre `fields` (`&fields=nom,code,population`).
- **Page suivante sur GitHub :** l'information se trouve dans un **en-tête HTTP**, nommé `Link` (ex: `Link: <https://api.github.com/...&page=2>; rel="next"`).
- **Avantage des liens en en-tête :** le corps JSON reste propre et ne contient que le tableau de données (pas besoin d'enveloppe métier autour). Un client ou un outil intermédiaire peut aussi lire la pagination sans avoir à parser tout le JSON.

---

## Étape 5 — Négociation de contenu

- **En-tête utilisé :** `Accept`.
  - Avec `application/vnd.github.raw+json`, GitHub renvoie le fichier README en Markdown brut.
  - Avec `application/vnd.github.html+json`, GitHub renvoie le README déjà converti en HTML.
- **Statut si aucun format demandé n'est disponible :** `406 Not Acceptable`.

---

## Étape 6 — L'échelle de Richardson

- **GitHub REST API : Niveau 2 (avec des éléments de niveau 3)**
  - *Justification :* URI orientées ressources (`/repos/nodejs/node`), bon usage des méthodes et statuts HTTP (`GET` -> `200`, `DELETE` -> `401`, ressource absente -> `404`). Elle intègre aussi de l'hypermédia sur certains aspects, comme l'en-tête `Link` avec `rel="next"` pour la pagination.
- **geo.api.gouv.fr : Niveau 2 (incomplet)**
  - *Justification :* URI orientées ressources (`/communes`) et utilisation de `GET` avec filtres, mais gestion des codes de statut imparfaite (`200` au lieu de `400`/`422` sur `codePostal=abc`) et absence de liens hypermédias.
- **Stripe API (exemple d'API connue) : Niveau 2**
  - *Justification :* Ressources bien découpées au pluriel (`/v1/customers`, `/v1/payment_intents`), verbes HTTP respectés (`POST` pour créer, `GET` pour lire, `DELETE` pour supprimer) et codes de statut précis (`400`, `401`, `402`, `404`), mais pas de navigation HATEOAS complète.
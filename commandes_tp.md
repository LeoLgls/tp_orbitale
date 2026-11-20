# Commandes du TP guidé (S1)

> Sous PowerShell, utilise `curl.exe` au lieu de `curl` (sinon c'est un alias d'`Invoke-WebRequest`).

## 1. Anatomie d'un échange HTTP
```bash
# -v affiche la requête envoyée (>) et la réponse reçue (<)
curl -v https://api.github.com/repos/nodejs/node -o /dev/null
```
Repère : la ligne de statut, `Content-Type`, `ETag`, `Cache-Control`, `X-RateLimit-Remaining`.

## 2. Cache conditionnel
```bash
ETAG=$(curl -sI https://api.github.com/repos/nodejs/node | grep -i '^etag' | cut -d' ' -f2 | tr -d '\r')
curl -s -o /dev/null -w "%{http_code}\n" -H "If-None-Match: $ETAG" https://api.github.com/repos/nodejs/node
```
Question : quel code obtiens-tu, et pourquoi la réponse ne contient-elle pas de corps ?

## 3. Les erreurs
```bash
curl -s https://api.github.com/repos/nodejs/ce-depot-n-existe-pas | jq
curl -s -X DELETE https://api.github.com/repos/nodejs/node -w "\n%{http_code}\n"
curl -s "https://geo.api.gouv.fr/communes?codePostal=abc" -w "\n%{http_code}\n"
```
Question : ces API ont-elles un format d'erreur commun ? Lequel est le plus utile au client ?

## 4. Collections, filtres et pagination
```bash
curl -s "https://geo.api.gouv.fr/communes?nom=Lyon&fields=nom,code,population" | jq
curl -sI "https://api.github.com/repos/nodejs/node/issues?per_page=5" | grep -i '^link'
```
Question : comment GitHub indique-t-il la page suivante ? Où est l'information : dans le corps ou dans les en-têtes ?

## 5. Négociation de contenu
```bash
curl -s -H "Accept: application/vnd.github.raw+json" https://api.github.com/repos/nodejs/node/readme | head -5
curl -s -H "Accept: application/vnd.github.html+json" https://api.github.com/repos/nodejs/node/readme | head -5
```

## 6. Échelle de Richardson
Pour chaque API, note le niveau atteint (0 à 3) et justifie :
- GitHub REST API
- geo.api.gouv.fr
- une API que tu utilises en entreprise ou en projet

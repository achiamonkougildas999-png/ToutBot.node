# ToutBot Prestige — version Node.js

Portage complet de `toutbot_app.py` (Flask) vers **Node.js pur**, sans aucune
dépendance npm : uniquement les modules natifs `node:http` et `node:sqlite`
(disponibles depuis Node 22). Donc **pas de `npm install`** nécessaire.

## Lancer en local

```bash
node --version   # il faut Node >= 22.5
node server.js
# ou : npm start
```

Le serveur écoute par défaut sur `http://0.0.0.0:3000`. Le mot de passe
admin par défaut est `Numberone_100_ans` (change-le avec la variable
d'environnement `TOUTBOT_ADMIN_PASSWORD` !).

## Fichiers

- `server.js` — serveur HTTP, routage, sécurité (CSRF, en-têtes, limite de débit), toutes les routes.
- `db.js` — schéma SQLite (`node:sqlite`, natif), migrations, FTS5.
- `session.js` — sessions par cookie, stockées côté serveur en SQLite.
- `business.js` — dépôts/retraits, abonnements, messages privés (logique métier).
- `search.js` — recherche interne (mots exacts, racines FR, vecteurs par hachage) + web (Wikipedia, DuckDuckGo, Brave, SearXNG) + fusion RRF.
- `ai.js` — client Pollinations (chat IA), avec repli automatique sur la recherche si l'IA ne répond pas.
- `templates.js` — toutes les pages HTML (même design/CSS que la version Python).
- `utils.js` — fonctions utilitaires (dates, montants, hachage de mot de passe, etc.).

## Variables d'environnement (mêmes noms que la version Python)

| Variable | Rôle | Défaut |
|---|---|---|
| `TOUTBOT_HOST` / `TOUTBOT_PORT` | Adresse/port d'écoute | `0.0.0.0` / `3000` |
| `TOUTBOT_DB` | Chemin du fichier SQLite | `./toutbot.db` |
| `TOUTBOT_ADMIN_PASSWORD` | Mot de passe admin | `Numberone_100_ans` |
| `TOUTBOT_SECRET_KEY` | (non utilisé — sessions stockées en base, pas de cookie signé) | — |
| `TOUTBOT_COOKIE_SECURE` | `1` pour forcer les cookies `Secure` (HTTPS) | `0` |
| `POLLINATIONS_KEY` | Clé API Pollinations pour l'IA | vide |
| `TOUTBOT_AI_ENDPOINT` / `TOUTBOT_AI_MODEL` | Réglages du moteur IA | auto |
| `TOUTBOT_WEB_SEARCH` | `1`/`0` pour activer la recherche internet | `1` |
| `BRAVE_API_KEY`, `SEARXNG_URL` | Moteurs de recherche web optionnels | vide |

## Différences assumées avec la version Python

1. **Elasticsearch / Qdrant non portés.** Ils étaient optionnels et
   désactivés par défaut dans le fichier Python d'origine. La recherche
   interne (mots exacts FTS5, racines françaises, vecteurs par hachage) et
   la recherche web (Wikipedia, DuckDuckGo, Brave, SearXNG) sont, elles,
   entièrement portées et fusionnées par RRF comme avant.
2. **Proxy HTTP sortant non porté** (`TOUTBOT_HTTP_PROXY`) : `fetch()`
   natif de Node ne le supporte pas sans librairie tierce (`undici`'s
   `ProxyAgent`). À ajouter toi-même si ton hébergeur en a besoin.
3. **Sessions** : stockées côté serveur dans une table SQLite plutôt que
   dans un cookie signé (comme Flask le fait par défaut). Fonctionnellement
   équivalent, et plus simple à auditer.

## Déploiement

N'importe quel hébergeur qui exécute Node.js ≥ 22 convient (Render, Fly.io,
Railway, un VPS avec `pm2`, etc.). Le point d'entrée est `server.js` (ou
`npm start`). Pense à définir `TOUTBOT_ADMIN_PASSWORD` et, si tu veux le
chat IA, `POLLINATIONS_KEY`.

Contrairement à PythonAnywhere, la plupart de ces hébergeurs n'imposent pas
de liste blanche de domaines sortants — donc l'IA et la recherche web
devraient fonctionner directement, sans les soucis d'allowlist rencontrés
avec la version Python.

## Test rapide

`test_flow.js` est un petit script de fumée (inscription, connexion,
publication, recherche, chat, portefeuille) que j'ai utilisé pour valider
ce portage. Pour le relancer :

```bash
node server.js &        # démarre le serveur sur le port 3000
node test_flow.js       # (ajuste le port en tête du fichier si besoin)
```

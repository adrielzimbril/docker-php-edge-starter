# Oricode Docker Starter

Starter d'infrastructure Docker reutilisable pour applications PHP avec Caddy comme proxy edge, HTTPS automatise, MySQL, Redis, cron jobs et support optionnel de Cloudflare Tunnel. Concu pour le developpement local, le staging et les deploiements self-hosted.

[English documentation](./README.en.md)

## ✨ Presentation

Ce depot fournit une base d'infrastructure prete a cloner, adapter et reutiliser pour des projets web qui ont besoin d'une stack simple, lisible et modulaire.

L'objectif n'est pas de livrer une application metier, mais un socle d'execution capable de servir de point de depart pour :
- une application PHP classique
- une plateforme web multi-domaines ou multi-sous-domaines
- un environnement local partage par une equipe
- un environnement de staging ou de preproduction
- une base self-hosted avec HTTPS automatise

Le projet reste volontairement compact :
- un `docker-compose.yml` central
- des configurations lisibles par service dans `.docker/`
- un fichier `.env` pour piloter le runtime
- un edge layer base sur Caddy

## 🎯 Objectifs du projet

Ce starter cherche a offrir un bon equilibre entre simplicite et capacite d'evolution.

Il a ete pense pour :
- demarrer vite
- rester comprehensible sans surcouche inutile
- permettre une adaptation facile a plusieurs contextes
- eviter de publier des secrets ou des donnees runtime
- fournir une base propre pour un depot public ou prive

## 🧱 Composants inclus

### Caddy

`Caddy` joue le role de point d'entree HTTP/HTTPS de la stack.

Il gere :
- la terminaison TLS
- les redirections HTTP vers HTTPS
- le routage par domaine et sous-domaine
- les certificats automatiques via DNS challenge
- le forwarding vers l'application PHP

### MySQL

`MySQL` fournit la persistence relationnelle.

Cas d'usage typiques :
- donnees applicatives
- tables metier
- stockage principal des ressources persistantes

### Redis

`Redis` est prevu pour :
- cache applicatif
- sessions
- queues
- stockage temporaire

### Cron

Le service `cron` permet d'isoler les taches planifiees dans un conteneur dedie.

Cela facilite :
- les executions periodiques
- la separation des responsabilites
- l'evolution des jobs sans polluer le conteneur principal

### Cloudflare Tunnel

Un template de tunnel est inclus pour les environnements qui souhaitent :
- exposer SSH de facon controlee
- router le domaine principal via un tunnel
- centraliser l'entree reseau sans ouvrir directement tous les ports

## 🗺️ Vue d'ensemble

```text
Internet / Tunnel
        |
        v
      Caddy
        |
        v
   PHP application
     |        |
     v        v
   MySQL    Redis
```

## 📁 Structure detaillee

```text
.
|-- .docker/
|   |-- caddy/
|   |   |-- Caddyfile
|   |   |-- Dockerfile
|   |   `-- caddyfile.json
|   |-- cloudflared/
|   |   `-- config/
|   |       `-- config.yml.template
|   |-- cron/
|   |   |-- Dockerfile
|   |   `-- task
|   |-- logs/
|   |-- mysql/
|   |   |-- Dockerfile
|   |   |-- my.cnf
|   |   `-- sql/
|   `-- nginx/
|       |-- default.conf
|       `-- oricode.com.conf
|-- .env.example
|-- .gitignore
|-- AUTHORS.md
|-- CONTRIBUTING.md
|-- LICENSE
|-- README.en.md
|-- README.md
|-- SECURITY.md
`-- docker-compose.yml
```

## 🧭 Role de chaque dossier

- `.docker/caddy/`
  Contient la configuration edge, TLS et reverse proxy.
- `.docker/cloudflared/`
  Contient le template de tunnel.
- `.docker/cron/`
  Contient l'image et la definition des taches cron.
- `.docker/logs/`
  Sert de point de montage pour les logs locaux.
- `.docker/mysql/`
  Contient la configuration MySQL et l'emplacement prevu pour les scripts SQL.
- `.docker/nginx/`
  Conserve des configurations Nginx de reference ou de compatibilite.

## ⚡ Demarrage rapide

### 1. Creer le fichier d'environnement

```powershell
Copy-Item .env.example .env
```

### 2. Adapter les variables

Renseigne au minimum :
- les credentials MySQL
- les informations Redis si necessaire
- les variables de domaine
- les variables Cloudflare si tu veux TLS DNS challenge ou tunnel

### 3. Relire les points d'entree

Avant le premier lancement, verifie :
- `docker-compose.yml`
- `.docker/caddy/Caddyfile`
- `.docker/cloudflared/config/config.yml.template`
- `.env`

### 4. Construire et lancer

```powershell
docker compose up -d --build
```

### 5. Arreter la stack

```powershell
docker compose down
```

## 🔧 Variables d'environnement

Le template [`.env.example`](./.env.example) est la source de reference.

### Bloc MySQL

- `DB_CONNECTION`
  Driver de connexion applicatif.
- `DB_HOST`
  Host MySQL accessible depuis les conteneurs.
- `DB_PORT`
  Port interne MySQL.
- `FORWARD_DB_PORT`
  Port expose cote machine.
- `DB_DATABASE`
  Base creee au demarrage.
- `DB_USERNAME`
  Utilisateur applicatif.
- `DB_PASSWORD`
  Mot de passe applicatif et root dans cette version de la stack.

### Bloc Redis

- `REDIS_HOST`
  Host Redis.
- `REDIS_PASSWORD`
  Mot de passe Redis si active.
- `REDIS_PORT`
  Port interne Redis.
- `FORWARD_REDIS_PORT`
  Port expose cote machine.
- `REDIS_CLIENT`
  Client attendu par l'application.

### Bloc TLS / Cloudflare

- `CLOUDFLARE_API_TOKEN`
  Token utilise pour le DNS challenge.
- `CLOUDFLARE_API_MAIL_PRIMARY`
  Email principal lie a l'automatisation TLS.
- `CLOUDFLARE_API_MAIL`
  Email secondaire consomme par certaines parties de la stack.
- `FORWARD_CADDY_PORT`
  Port HTTP de l'host.
- `FORWARD_CADDY_SSL_PORT`
  Port HTTPS de l'host.

### Bloc runtime / mapping

- `DOCKER_DB_CONTAINER_NAME`
  Nom du conteneur MySQL.
- `WWWGROUP`
  GID dans les conteneurs.
- `WWWUSER`
  UID dans les conteneurs.
- `WWWWORKDIR`
  Chemin de travail monte dans le runtime web.

### Bloc domaine / tunnel

- `TUNNEL_UUID`
  Identifiant du tunnel.
- `DOMAIN`
  Domaine principal.
- `HOST_HOSTNAME`
  Prefixe de hostname pour certains endpoints techniques.

## 🌐 Reverse proxy et TLS

Les fichiers principaux sont :
- [`.docker/caddy/Caddyfile`](./.docker/caddy/Caddyfile)
- [`.docker/caddy/caddyfile.json`](./.docker/caddy/caddyfile.json)

Le `Caddyfile` est la version la plus simple a maintenir.

Le JSON peut etre utile si tu veux :
- comparer une configuration exportee
- conserver une version machine-readable
- preparer une transition vers une autre forme de generation de config

## ☁️ Tunnel distant

Le template [`.docker/cloudflared/config/config.yml.template`](./.docker/cloudflared/config/config.yml.template) peut etre adapte selon ton usage.

Exemples :
- exposer une entree SSH securisee
- router le domaine principal vers le reverse proxy
- router un wildcard vers le meme edge layer

## 🗃️ Base de donnees et cache

### MySQL

Fichiers associes :
- [`.docker/mysql/Dockerfile`](./.docker/mysql/Dockerfile)
- [`.docker/mysql/my.cnf`](./.docker/mysql/my.cnf)

Le dossier `.docker/mysql/sql/` peut servir a stocker :
- scripts d'initialisation
- donnees de seed
- patchs SQL de bootstrap

### Redis

Redis reste volontairement minimaliste pour servir de service de support facilement remplacable ou etendable.

## ⏰ Taches planifiees

Le service cron repose sur :
- [`.docker/cron/Dockerfile`](./.docker/cron/Dockerfile)
- [`.docker/cron/task`](./.docker/cron/task)

Tu peux l'utiliser pour :
- jobs periodiques
- maintenance
- synchronisations internes
- file d'attente simple si ton app s'appuie sur cron

## 🛡️ Fichiers ignores et publication

Le depot est prepare pour rester partageable sans embarquer de donnees sensibles.

Le `.gitignore` couvre notamment :
- `.env`
- `.idea/`
- les certificats et donnees ACME
- les donnees runtime Caddy
- les logs
- les donnees locales MySQL
- les donnees locales Redis

## 🔒 Bonnes pratiques de securite

- ne jamais committer un vrai `.env`
- ne jamais committer de certificats generes
- ne jamais committer de cles privees
- limiter les permissions du token Cloudflare
- faire tourner les secrets si une fuite est suspectee
- verifier l'historique Git avant publication d'un depot public

Pour la politique de signalement, voir [SECURITY.md](./SECURITY.md).

## 🧩 Cas d'usage possibles

Cette base peut convenir pour :
- un projet mono-domaine
- un SaaS multi-tenant par sous-domaines
- une stack de demo ou de staging
- un environnement de dev d'equipe
- un template d'infrastructure reutilisable

## 🛠️ Personnalisation rapide

Tu peux personnaliser :
- les domaines
- les sous-domaines
- les images Docker
- les ports exposes
- les volumes montes
- la strategie TLS
- les routes Caddy
- les jobs cron

## 🧪 Verification manuelle recommandee

Avant de pousser publiquement :
- verifier que `.env` n'est pas tracke
- verifier que les certificats ne sont pas trackes
- verifier que les fichiers runtime ne sont pas trackes
- verifier que les domaines exposes sont volontaires
- verifier que les chemins montes sont suffisamment generiques
- verifier que l'historique Git ne contient pas d'anciens secrets

## 🆘 Troubleshooting

### Le port 80 ou 443 est deja pris

Modifie :
- `FORWARD_CADDY_PORT`
- `FORWARD_CADDY_SSL_PORT`

### Le port MySQL ou Redis est deja utilise

Modifie :
- `FORWARD_DB_PORT`
- `FORWARD_REDIS_PORT`

### Le certificat ne s'emet pas

Verifier :
- le token Cloudflare
- les droits du token
- la zone DNS
- le domaine configure dans `.env`
- la configuration Caddy

### Le tunnel ne route pas

Verifier :
- `TUNNEL_UUID`
- `DOMAIN`
- `HOST_HOSTNAME`
- le template cloudflared
- la resolution DNS

## 🤝 Contribution

Les contributions sont bienvenues. Voir [CONTRIBUTING.md](./CONTRIBUTING.md).

## 📜 Licence

Ce projet est distribue sous licence MIT. Voir [LICENSE](./LICENSE).

## 👤 Maintainer

- Website: <https://www.adrielzimbril.com/>
- GitHub: <https://github.com/adrielzimbril>

## 📝 Conclusion

Ce depot est un socle d'infrastructure technique. Il ne cherche pas a masquer la complexite derriere trop d'outillage, mais a fournir une base claire, publiable et facilement recontextualisable pour des projets web modernes.

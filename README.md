# niamoto-nc

Stack Docker + contenu pour servir le site statique [niamoto.nc](https://niamoto.nc),
généré par [Niamoto](https://github.com/niamoto/niamoto) en mode "Export", derrière
un proxy Traefik.

## Architecture

```
Traefik (host)  →  niamoto-nc (nginx:alpine)  →  ./public/
   |
   ├─ termine SSL (ACME / Let's Encrypt)
   ├─ redirige 80 → 443
   └─ route niamoto.nc, www.niamoto.nc, niamoto.endemia.nc
```

Le stack expose un nginx **sans SSL** ni binding de port : Traefik s'en charge via
labels. Le contenu statique vit dans `public/` (versionné avec le repo).

## Structure

```
niamoto-nc/
├── docker-compose.yml              # Stack nginx + labels Traefik
├── docker-compose.override.yml.example
├── .env.example                    # Variables d'env
├── nginx/
│   └── default.conf                # Config nginx (gzip, cache, sans SSL)
├── public/                         # Site statique généré par Niamoto (Export)
│   ├── index.html
│   ├── api/
│   ├── taxon/
│   ├── shape/
│   ├── plot/
│   └── ...
└── README.md
```

## Prérequis

- Docker + docker compose
- Un Traefik externe avec :
  - Un network Docker (par défaut `traefik`)
  - Un entrypoint HTTPS (par défaut `websecure`)
  - Un certresolver ACME (par défaut `letsencrypt`)

## Installation

```bash
git clone https://github.com/niamoto/niamoto-nc.git
cd niamoto-nc
cp .env.example .env
# Adapter .env selon les noms de network / entrypoint / certresolver de Traefik
docker compose up -d
```

Optionnel — pour ajouter des middlewares Traefik (auth, anubis, géo) :

```bash
cp docker-compose.override.yml.example docker-compose.override.yml
# Éditer override.yml selon le besoin
docker compose up -d
```

## Mise à jour du site

Le site statique est versionné dans `public/`. Workflow :

```bash
# 1. Sur la machine de génération : régénérer le site avec Niamoto Export
#    puis remplacer le contenu de public/

# 2. Commit + push
git add public/
git commit -m "site: update YYYY-MM-DD"
git push

# 3. Sur le serveur (dev.endemia.nc) :
git pull
# nginx voit les nouveaux fichiers immédiatement, pas de redémarrage nécessaire
```

Pour automatiser la mise à jour serveur, un cron `git pull` toutes les N minutes
ou un webhook GitHub → script de pull suffit.

## Variables d'environnement

| Variable | Défaut | Description |
|---|---|---|
| `TRAEFIK_NETWORK` | `traefik` | Network Docker partagé avec Traefik |
| `TRAEFIK_ENTRYPOINT` | `websecure` | Entrypoint Traefik à utiliser |
| `TRAEFIK_CERTRESOLVER` | `letsencrypt` | Certresolver ACME |

## Domaines servis

- `niamoto.nc`
- `www.niamoto.nc`
- `niamoto.endemia.nc`

Pour modifier la liste, éditer la rule du router `niamoto` dans `docker-compose.yml`.

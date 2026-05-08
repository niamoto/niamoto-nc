# niamoto-nc

Stack Docker minimal pour servir le site statique [niamoto.nc](https://niamoto.nc),
généré par [Niamoto](https://github.com/niamoto/niamoto) en mode "Export", derrière
un proxy Traefik.

## Architecture

```
Traefik (host)  →  niamoto-nc (nginx:alpine)  →  /home/niamoto/nginx-www
   |
   ├─ termine SSL (ACME / Let's Encrypt)
   ├─ redirige 80 → 443
   └─ route niamoto.nc, www.niamoto.nc, niamoto.endemia.nc
```

Le stack expose un nginx **sans SSL** ni binding de port : Traefik s'en charge via
labels. Le site statique est monté en read-only.

## Prérequis

- Docker + docker compose
- Un Traefik externe avec :
  - Un network Docker (par défaut `traefik`)
  - Un entrypoint HTTPS (par défaut `websecure`)
  - Un certresolver ACME (par défaut `letsencrypt`)
- Le site statique présent sur le host (par défaut `/home/niamoto/nginx-www`)

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

Le site statique vit dans `/home/niamoto/nginx-www` (configurable via `NIAMOTO_WWW_PATH`).
Pour publier une nouvelle version, remplacer le contenu du dossier — nginx sert
les nouveaux fichiers immédiatement, aucun redémarrage nécessaire.

## Variables d'environnement

| Variable | Défaut | Description |
|---|---|---|
| `NIAMOTO_WWW_PATH` | `/home/niamoto/nginx-www` | Path host du site statique |
| `TRAEFIK_NETWORK` | `traefik` | Network Docker partagé avec Traefik |
| `TRAEFIK_ENTRYPOINT` | `websecure` | Entrypoint Traefik à utiliser |
| `TRAEFIK_CERTRESOLVER` | `letsencrypt` | Certresolver ACME |

## Domaines servis

- `niamoto.nc`
- `www.niamoto.nc`
- `niamoto.endemia.nc`

Pour modifier la liste, éditer la rule du router `niamoto` dans `docker-compose.yml`.

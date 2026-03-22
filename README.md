# Homelab

Stacks Docker Compose pour la maison : infrastructure partagée (reverse proxy, torrents, fichiers) et média (films/séries).

## Architecture

| Stack   | Rôle |
|--------|------|
| **[Core](./core/)** | Infrastructure : Traefik (reverse proxy + TLS), Transmission (client torrent), FileBrowser |
| **[Prism](./prism/)** | Média : indexeur (Jackett), orchestrateurs (Radarr, Sonarr), gestion des demandes (Seerr), serveur (Plex) |
| **[Pulse](./pulse/)** | Jeux : orchestrateur (Questarr), bibliothèque (Romm), streaming NVIDIA/Moonlight (Wolf). *Non inclus dans le compose racine.* |

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Pour Pulse (streaming jeux) : GPU NVIDIA et [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

## Ordre de démarrage et réseaux Docker

Les réseaux **`infra-network`** et **`prism-network`** sont définis dans **[core/docker-compose.yml](./core/docker-compose.yml)**. Il faut **lancer ce compose en premier** (une fois suffit pour créer les réseaux) : ainsi Traefik, Transmission et les stacks qui s’y rattachent peuvent communiquer correctement.

- **Compose à la racine** : l’inclusion de `core/docker-compose.yml` crée ces réseaux au premier `docker compose up` ; l’ordre est alors géré par Compose.
- **Stacks lancées séparément** (`cd prism`, `cd pulse`, etc.) : démarre d’abord **Core** (`cd core && docker compose up -d`), puis Prism ou Pulse. Sans ces réseaux, Prism ne démarre pas (réseaux déclarés `external`) et la communication entre services échoue.

Si tu ne peux pas lancer Core en premier, crée les réseaux **manuellement** :

```bash
docker network create infra-network
docker network create prism-network
```

## Démarrer tout (Core + Prism)

À la racine du dépôt :

```bash
docker compose up -d
```

Le [docker-compose.yml](./docker-compose.yml) à la racine inclut **core** et **prism**. Les données (configs, certificats, médias) sont centralisées dans le dossier **[data/](./data/)** :

| Dossier | Usage |
|---------|--------|
| `data/config/` | Configs par service : Core (transmission, filebrowser), Prism (jackett, radarr, sonarr, seerr, plex), Pulse (questarr, romm, romm-db, wolf) |
| `data/letsencrypt/` | Certificats TLS (Traefik) |
| `data/downloads/` | Téléchargements torrent (Transmission, Radarr, Sonarr, Questarr) |
| `data/watch/` | Torrents à surveiller (Transmission) |
| `data/movies/` | Films (Radarr, Plex) |
| `data/tv/` | Séries (Sonarr, Plex) |
| `data/games/` | Jeux (Questarr, Romm, Wolf) |

Chaque stack utilise des chemins relatifs vers `data/` depuis son répertoire (`core/`, `prism/`, `pulse/`).

## Démarrer une stack seule

**Core en premier** (voir [Ordre de démarrage et réseaux Docker](#ordre-de-démarrage-et-réseaux-docker)), puis :

```bash
# Infrastructure (Traefik, Transmission, FileBrowser) — crée infra-network et prism-network
cd core && docker compose up -d

# Média (Jackett, Radarr, Sonarr, Seerr, Plex) — requiert les réseaux ci-dessus
cd prism && docker compose up -d

# Jeux (Questarr, Romm, Wolf) — requiert infra-network
cd pulse && docker compose up -d
```

Pour que Radarr/Sonarr envoient des torrents à Transmission, les deux stacks doivent tourner et être sur le même réseau (Traefik dans Core est attaché à `prism-network` ; si besoin, attacher aussi Transmission à `prism-network` dans Core).

## Réseaux

Ces réseaux sont **créés par Core** (`core/docker-compose.yml`) ; Prism et Pulse les réutilisent (souvent en `external`). Sans les créer d’abord (via Core ou `docker network create`), les services ne peuvent pas communiquer correctement.

- **infra-network** (Core) : Traefik, Transmission, FileBrowser, Tailscale, etc. Pulse (Questarr, Romm) s’y attache.
- **prism-network** (défini dans Core, utilisé par Prism) : Jackett, Radarr, Sonarr, Seerr, Plex, etc. Traefik est aussi sur ce réseau pour exposer les services en HTTPS.

Voir le README de chaque stack pour les ports, le flux et la sécurité.

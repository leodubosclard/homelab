# Homelab

Stacks Docker Compose pour la maison : infrastructure partagée (reverse proxy, torrents, fichiers) et média (films/séries).

## Architecture

| Stack   | Rôle |
|--------|------|
| **[Core](./core/)** | Infrastructure : Traefik (reverse proxy + TLS), Transmission (client torrent), FileBrowser |
| **[Prism](./prism/)** | Média : indexeur (Jackett), orchestrateurs (Radarr, Sonarr), gestion des demandes (Seerr), serveur (Plex) |
| **[Pulse](./pulse/)** | Jeux : indexeur (Prowlarr), orchestrateur (Questarr), bibliothèque (Romm), streaming NVIDIA/Moonlight (Wolf). *Non inclus dans le compose racine.* |

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Pour Pulse (streaming jeux) : GPU NVIDIA et [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

## Démarrer tout (Core + Prism)

À la racine du dépôt :

```bash
docker compose up -d
```

Le [docker-compose.yml](./docker-compose.yml) à la racine inclut **core** et **prism**. Les données (configs, certificats, médias) sont centralisées dans le dossier **[data/](./data/)** :

| Dossier | Usage |
|---------|--------|
| `data/config/` | Configs par service (transmission, filebrowser, jackett, radarr, sonarr, seerr, plex) |
| `data/letsencrypt/` | Certificats TLS (Traefik) |
| `data/downloads/` | Téléchargements torrent (Transmission, Radarr, Sonarr) |
| `data/watch/` | Torrents à surveiller (Transmission) |
| `data/movies/` | Films (Radarr, Plex) |
| `data/tv/` | Séries (Sonarr, Plex) |

Chaque stack utilise des chemins relatifs vers `data/` depuis son répertoire (`core/`, `prism/`).

## Démarrer une stack seule

```bash
# Infrastructure (Traefik, Transmission, FileBrowser)
cd core && docker compose up -d

# Média (Jackett, Radarr, Sonarr, Seerr, Plex)
cd prism && docker compose up -d
```

Pour que Radarr/Sonarr envoient des torrents à Transmission, les deux stacks doivent tourner et être sur le même réseau (Traefik dans Core est attaché à `prism-network` ; si besoin, attacher aussi Transmission à `prism-network` dans Core).

## Réseaux

- **infra-network** (Core) : Traefik, Transmission, FileBrowser.
- **prism-network** (Prism) : Jackett, Radarr, Sonarr, Seerr, Plex. Traefik est aussi sur ce réseau pour exposer les services en HTTPS.

Voir le README de chaque stack pour les ports, le flux et la sécurité.

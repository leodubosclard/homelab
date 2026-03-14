# Core

Stack d’infrastructure partagée : reverse proxy (Traefik), client torrent (Transmission) et navigateur de fichiers (FileBrowser).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose

## Démarrer

```bash
docker compose up -d
```

## Services

| Service         | Port(s)     | Rôle |
|-----------------|-------------|------|
| **Traefik**     | 80, 443     | Reverse proxy avec TLS (Let’s Encrypt). Expose les services Prism en HTTPS. |
| **Transmission**| 9091, 51413 | Client torrent. Dossier de watch : `../data/watch`. Téléchargements : `../data/downloads`. |
| **FileBrowser** | 80 (interne) | Interface web pour parcourir les fichiers (racine : `../data`). |

Les URLs sont définies via les labels Traefik (ex. `transmission.flix.leodubosclard.com`, `files.flix.leodubosclard.com`).

## Répertoires

Les données sont dans **[../data/](../data/)** (voir README racine) :

- `../data/config/transmission` — configuration Transmission
- `../data/config/filebrowser` — base FileBrowser
- `../data/letsencrypt` — certificats ACME (Traefik)
- `../data/downloads` — téléchargements (en cours et terminés)
- `../data/watch` — déposer des fichiers `.torrent` pour Transmission

## Réseaux

- **infra-network** : Traefik, Transmission, FileBrowser
- **prism-network** : Traefik uniquement (pour router le trafic vers les services Prism)

Pour que Radarr/Sonarr (Prism) envoient des torrents à ce Transmission, ajouter **prism-network** au service `transmission` dans ce `docker-compose.yml`.

## Sécurité

- **Transmission** : identifiants via variables d’environnement (`USER` / `PASS`) — à modifier avant toute exposition.
- **FileBrowser** : pas d’auth par défaut dans cette config — à configurer selon tes besoins.
- **Traefik** : email ACME dans la config — adapte si besoin.

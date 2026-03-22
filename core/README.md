# Core

Stack d’infrastructure partagée : reverse proxy (Traefik), client torrent (Transmission) et navigateur de fichiers (FileBrowser).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose

## Démarrer

Ce fichier compose **crée les réseaux Docker** `infra-network` et `prism-network`. Lance-le **en premier** si tu déploies les autres stacks (Prism, Pulse) séparément : sans ces réseaux, la communication entre conteneurs ne fonctionne pas (tu peux aussi les [créer manuellement](../README.md#ordre-de-démarrage-et-réseaux-docker) avec `docker network create`).

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

Les blocs `networks` en bas de ce fichier **déclarent et créent** `infra-network` et `prism-network`. Les autres stacks du dépôt s’y connectent (réseaux souvent marqués `external` dans leur compose) : lancer **ce** compose avant Prism/Pulse évite les erreurs au démarrage.

- **infra-network** : Traefik, Transmission, FileBrowser, etc.
- **prism-network** : Traefik (et les services Prism une fois leur stack démarrée ; Traefik route le trafic vers Prism)

Pour que Radarr/Sonarr (Prism) envoient des torrents à ce Transmission, ajouter **prism-network** au service `transmission` dans ce `docker-compose.yml`.

## Sécurité

- **Transmission** : identifiants via variables d’environnement (`USER` / `PASS`) — à modifier avant toute exposition.
- **FileBrowser** : pas d’auth par défaut dans cette config — à configurer selon tes besoins.
- **Traefik** : email ACME dans la config — adapte si besoin.

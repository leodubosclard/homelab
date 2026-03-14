# Pulse

Stack jeux : téléchargement et streaming de jeux PC via torrents, avec une bibliothèque type Plex (Romm) et streaming NVIDIA/Moonlight (Wolf).

**Note :** Cette stack n’est pas incluse dans le `docker-compose.yml` racine. Pour l’utiliser, lance son propre compose depuis ce dossier (voir ci‑dessous).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Pour Wolf : GPU NVIDIA et [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
- Un client torrent sur un réseau partagé (ex. **torrent-network**), par ex. Transmission (Core) si tu l’y attaches.

## Démarrer

```bash
docker compose up -d
```

*(À condition qu’un `docker-compose.yml` soit présent dans ce dossier.)*

## Services

| Service      | Port  | Rôle |
|-------------|-------|------|
| **Tailscale**  | —     | VPN : accès distant au homelab (network host) |
| **Prowlarr**   | 9696  | Indexeur : recherche de torrents pour Questarr |
| **Questarr**   | 5000  | Orchestrateur : recherche de jeux via Prowlarr, téléchargement via le client torrent, déplacement vers `./games` |
| **Romm**       | 8080  | Bibliothèque de jeux : métadonnées et gestion de `./games` (type Plex pour les jeux) |
| **Romm DB**    | —     | Base MariaDB pour Romm |
| **Wolf**       | —     | Streaming jeux : serveur NVIDIA/Moonlight ; stream depuis `./games` (network host, GPU) |

## Flux

1. Dans **Questarr** : ajouter un jeu (recherche, plateforme, etc.).
2. Questarr interroge **Prowlarr** pour les torrents.
3. Questarr déclenche le téléchargement (le client torrent doit être sur le même réseau, ex. Transmission).
4. Les fichiers terminés sont déplacés dans `./games`.
5. **Romm** indexe `./games` pour la bibliothèque ; **Wolf** stream les jeux vers les clients Moonlight.

## Répertoires

- `prowlarr/`, `questarr/` — config Prowlarr et Questarr
- `romm/`, `romm-db/` — données Romm et base
- `wolf/` — config Wolf
- `tailscale/` — état Tailscale
- `downloads/` — téléchargements (partagés avec le client torrent)
- `games/` — jeux installés (alimenté par Questarr, utilisé par Romm et Wolf)

## Sécurité

- **Romm / Romm DB** : mots de passe et secrets dans le compose (`DB_PASSWD`, `ROMM_AUTH_SECRET`, etc.) — à changer avant exposition.
- **Tailscale** : suivre la [doc Tailscale](https://tailscale.com/kb/1025/install-linux/#authenticate-the-device) pour joindre le réseau.

## Réseau

- **Tailscale** et **Wolf** utilisent `network_mode: host` (nécessaire pour Tailscale et la découverte Moonlight).
- Les autres services utilisent le bridge interne (ex. **torrent-network**) ; le client torrent (ex. Transmission dans Core) doit être sur ce réseau pour que Questarr l’utilise.

# Pulse

Stack jeux : téléchargement et streaming de jeux PC via torrents, avec une bibliothèque type Plex (Romm) et streaming NVIDIA/Moonlight (Wolf).

**Note :** Cette stack n’est pas incluse dans le `docker-compose.yml` racine. Pour l’utiliser, lance son propre compose depuis ce dossier (voir ci‑dessous).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Pour Wolf : GPU NVIDIA et [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
- Un client torrent sur le réseau **infra-network** (ex. Transmission dans [Core](../core/)) pour que Questarr puisse y envoyer les téléchargements.

## Démarrer

Depuis ce dossier :

```bash
docker compose up -d
```

*(Les réseaux Docker partagés avec le reste du homelab sont créés par **[Core](../core/)** : lance `core/docker-compose.yml` en premier. Sinon **infra-network** n’existe pas, la communication avec Transmission / les autres services ne fonctionne pas ; tu peux aussi créer le réseau à la main — voir le [README racine](../README.md#ordre-de-démarrage-et-réseaux-docker).)*

## Services

| Service      | Port  | Rôle |
|-------------|-------|------|
| **Questarr**   | 5000  | Orchestrateur : recherche de jeux, téléchargement via le client torrent, déplacement vers `../data/games` |
| **Romm**       | 8080  | Bibliothèque de jeux : métadonnées et gestion de `../data/games` (type Plex pour les jeux) |
| **Romm DB**    | —     | Base MariaDB pour Romm |
| **Wolf**       | —     | Streaming jeux : serveur NVIDIA/Moonlight ; stream depuis `../data/games` (network host, GPU) |

## Flux

1. Dans **Questarr** : ajouter un jeu (recherche, plateforme, etc.).
2. Questarr déclenche le téléchargement (le client torrent doit être sur **infra-network**, ex. Transmission dans Core).
3. Les fichiers terminés sont déplacés dans `../data/games`.
4. **Romm** indexe `../data/games` pour la bibliothèque ; **Wolf** stream les jeux vers les clients Moonlight.

## Répertoires

Les données sont dans **[../data/](../data/)** (voir README racine) :

- `../data/config/questarr` — config Questarr
- `../data/config/romm/`, `../data/config/romm-db/` — config et données Romm + base
- `../data/config/wolf/` — config Wolf
- `../data/downloads/` — téléchargements (partagé avec Core / Prism)
- `../data/games/` — jeux installés (alimenté par Questarr, utilisé par Romm et Wolf)

## Sécurité

- **Romm / Romm DB** : mots de passe et secrets dans le compose (`DB_PASSWD`, `ROMM_AUTH_SECRET`, etc.) — à changer avant exposition.

## Réseau

- **Wolf** utilise `network_mode: host` (nécessaire pour la découverte Moonlight et le GPU).
- Les autres services utilisent le bridge **infra-network** (externe) ; le client torrent (ex. Transmission dans Core) doit être sur ce réseau pour que Questarr l’utilise.

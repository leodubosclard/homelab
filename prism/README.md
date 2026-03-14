# Prism

Stack média : téléchargement et visionnage de films et séries via torrents, avec Plex (et optionnellement Jellyfin).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Un client torrent accessible (par ex. **Transmission** dans [Core](../core/)) ; pour que Radarr/Sonarr l’utilisent, il doit être sur le même réseau (ex. **prism-network**).

## Démarrer

Depuis ce dossier :

```bash
docker compose up -d
```

Ou à la racine du dépôt : `docker compose up -d` (inclut Core + Prism).

## Services

| Service   | Port  | Rôle |
|----------|-------|------|
| **Jackett** | 9117 | Indexeur : recherche de torrents (trackers publics/privés) pour Radarr et Sonarr |
| **Radarr**  | 7878 | Orchestrateur films : recherche via Jackett, envoi des torrents au client, déplacement des fichiers terminés vers `../movies` |
| **Sonarr**  | 8989 | Orchestrateur séries : idem pour les séries, sortie dans `../tv` |
| **Seerr**   | 5055 | Gestion des demandes : demandes de films/séries envoyées aux orchestrateurs |
| **Plex**    | 32400 | Serveur média : sert films et séries depuis `../movies` et `../tv` |

Jellyfin est présent en commentaire dans le compose comme alternative à Plex.

## Flux

1. Dans **Seerr** (ou directement dans Radarr/Sonarr) : ajouter un film ou une série.
2. Radarr/Sonarr interroge **Jackett** pour trouver des torrents.
3. Radarr/Sonarr envoie le torrent au client (ex. Transmission dans Core) ou tu déposes un `.torrent` dans le dossier watch du client.
4. Le client télécharge dans `../downloads` ; Radarr/Sonarr déplace les fichiers finis vers `../movies` ou `../tv`.
5. **Plex** indexe ces dossiers et tu peux lire le contenu sur tes appareils.

## Répertoires

Les chemins sont relatifs au parent du dossier `prism/` (racine du dépôt si tu lances depuis la racine) :

- `../config/jackett`, `../config/radarr`, `../config/sonarr`, `../config/overseerr`, `../config/plex` — config par service
- `../downloads` — téléchargements (partagés avec le client torrent)
- `../movies` — films prêts à lire (alimenté par Radarr, servi par Plex)
- `../tv` — séries (alimenté par Sonarr, servi par Plex)

## Réseau

Tous les services tournent sur le bridge **prism-network**. Traefik (dans Core) est attaché à ce réseau pour exposer les interfaces en HTTPS.

## Sécurité

- Changer les identifiants par défaut des interfaces (Jackett, Radarr, Sonarr, Seerr, Plex) avant toute exposition.
- Les URLs et le TLS sont gérés par Traefik (Core) via les labels du compose.

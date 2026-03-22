# Prism

Stack média : téléchargement et visionnage de films et séries via torrents, avec Plex (et optionnellement Jellyfin).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Les réseaux **`infra-network`** et **`prism-network`** doivent exister avant ce compose (ils sont déclarés `external`). **Lance d’abord** [Core](../core/) : `core/docker-compose.yml` les crée. Sinon la communication entre services ne fonctionnera pas ; tu peux aussi les créer à la main (voir le [README racine](../README.md#ordre-de-démarrage-et-réseaux-docker)).
- Un client torrent accessible (par ex. **Transmission** dans Core) ; pour que Radarr/Sonarr l’utilisent, il doit être sur le même réseau (ex. **prism-network**).

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
| **Radarr**  | 7878 | Orchestrateur films : recherche via Jackett, envoi des torrents au client, déplacement des fichiers terminés vers `../data/movies` |
| **Sonarr**  | 8989 | Orchestrateur séries : idem pour les séries, sortie dans `../data/tv` |
| **Seerr**   | 5055 | Gestion des demandes : demandes de films/séries envoyées aux orchestrateurs |
| **Plex**    | 32400 | Serveur média : sert films et séries depuis `../data/movies` et `../data/tv` |

Jellyfin est présent en commentaire dans le compose comme alternative à Plex.

## Flux

1. Dans **Seerr** (ou directement dans Radarr/Sonarr) : ajouter un film ou une série.
2. Radarr/Sonarr interroge **Jackett** pour trouver des torrents.
3. Radarr/Sonarr envoie le torrent au client (ex. Transmission dans Core) ou tu déposes un `.torrent` dans le dossier watch du client.
4. Le client télécharge dans `../data/downloads` ; Radarr/Sonarr déplace les fichiers finis vers `../data/movies` ou `../data/tv`.
5. **Plex** indexe ces dossiers et tu peux lire le contenu sur tes appareils.

## Répertoires

Les données sont dans **[../data/](../data/)** (voir README racine) :

- `../data/config/jackett`, `../data/config/radarr`, `../data/config/sonarr`, `../data/config/seerr`, `../data/config/plex` — config par service
- `../data/downloads` — téléchargements (partagés avec le client torrent)
- `../data/movies` — films prêts à lire (alimenté par Radarr, servi par Plex)
- `../data/tv` — séries (alimenté par Sonarr, servi par Plex)

## Réseau

Tous les services tournent sur le bridge **prism-network**. Ce compose attend aussi **infra-network** et **prism-network** en `external` : ils sont **créés par** [Core](../core/) (`docker compose up` dans `core/`). Sans cela, le démarrage échoue ou la communication entre services ne fonctionne pas (alternative : création manuelle des réseaux, voir le [README racine](../README.md#ordre-de-démarrage-et-réseaux-docker)). **Traefik** (Core) est attaché à **prism-network** pour exposer les interfaces en HTTPS.

## Sécurité

- Changer les identifiants par défaut des interfaces (Jackett, Radarr, Sonarr, Seerr, Plex) avant toute exposition.
- Les URLs et le TLS sont gérés par Traefik (Core) via les labels du compose.

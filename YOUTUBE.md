# 🎬 YouTube → Audious (workflow optionnel)

> ⚠️ **Important (légal / CGU)**  
> Audious gère une bibliothèque **locale**. Tout téléchargement depuis des plateformes tierces (YouTube, etc.) dépend de votre usage, des CGU applicables et du droit d’auteur.  
> Les scripts ci-dessous sont de l’outillage **personnel / optionnel** et ne font pas partie du cœur “API/Frontend” d’Audious.

Ce guide documente le script :

- `scan/download_youtube_playlist.sh`

…et comment l’utiliser avec :

- `scan/import.php` (ingestion Audious)
- `sync_to_server.sh` *(optionnel, si vous téléchargez sur une autre machine)*
- `find_youtube_urls.sh` *(optionnel, extraction d’URLs depuis une tracklist)*

---

## 1) À quoi sert `download_youtube_playlist.sh` ?

`download_youtube_playlist.sh` automatise un pipeline :

1. **Lister** les IDs d’une playlist YouTube
2. **Télécharger** proprement chaque item via `yt-dlp` + `ffmpeg` (**MP3**)
3. **Déposer** le résultat dans un **STAGING_DIR** (séparé de `AUDIO_DIR`)
4. *(optionnel)* **Importer** automatiquement dans Audious via `scan/import.php --move`

Le script gère plusieurs cas “réalistes” :
- timeouts “anti-freeze”
- proxy optionnel + détection d’accessibilité
- cookies optionnels
- stratégies de fallback clients/formats
- quarantaine (`blocked`) + *retry* automatique
- cas “copyright blocked” isolé dans un log dédié

---

## 2) Pré-requis

Sur la machine qui exécute le script :

- `yt-dlp`
- `ffmpeg`
- `bash`
- `php` **uniquement si** `DOWNLOAD_ONLY=0` (import automatique)

> Sur macOS : `timeout` peut manquer (Coreutils). Le script continue sans `timeout`, mais le “hard timeout” par commande ne s’appliquera pas.

---

## 3) Chargement de configuration `.env`

Au lancement, le script tente de charger un `.env` dans cet ordre :

1. `scan/.env`
2. `.env` à la racine du projet

Puis il applique ses **defaults** (surchargeables via `.env`).

Extrait (log) :
- `[INFO] Loaded env from ...`

---

## 4) Variables d’environnement importantes

### Pipeline & dossiers

| Variable | Rôle | Exemple |
|---|---|---|
| `PLAYLIST_URL` | URL de la playlist | `https://www.youtube.com/playlist?list=...` |
| `STAGING_DIR` | Dossier “tampon” pour les downloads (DOIT être séparé) | `/home/ubuntu/audious/scan/staging` |
| `AUDIO_DIR` | Dossier **final** d’Audious (utilisé par `import.php`) | `/home/ubuntu/music` |
| `IMPORT_PHP` | Chemin de `import.php` | `/home/ubuntu/audious/scan/import.php` |
| `DOWNLOAD_ONLY` | Mode de fonctionnement (voir §5) | `0` ou `1` |

✅ Sécurité : le script refuse si `STAGING_DIR` est *dans* `AUDIO_DIR`.

### Auth / cookies / proxy

| Variable | Rôle | Notes |
|---|---|---|
| `COOKIE_FILE` | Chemin cookie Netscape | utilisé si présent |
| `YOUTUBE_COOKIES` | alias possible | `COOKIE_FILE` peut pointer dessus |
| `YT_PROXY` | proxy (optionnel) | testé (tcp) puis activé/désactivé |
| `FORCE_AUDIO_ONLY` | forcé à `1` si proxy validé | stabilise le téléchargement |

### Réseau / robustesse / anti-freeze

| Variable | Rôle | Default (dans le script) |
|---|---|---|
| `YT_CMD_TIMEOUT` | timeout dur par appel yt-dlp | `600` |
| `YT_SOCKET_TIMEOUT` | socket timeout yt-dlp | `15` |
| `YT_RETRIES` | retries yt-dlp | `10` |
| `YT_RETRY_SLEEP` | pause entre retries yt-dlp | `5` |
| `YT_SLEEP_REQUESTS` | sleep-requests yt-dlp | `2` |
| `PER_ITEM_SLEEP` | pause entre les items | `0` |
| `YT_SLEEP_INTERVAL` | sleep-interval yt-dlp (fixe ou plage) | vide = off |
| `YT_CONCURRENT_FRAGMENTS` | fragments simultanés HLS | `1` (= +stable) |
| `YT_LIMIT_RATE` | limite débit | vide = off |
| `YT_GEO_BYPASS_COUNTRY` | geo-bypass-country | ex `FR` |

---

## 5) Les 2 modes : `DOWNLOAD_ONLY`

Le script a deux modes principaux :

### ✅ Mode A — Download **only** (ex: Mac)

- `DOWNLOAD_ONLY=1`
- Le script télécharge MP3 + metadata + thumbnail dans `STAGING_DIR`
- Il **n’appelle jamais** `import.php`

Exemple :

```bash
cd /path/to/audious/scan
DOWNLOAD_ONLY=1 STAGING_DIR="$PWD/staging" ./download_youtube_playlist.sh
```

### ✅ Mode B — Download **+ import** (ex: serveur)

- `DOWNLOAD_ONLY=0` *(défaut)*
- Après download, le script lance :

```bash
php scan/import.php -f "<fichier>" --move --verbose
```

Exemple :

```bash
cd /home/ubuntu/audious/scan
DOWNLOAD_ONLY=0 ./download_youtube_playlist.sh
```

---

## 6) Logs & fichiers générés

Le script maintient plusieurs logs :

| Fichier | Rôle |
|---|---|
| `download_youtube_playlist.log` | log principal |
| `download_youtube_playlist_processed.log` | URLs marquées “ok/traitées” |
| `download_youtube_playlist_blocked.log` | URLs en quarantaine (anti-bot, no output, etc.) |
| `download_youtube_playlist_copyright.log` | URLs bloquées copyright (définitif) |
| `scan/.downloads.log` | archive yt-dlp (évite de re-télécharger) |

---

## 7) Quarantaine & retry automatique

À chaque exécution, le script commence par :

1. **Rejouer** les URLs présentes dans `blocked.log` (*une seule passe*)
2. Puis traiter la playlist courante

Pendant le retry, il passe `RETRY_CONTEXT=1` :
- si un item échoue encore → il peut être marqué “processed” pour éviter la boucle infinie

---

## 8) Astuces d’exploitation (recommandées)

### A) “Download sur Mac → Sync serveur → Import serveur”

Si vous téléchargez sur Mac (mode A), vous pouvez ensuite synchroniser vers le serveur puis importer.

#### Exemple `sync_to_server.sh` (optionnel)

Le script peut être très simple, par exemple à base de `rsync` :

```bash
#!/usr/bin/env bash
set -euo pipefail

LOCAL_STAGING="${LOCAL_STAGING:-./scan/staging}"
REMOTE_USER="${REMOTE_USER:-ubuntu}"
REMOTE_HOST="${REMOTE_HOST:-srv855403}"
REMOTE_STAGING="${REMOTE_STAGING:-/home/ubuntu/audious/scan/staging}"

rsync -av --progress --partial   "$LOCAL_STAGING/"   "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_STAGING}/"
```

Puis, côté serveur :

```bash
cd /home/ubuntu/audious
php scan/import.php -d /home/ubuntu/audious/scan/staging --move --verbose
```

> Ajustez selon vos options réelles de `import.php` (fichier vs dossier).

### B) Préparer une liste d’URLs à partir d’une tracklist

Si vous avez un `tracks.txt` (tracklist texte), vous pouvez générer un TSV :

```bash
./find_youtube_urls.sh ../tracks.txt ../youtube_urls.tsv
```

Ensuite, selon votre outil, vous pouvez itérer sur la colonne URL / ID pour alimenter un autre pipeline (ou enrichir une playlist).

---

## 9) Exemple minimal de `.env` (scan/.env)

```env
# Playlist à traiter
PLAYLIST_URL="https://www.youtube.com/playlist?list=PLxxxxxxxxxxxxxxxx"

# Dossiers
STAGING_DIR="/home/ubuntu/audious/scan/staging"
AUDIO_DIR="/home/ubuntu/music"
IMPORT_PHP="/home/ubuntu/audious/scan/import.php"

# Mode
DOWNLOAD_ONLY=0

# Optionnel
# YOUTUBE_COOKIES="/home/ubuntu/.config/yt/cookies.txt"
# YT_PROXY="http://user:pass@host:port"
# YT_SLEEP_INTERVAL="1-3"
# YT_LIMIT_RATE="1M"
```

---

## 10) Dépannage rapide

- **Le script refuse** : `STAGING_DIR must not be inside AUDIO_DIR`  
  → mettez un staging séparé (recommandé et plus safe).

- **Beaucoup de `blocked` / anti-bot**  
  → essayez avec des cookies (`YOUTUBE_COOKIES`) et/ou ajustez `YT_SLEEP_INTERVAL`.

- **429 Too Many Requests**  
  → le script tente un retry sans proxy ; sinon, laissez refroidir + augmentez les sleeps.

- **Copyright blocked**  
  → l’URL est ajoutée à `*_copyright.log` **et** marquée `processed` (skip définitif).

---

### Annexe — “Ce script n’est pas le cœur d’Audious”

Ce workflow est volontairement documenté **à part** :
- il aide à alimenter une bibliothèque *privée*,
- mais il ne fait pas partie du “produit Web/API” Audious,
- et vous restez responsable des usages / sources.

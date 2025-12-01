# 🎧 **Audious**

![PHP](https://img.shields.io/badge/PHP-8.3-blue?logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-00758F?logo=mysql&logoColor=white)
![JS](https://img.shields.io/badge/Vanilla_JS-yellow?logo=javascript&logoColor=white)
![SCSS](https://img.shields.io/badge/SCSS-CF649A?logo=sass&logoColor=white)
![Source](https://img.shields.io/badge/Source-Private-red)
![Build](https://img.shields.io/badge/Build-manual-success?logo=github&logoColor=white)
![Made with ❤️ on Oléron Island 🇫🇷](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F%20on-Ol%C3%A9ron%20Island%20🇫🇷-blueviolet)

> **Audious** est une plateforme **d’auto‑hébergement de bibliothèque audio** minimaliste, rapide et open-source.  
> Elle permet de créer votre propre service d’écoute – strictement **privé** ou **ouvert à d’autres utilisateurs** (amis, communauté, intranet…) – pour organiser, taguer et lire vos fichiers audio.  
> Audious diffuse uniquement les fichiers que vous lui fournissez : il vous appartient de n’importer que des contenus pour lesquels vous disposez des droits ou d’une autorisation d’usage conforme à la loi et aux CGU des plateformes concernées. Certains scripts d’exemple (notamment ceux s’appuyant sur des outils externes comme `yt-dlp`) sont fournis uniquement pour un usage strictement personnel et ne font pas partie du cœur officiel du projet.  
> L’ensemble s’utilise via une interface web au design **cyber‑néon**.

---

## 📚 Sommaire

1. [⚡️ Stack technique](#️-stack-technique)
2. [🚀 Fonctionnalités principales](#-fonctionnalités-principales)
3. [🧰 Installation rapide](#-installation-rapide)
4. [🗄️ Base de données](#️-base-de-données)
5. [⚙️ Configuration `.env`](#️-configuration-env)
6. [🗃️ Permissions & dossiers système](#-permissions--dossiers-système)
7. [📧 Email & vérification de compte](#-email--vérification-de-compte)
8. [🎵 Import audio & gestion des tags](#-import-audio--gestion-des-tags)
9. [🔗 Intégration avec outils externes (yt-dlp, etc.)](#-intégration-avec-outils-externes-yt-dlp-etc)
10. [🧱 Structure simplifiée](#-structure-simplifiée)
11. [🧩 Commandes CLI intégrées](#-commandes-cli-intégrées)
12. [🧪 Sécurité & modes d’exécution](#-sécurité--modes-dexécution)
13. [⚠️ Avertissement légal (contenus tiers)](#️-avertissement-légal-contenus-tiers)
14. [🔮 Roadmap](#-roadmap)
15. [🧾 Licence](#-licence)

---

## ⚡️ **Stack technique**

| Type        | Tech |
|------------|------|
| 🖥 Backend  | PHP 8.3 • MySQL 8 • PDO • Composer • Dotenv • Monolog • getID3 |
| 🎛 Frontend | Vanilla JS (ES6) • SCSS • HTML5 Audio API |
| 📡 Infra    | Apache/Nginx • Icecast (radio) • Bash • CLI PHP dédié |
| 🎨 Design   | Thème “Cyber” : Orbitron + Inter, bleu néon, sombre |

---

## 🚀 **Fonctionnalités principales**

- 🎵 Lecture fluide et continue (footer audio player persistant)
- 🔁 Contrôles Shuffle / Repeat / Seek
- ❤️ Système de likes et compteur dynamique
- 🧠 Recherche par texte + filtrage par tags
- 🏷 Génération de genres et tags à partir des métadonnées (configurable)
- 📦 Import de fichiers audio locaux (dossiers, fichiers uniques, scripts externes)
- 📻 Intégration Icecast avec historique enrichi (`icecast_history`)
- 🎨 Pochettes auto-générées (WebP carré) + nettoyage des artworks orphelins
- 👤 Authentification, création de compte et **vérification email**
- ⚙️ CLI intégré pour maintenance, purge, tags, utilisateurs, déduplication
- 🌌 Interface responsive au design futuriste (thème “Cyber” bleu néon)

> ℹ️ Audious se concentre sur la **gestion et la lecture de fichiers audio déjà présents sur votre serveur**.  
> La récupération de contenus depuis des plateformes tierces (YouTube, etc.) reste sous votre entière responsabilité et n’est pas une fonction centrale du projet.

---

## 🧰 **Installation rapide**

```bash
git clone https://github.com/vpalluel/Audious.git
cd Audious
composer install
cp .env.example .env
# puis éditez .env pour configurer la base, les chemins, l’email, etc.
```

---

## 🗄️ **Base de données**

```bash
mysql -u root -p
CREATE DATABASE audious CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;

mysql -u root -p audious < db/audious.sql
```

> Le schéma complet est dans `db/audious.sql` — tables (`songs`, `albums`, `artists`, `tags`, `song_tags`, `likes`, `playlists`, etc.), clés étrangères et indexes d’intégrité.

---

## ⚙️ **Configuration `.env`**

### 🌍 Application

| Variable   | Description |
|-----------|-------------|
| `APP_URL` | URL publique de l’instance (ex. `https://audious.dev`) |

---

### 🗄️ Base de données

| Variable      | Description |
|--------------|-------------|
| `DB_HOST`    | Hôte MySQL (`localhost` en général) |
| `DB_NAME`    | Nom de la base (`audious`) |
| `DB_USER`    | Utilisateur MySQL |
| `DB_PASS`    | Mot de passe MySQL |
| `DB_CHARSET` | Charset (`utf8mb4` recommandé) |

---

### 🎨 Fichiers & médias

| Variable      | Description |
|--------------|-------------|
| `AUDIO_DIR`  | Chemin **absolu** vers les fichiers audio (ex. `/home/ubuntu/music`) |
| `ASSETS_DIR` | Dossier **relatif à `/public`** pour les artworks générés (ex. `assets/images/artwork/`) |

> L’image par défaut utilisée pour les morceaux sans pochette est `assets/images/default.webp`.

---

### 🧠 Import & métadonnées

| Variable               | Description |
|------------------------|-------------|
| `SUPPORTED_EXTENSIONS` | Extensions audio supportées (`mp3,flac,wav,ogg,opus,webm,...`) |
| `INSERT_AUDIO_TAGS`    | `true` / `false` — extrait les tags (mood, grouping, comments…) des fichiers audio et les insère dans `tags` |
| `TAG_GENRE_AS_TAG`     | `true` / `false` — ajoute aussi le **genre** (normalisé) comme tag en plus de la table `genres` |

Comportement de base à l’import (`scan/import.php`) :

- détection de doublons par **checksum SHA-1** (évite les copies de fichiers identiques) ;
- détection de doublon logique par `(titre, artiste, album)` ;
- vérification de l’espace disque avant copie/déplacement ;
- génération d’une pochette WebP carrée par album lorsque des données d’artwork sont disponibles.

---

### 📤 Upload

| Variable        | Description |
|----------------|-------------|
| `UPLOAD_DEBUG` | `1` pour loguer les détails d’upload, `0` sinon |
| `UPLOAD_MAX_MB`| Taille maximale d’upload (en Mo) |

---

## 🗃️ **Permissions & dossiers système**

Audious manipule des fichiers sur le disque (audio, artworks, staging).  
Pour que l’application fonctionne correctement **en web** (`www-data`) et **en CLI** (`ubuntu` par exemple), certains dossiers doivent être accessibles en écriture.

### Dossiers critiques

Dans un setup classique :

```env
AUDIO_DIR=/home/ubuntu/music
ASSETS_DIR=assets/images/artwork/
STAGING_DIR=/home/ubuntu/audious/scan/staging
```

#### `AUDIO_DIR` — fichiers audio finaux

- Chemin absolu, ex. `/home/ubuntu/music`.
- Utilisé par `scan/import.php` comme **destination finale** des fichiers audio, sous forme `UUID.ext`.
- Dans la base, `songs.file_path` stocke uniquement `UUID.ext` (relatif à `AUDIO_DIR`).
- Doit :
  - exister,
  - être inscriptible par l’utilisateur CLI (ex. `ubuntu`),
  - être lisible/éventuellement inscriptible par le user web (ex. `www-data`).

#### `ASSETS_DIR` — pochettes (artworks)

- Dossier **relatif à `public/`**, par exemple :

  ```text
  /home/ubuntu/audious/public/assets/images/artwork
  ```

- Utilisé par `scan/import.php` pour enregistrer les pochettes générées en WebP (`*.webp`).
- Doit :
  - exister (ou pouvoir être créé par PHP),
  - être inscriptible par le user qui lance `php scan/import.php` (CLI) et, idéalement, lisible par le webserver pour les servir.

#### `STAGING_DIR` — zone tampon (staging uploads)

- Dossier absolu, ex. `/home/ubuntu/audious/scan/staging`.
- Utilisé comme **zone tampon** :
  - par les scripts CLI (pré-traitement de fichiers),
  - par l’API d’upload web (upload → `STAGING_DIR` → `scan/import.php --move` → `AUDIO_DIR`).
- Doit :
  - être inscriptible à la fois par l’utilisateur CLI (ex. `ubuntu`) et par `www-data`.

### Exemple de configuration des droits

En supposant :

- projet dans `/home/ubuntu/audious`  
- utilisateur système principal : `ubuntu`  
- utilisateur/groupe web : `www-data`

```bash
# AUDIO_DIR (fichiers audio finaux)
sudo mkdir -p /home/ubuntu/music
sudo chown -R ubuntu:www-data /home/ubuntu/music
sudo chmod -R 775 /home/ubuntu/music

# STAGING_DIR (zone tampon upload / scripts)
sudo mkdir -p /home/ubuntu/audious/scan/staging
sudo chown -R ubuntu:www-data /home/ubuntu/audious/scan/staging
sudo chmod -R 775 /home/ubuntu/audious/scan/staging

# ASSETS_DIR (artworks WebP)
sudo mkdir -p /home/ubuntu/audious/public/assets/images/artwork
sudo chown -R ubuntu:www-data /home/ubuntu/audious/public/assets/images/artwork
sudo chmod -R 775 /home/ubuntu/audious/public/assets/images/artwork
```

> 💡 Recommandation :
> - `owner` = utilisateur CLI (ex. `ubuntu`)  
> - `group` = user/groupe web (ex. `www-data`)  
> - permissions `775` sur les dossiers → `owner` et `group` peuvent lire/écrire, les autres seulement lire.

---

## 📧 **Email & vérification de compte**

Audious supporte plusieurs drivers d’email via `MAIL_DRIVER` :

| Driver    | Effet |
|-----------|-------|
| `smtp`    | Envoi via un serveur SMTP (ex. Free) |
| `mailgun` | Envoi via l’API Mailgun |
| `log`     | Aucun email envoyé, tout est logué dans `logs/mail.log` |
| `null`    | No-op complet (utile en dev ou pour les démos) |

### Variables mail

| Variable             | Description |
|----------------------|-------------|
| `MAIL_DRIVER`        | `smtp`, `mailgun`, `log`, `null` |
| `MAIL_FROM_ADDRESS`  | Adresse “from” (ex. `audious@app.dev`) |
| `MAIL_FROM_NAME`     | Nom d’expéditeur (ex. `Audious`) |
| `MAIL_REPLY_TO`      | Adresse de réponse (optionnelle) |

#### SMTP

| Variable     | Description |
|-------------|-------------|
| `SMTP_HOST` | Hôte SMTP (ex. `smtp.example.com`) |
| `SMTP_PORT` | Port SMTP (`587` recommandé) |
| `SMTP_USER` | Utilisateur SMTP |
| `SMTP_PASS` | Mot de passe SMTP |
| `SMTP_FROM` | Adresse utilisée dans `setFrom()` si différent de `MAIL_FROM_ADDRESS` |

#### Mailgun

| Variable         | Description |
|------------------|-------------|
| `MAILGUN_DOMAIN` | Domaine Mailgun (ex. `mg.example.com`) |
| `MAILGUN_API_KEY`| Clé API |
| `MAILGUN_REGION` | `us` ou `eu` |

Les endpoints `public/api/register.php` et `public/api/verify.php` utilisent un helper central (`src/mail.php`) pour envoyer les emails de vérification en fonction du driver sélectionné.  
La page `verify.php` affiche une vue HTML “cyber” lorsqu’elle est ouverte dans un navigateur, et retourne du JSON si `Accept: application/json` ou `?format=json` est utilisé.

---

## 🎵 **Import audio & gestion des tags**

L’import se fait via :

```bash
php scan/import.php --source=/chemin/vers/dossier --recursive
# ou
php scan/import.php --file=/chemin/vers/fichier.mp3
```

### Comportement principal

- copie/déplacement des fichiers vers `AUDIO_DIR` avec des noms UUID (`UUID.ext`) ;
- insertion dans `songs` avec :
  - `file_path` relatif (ex. `b3c2f1d0-... .mp3`) ;
  - `genre_id` (genre normalisé) ;
  - `picture_path` (pochette générée ou image par défaut) ;
  - `checksum_sha1` (pour éviter les doublons de contenu) ;
- liaison avec `artists`, `albums`, `genres` et `tags` via des helpers type `insertOrGetID...`.

### Tags & genres

Selon `.env` :

- `INSERT_AUDIO_TAGS=true`  
  → les tags (mood, grouping, comments…) extraits via getID3 sont insérés dans `tags` / `song_tags`.

- `TAG_GENRE_AS_TAG=true`  
  → le genre normalisé (ex. `Techno`, `Deep House`) est aussi ajouté comme tag, en plus de la table `genres`.

Une logique de règles (alias + regex) permet de normaliser les genres (`config/genre_rules.php`).

Des commandes CLI dédiées permettent de :

- auditer les tags,
- les purger,
- les reconstruire à partir des genres (`tags:rebuild`).

---

## 🔗 **Intégration avec outils externes (yt-dlp, etc.)**

Audious n’inclut **aucun** outil de téléchargement de contenus depuis des plateformes tierces dans son cœur applicatif.  
En revanche, rien ne vous empêche de :

- utiliser des outils externes (par ex. `yt-dlp`, `ffmpeg`, scripts maison) pour **générer des fichiers audio**,
- déposer ces fichiers dans un répertoire de **staging**,
- puis lancer `scan/import.php` pour les intégrer dans votre bibliothèque Audious.

### Exemple de flux possible (à adapter, non garanti)

1. Utiliser un script externe (shell, Python, etc.) pour récupérer des fichiers audio et les placer dans un dossier de staging :

   ```text
   /home/ubuntu/audious/scan/staging
   ```

2. Lancer l’import dans Audious :

   ```bash
   cd /home/ubuntu/audious

   php scan/import.php --source=/home/ubuntu/audious/scan/staging --recursive --move --verbose
   # ou en version courte :
   # php scan/import.php -s /home/ubuntu/audious/scan/staging --move --verbose
   ```

> ⚠️ **Important :**  
> - L’utilisation d’outils comme `yt-dlp` peut être soumise aux **Conditions Générales d’Utilisation** des plateformes concernées (YouTube, etc.) et au **droit d’auteur**.  
> - Vous êtes entièrement responsable de vérifier que vos usages respectent ces règles et que vous disposez des droits nécessaires sur les contenus manipulés.  
> - Audious n’est ni affilié, ni approuvé par YouTube ou toute autre plateforme tierce.

---

## 🧱 **Structure simplifiée**

```text
.
├── bin/
│   └── audious-cli.php         # CLI centralisée (backup, purge, tags, users, files...)
├── config/
│   ├── genre_rules.php
│   └── genre_patterns.php
├── db/
│   └── audious.sql             # Schéma complet
├── public/
│   ├── api/                    # Endpoints JSON (search, likes, auth, etc.)
│   ├── assets/
│   │   └── images/
│   ├── css/
│   │   └── base.css
│   └── index.php               # Frontend principal
├── sass/
│   ├── _base.scss              # Thème Cyber (footer, navbar, etc.)
│   └── admin.scss              # WIP backoffice léger
├── scan/
│   ├── import.php              # Script d’import audio
├── src/
│   └── mail.php                # Helper mail (MAIL_DRIVER)
├── logs/
└── vendor/
```

> Certains scripts additionnels (d’exemples ou d’outillage) peuvent exister dans `scan/` ou ailleurs, mais ne sont pas nécessaires au fonctionnement de base d’Audious.

---

## 🧩 **Commandes CLI intégrées**

Toutes les commandes passent par `bin/audious-cli.php` :

```bash
php bin/audious-cli.php help
```

Exemples courants :

| Commande | Description |
|----------|-------------|
| `php bin/audious-cli.php db:backup [--schema-only]` | Sauvegarde la base (structure ou complète) |
| `php bin/audious-cli.php db:purge [--tables=...] [--all] [--yes] [--force]` | Purge sélective ou complète de la base |
| `php bin/audious-cli.php tags:check [--json]` | Vérifie l’intégrité des tags |
| `php bin/audious-cli.php tags:rebuild [--yes] [--force]` | Reconstruit `tags`/`song_tags` à partir des genres |
| `php bin/audious-cli.php user:create --email=... [--username=...] [--password=...] [--admin] [--verbose] [--dry-run|--force]` | Crée un utilisateur (optionnellement admin) |
| `php bin/audious-cli.php files:dedupe --path=/home/ubuntu/music [...]` | Détection de doublons de fichiers (SHA-256) |
| `php bin/audious-cli.php artworks:cleanup [--delete] [--force] [--yes]` | Liste ou supprime les pochettes orphelines |

> La liste exacte peut évoluer, se référer à `php bin/audious-cli.php help` pour la version à jour.

---

## 🧪 **Sécurité & modes d’exécution**

Beaucoup de commandes CLI acceptent des flags communs :

| Option      | Effet |
|------------|-------|
| `--dry-run`| Simule sans rien modifier |
| `--force`  | Désactive certains garde-fous (mais pas tous) |
| `--yes`    | Valide automatiquement les confirmations interactives |
| `--verbose`| Affiche plus de détails dans la sortie ou les logs |
| `--json`   | Sortie JSON (intégration avec d’autres outils) |

---

## ⚠️ **Avertissement légal (contenus tiers)**

- Audious est conçu pour gérer et lire des fichiers audio dont **vous possédez les droits** (créations personnelles, contenus libres de droits, catalogue licencié, etc.).  
- Le projet ne fournit aucune garantie quant à la conformité de vos usages avec :
  - les **CGU** de plateformes tierces (YouTube, Spotify, etc.) ;
  - le **droit d’auteur** et les législations applicables dans votre pays.

En utilisant Audious, **vous êtes seul responsable** :

- de la provenance des fichiers importés ;
- du respect des licences et droits associés aux œuvres ;
- de la diffusion éventuelle de ces contenus (usage strictement privé vs. public).

---

## 🔮 **Roadmap**

- [ ] Backoffice léger complet pour gérer tags / genres / users
- [ ] PWA (mode offline + contrôles sur écran verrouillé)
- [ ] Lecture collaborative / “radio” synchrone
- [ ] Export / import de playlists, likes, historiques
- [ ] Module d’administration Icecast depuis l’interface
- [ ] Pack “installable” (CodeCanyon / zip prêt à déployer)

---


## 🧾 Licence & Droits

- **Documentation** : publique (vous pouvez la lire et la partager).
- **Code source Audious** : **propriétaire** (dépôt privé). Toute reproduction, redistribution ou mise à disposition du code est interdite sans autorisation écrite.
- **Marque & identité** : “Audious” et ses éléments visuels restent la propriété de leur auteur.

📩 Pour une licence, une démo, ou un accès partenaire : contactez-moi.


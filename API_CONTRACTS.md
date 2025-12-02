# 📐 API_CONTRACTS.md

Objectif : décrire les **champs minimum** attendus par le front (`app.js`) pour chaque endpoint,
afin de pouvoir refactorer le code PHP sans casser l’UI.

> Base API : `https://audious.dev/api/`

---

## `GET /search.php`

### Succès (liste)

```json
{
  "status": "success",
  "results": [
    {
      "song_id": 123,
      "song_title": "…",
      "song_duration": 312,
      "song_file": "…",
      "song_picture": "…",
      "album_name": "…",
      "artist_name": "…",
      "total_likes": 0,
      "liked": 0
    }
  ],
  "offset": 0,
  "limit": 20
}
```

**Attendu côté front**

- `results` doit être un tableau.
- Champs indispensables pour lecture + rendu :
  - `song_id`
  - `song_file`
  - `song_title`
- Likes :
  - `liked` doit exister (0/1). Le front le normalise en `user_likes`.
  - `total_likes` doit être numérique (sinon le front forcera à `0`).
- Tags :
  - si `withTags=1` dans la requête, chaque item peut contenir `tags: [{id,name,slug}]`.
  - sinon `tags` peut être absent (mode perf).

### Succès (par ID)

Quand `songId=123` est fourni, l’API renvoie un tableau `results` contenant **0 ou 1** élément.

### Erreurs

- `401` si scope protégé sans session (`likes`, `myrecent`, `playlists` si activé).
- `500` si erreur DB (le front affiche un message générique).

---

## `GET /tags_suggestions.php`

```json
{
  "tags": [
    { "id": 1, "name": "Techno", "slug": "techno" }
  ]
}
```

**Attendu côté front**

- Le front lit `j.tags` directement.
- Chaque tag doit au moins avoir `name` ou `slug`.

---

## `GET /playlists.php?scope=mine` (auth)

```json
{
  "status": "success",
  "data": {
    "playlists": [
      {
        "id": 12,
        "name": "Ma playlist",
        "track_count": 10,
        "is_public": 0,
        "cover_path": "assets/images/default.webp"
      }
    ]
  }
}
```

**Attendu côté front**

- Chemin utilisé : `data.playlists[]`.
- Champs utilisés :
  - `id`
  - `name`
  - `track_count`
  - `is_public` (0/1)
  - `cover_path` (peut être `null` → fallback côté front).

---

## `GET /playlist_songs.php?playlist_id=…` (auth)

```json
{
  "status": "success",
  "results": [
    {
      "song_id": 123,
      "song_title": "…",
      "song_file": "…",
      "song_picture": "…",
      "artist_name": "…",
      "total_likes": 0,
      "liked": 0
    }
  ]
}
```

**Attendu côté front**

- Même “shape” de morceau que `search.php` (au minimum les champs utilisés pour render/play).
- `results` doit être un tableau.

---

## `POST /likesong.php` (auth + CSRF)

### Body (JSON)

```json
{ "song_id": 123 }
```

### Réponse succès

```json
{ "status": "success", "liked": true, "total_likes": 5 }
```

**Attendu côté front**

- `status` doit valoir `"success"` pour être traité comme OK.
- `liked` (bool) : indique si le morceau est désormais liké.
- `total_likes` (int) : nouveau compteur, utilisé pour mettre à jour l’UI.

---

## `POST /playlist_create.php` (auth + CSRF)

### Body (JSON)

```json
{ "name": "Ma playlist", "is_public": 1 }
```

### Réponse succès (minimum)

```json
{ "status": "success", "id": 12 }
```

**Attendu côté front**

- `status="success"`.
- `id` (int) : identifiant de la nouvelle playlist.
- Après succès, le front invalide son cache local et refetch la liste.

---

## `POST /playlist_addsong.php` (auth + CSRF)

### Body (JSON)

```json
{ "playlist_id": 12, "song_id": 123 }
```

### Réponse succès (minimum)

```json
{ "status": "success", "already_in_playlist": false }
```

**Attendu côté front**

- `status="success"`.
- `already_in_playlist` (bool, optionnel mais géré) :
  - `true` → affiche un toast “Déjà dans la playlist”.
  - `false` → affiche un toast “Ajouté à la playlist ✅”.

---

## `GET /me.php`

Le front accepte plusieurs formats mais s’attend **au minimum** à :

- `status: "success"` si connecté.
- Un identifiant utilisateur > 0, via :
  - soit `user_id`
  - soit `user.id`

Champs utilisés dans l’UI :

- `username` (ou `user.username`)
- `email` (ou `user.email`)
- `role` optionnel.

---

## `GET /csrf.php`

Le front est tolérant sur le nom de champ ; il essaie dans cet ordre :

- `csrf`
- `csrf_token`
- `csrfToken`
- `token`
- `data.csrf`
- `data.csrf_token`

**Recommandation** côté serveur : renvoyer un JSON du type :

```json
{ "csrf": "abcdef123456…" }
```

---

## `POST /logout.php` (auth + CSRF)

Body vide ou minimal, mais :

- Utilise le header `X-CSRF-TOKEN`.
- Le front ne lit pas de payload spécifique ; il se contente de :
  - appeler `/logout.php`
  - puis de réinitialiser l’UI (badge utilisateur → bouton “Se connecter”).

---

## Résumé rapide (champs clés)

- **Morceau (song)** : `song_id`, `song_title`, `song_file`, `song_picture?`, `artist_name?`, `album_name?`, `song_duration?`, `total_likes`, `liked`.
- **Playlist** : `id`, `name`, `track_count`, `is_public`, `cover_path?`.
- **Tag** : `id?`, `name`, `slug`.
- **Like toggle** : `status`, `liked`, `total_likes`.
- **Profil** : `status`, `user_id` ou `user.id`, + `username`/`email`.
- **CSRF** : au moins un champ parmi `csrf`, `csrf_token`, `csrfToken`, `token`, `data.csrf`, `data.csrf_token`.

Ce document sert de référence rapide pour vérifier, après un refactor PHP ou une migration MySQL, que les JSON retournés restent compatibles avec le front actuel.

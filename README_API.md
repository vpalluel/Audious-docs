# 🔌 Audious — `README_API.md`

Aperçu rapide (et pragmatique) des endpoints **utilisés par le front actuel** d’Audious, avec exemples `curl`.

> Notes
> - Tous les endpoints sont sous `public/api/`.
> - Auth = session cookie (via login). Avec `curl`, utilise un cookie jar (`-c cookies.txt -b cookies.txt`).
> - Les endpoints mutateurs (`POST`) utilisent un **token CSRF** via header `X-CSRF-TOKEN` (voir section CSRF).

---

## ⚙️ Préparation

```bash
BASE="https://audious.dev/api"
HDR_JSON="-H Accept:application/json"
COOKIE="-c cookies.txt -b cookies.txt"
```

---

## 🔐 Auth / Session

### `GET /me.php`
Retourne l’utilisateur courant (ou non connecté).

```bash
curl -sS $HDR_JSON $COOKIE "$BASE/me.php" | jq .
```

### `POST /logout.php`
```bash
# nécessite CSRF (voir section CSRF)
curl -sS $HDR_JSON $COOKIE -X POST -H "X-CSRF-TOKEN: $CSRF" "$BASE/logout.php" | jq .
```

> Le login n’est pas détaillé ici (selon ton implémentation), mais l’idée reste la même : récupérer une session cookie dans `cookies.txt`.

---

## 🛡️ CSRF

### `GET /csrf.php`
Récupérer un token et l’utiliser ensuite dans `X-CSRF-TOKEN`.

```bash
CSRF=$(curl -sS $HDR_JSON $COOKIE "$BASE/csrf.php" | jq -r '.csrf // .csrf_token // .csrfToken // .token // .data.csrf // .data.csrf_token // empty')
echo "CSRF=$CSRF"
```

---

## 🔎 Recherche & navigation audio

### `GET /search.php`

Paramètres principaux :
- `q` texte (optionnel)
- `scope=all|likes|recent|myrecent|popular` *(+ `playlists` disponible : voir annexe de CURL_SEARCH.md)*
- `offset`, `limit`
- `tag=...` (CSV) ou `tag[]=...` (array), `tag_mode=any|all`
- `ft=1|0` (force FULLTEXT / LIKE)
- `shuffle=1` + `seed=<int>` (seed optionnel si ton API le génère)
- `songId=<int>` (mode “get by id”)
- `withTags=1` pour inclure `tags[]` dans chaque résultat (par défaut souvent OFF)

Exemples :

```bash
# Liste simple
curl -sS -G $HDR_JSON "$BASE/search.php" --data "scope=all" --data "limit=20" | jq .

# Recherche texte
curl -sS -G $HDR_JSON "$BASE/search.php" --data-urlencode "q=kas:st" | jq .

# Filtre tags
curl -sS -G $HDR_JSON "$BASE/search.php" --data "tag=techno,acid" --data "tag_mode=any" | jq .

# Par ID (avec tags)
curl -sS $HDR_JSON "$BASE/search.php?songId=123&withTags=1" | jq .

# Shuffle stable
curl -sS -G $HDR_JSON "$BASE/search.php" --data "shuffle=1" --data "seed=42" --data "limit=30" | jq .
```

### `GET /audio.php`
Joue un fichier à partir d’un `song_file` (retourné par `search.php` / `playlist_songs.php`).

```bash
curl -I "$BASE/audio.php?file=$(python3 - <<'PY'
import urllib.parse
print(urllib.parse.quote("example.mp3"))
PY
)"
```

*(en pratique, le client utilise `<audio src="...">` / `new Audio(...)` plutôt que `curl`.)*

### `POST /track_play.php`
Log une lecture (front envoie `application/x-www-form-urlencoded`).

```bash
curl -sS $HDR_JSON $COOKIE -X POST   -H "Content-Type: application/x-www-form-urlencoded"   --data "songId=123"   "$BASE/track_play.php" | jq .
```

---

## 🏷️ Tags

### `GET /tags_suggestions.php`
Utilisé pour les “pills” de tags (top/random).

Params vus côté front : `mode=top|random`, `limit`, `pool`, `q` (optionnel).

```bash
curl -sS -G $HDR_JSON "$BASE/tags_suggestions.php" --data "mode=top" --data "limit=18" --data "pool=200" | jq .
```

---

## ❤️ Likes

### `POST /likesong.php`
Toggle like (JSON body) + CSRF.

```bash
curl -sS $HDR_JSON $COOKIE -X POST   -H "Content-Type: application/json"   -H "X-CSRF-TOKEN: $CSRF"   --data '{"song_id":123}'   "$BASE/likesong.php" | jq .
```

Réponses attendues côté front :
- `status=success`
- `liked` (bool)
- `total_likes` (int)

---

## 📂 Playlists (liste + contenu + ajout)

### `GET /playlists.php`
Le front utilise `scope=mine` (session requise).

```bash
curl -sS -G $HDR_JSON $COOKIE "$BASE/playlists.php" --data "scope=mine" --data "limit=20" --data "offset=0" | jq .
```

Le front s’attend à quelque chose comme :
- `status=success`
- `data.playlists[]`

### `GET /playlist_songs.php`
Paramètres : `playlist_id`, `offset`, `limit`, plus `q` et `tag` (optionnels) côté front.

```bash
curl -sS -G $HDR_JSON $COOKIE "$BASE/playlist_songs.php"   --data "playlist_id=12" --data "limit=20" --data "offset=0" | jq .
```

### `POST /playlist_create.php`
Body JSON : `{ "name": "...", "is_public": 0|1 }` + CSRF.

```bash
curl -sS $HDR_JSON $COOKIE -X POST   -H "Content-Type: application/json"   -H "X-CSRF-TOKEN: $CSRF"   --data '{"name":"Ma playlist","is_public":1}'   "$BASE/playlist_create.php" | jq .
```

### `POST /playlist_addsong.php`
Body JSON : `{ "playlist_id": <int>, "song_id": <int> }` + CSRF.

```bash
curl -sS $HDR_JSON $COOKIE -X POST   -H "Content-Type: application/json"   -H "X-CSRF-TOKEN: $CSRF"   --data '{"playlist_id":12,"song_id":123}'   "$BASE/playlist_addsong.php" | jq .
```

Le front gère aussi :
- `already_in_playlist` (bool) si présent dans la réponse.

---

## 👤 Profil

### `GET /profile.php`
Retourne `user` + `stats` (utilisé par la vue Profil).

```bash
curl -sS $HDR_JSON $COOKIE "$BASE/profile.php" | jq .
```

---

## 📤 Upload (si activé)

### `POST /upload.php`
Exemple (champ `audio`). *(Adaptable selon ton implémentation serveur.)*

```bash
curl -sS $HDR_JSON $COOKIE -X POST   -H "X-CSRF-TOKEN: $CSRF"   -F "audio=@/path/to/track.mp3"   "$BASE/upload.php" | jq .
```

---

## ✅ Debug rapide

```bash
# Vérifier session + CSRF
curl -sS $HDR_JSON $COOKIE "$BASE/me.php" | jq .
curl -sS $HDR_JSON $COOKIE "$BASE/csrf.php" | jq .
```

---

### 🔁 Référence complémentaire
- Voir aussi `CURL_SEARCH.md` pour le détail de `search.php` (tags, ft/like, shuffle, scopes + annexe playlists).

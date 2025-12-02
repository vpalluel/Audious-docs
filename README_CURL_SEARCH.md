# 🎧 `README_CURL_SEARCH.md`

Commandes `curl` pour interagir avec **`public/api/search.php`** d’Audious : recherche texte, filtres de tags, scopes, tri, shuffle déterministe, etc.

> ℹ️ Rappels clés (spécifiques à ton API)
> - Paramètre de **portée** : `scope=all|likes|recent|myrecent|popular` (par défaut `all`).  
>   *(Le scope `playlists` est documenté en annexe, car le front Playlists utilise d’autres endpoints.)*
> - **Auth requise** (session) pour `likes`, `myrecent` *(et `playlists`)* → sinon `401`.
> - **Recherche** : FULLTEXT auto si `q` ≥ 2 caractères (ou force via `ft=1|0`), fallback automatique vers LIKE si erreur FT ou 0 résultat.
> - **Tags (filtre)** : `tag=…`, `tags=…` (CSV) ou `tag[]=` (tableau). Supporte `id:42` et les slugs (comparaison sur `tags.slug` ou `tags.name`). `tag_mode=any|all` (défaut `any`).
> - **Tags (payload)** : par défaut l’API peut **ne pas embarquer les tags** par morceau. Utilise `withTags=1` pour inclure `tags[]` dans les résultats.
> - **Shuffle** : `shuffle=1` active un ordre pseudo-aléatoire **stable**. Si `seed` est absent, l’API peut en **générer un** et le renvoyer dans la réponse.
> - **Limits** : `limit` ∈ [1,100], `offset` ≥ 0.

---

## ⚙️ Préparation

```bash
BASE="https://audious.one/api/search.php"
HDR="-H Accept:application/json"
```

---

## 1️⃣ Récupérer un morceau par ID

Sans tags (léger) :

```bash
curl -sS $HDR "$BASE?songId=123" | jq .
```

Avec tags :

```bash
curl -sS $HDR "$BASE?songId=123&withTags=1" | jq .
```

---

## 2️⃣ Recherche texte simple (+ pagination)

```bash
curl -sS -G $HDR "$BASE" \
  --data-urlencode "q=kas:st" \
  --data "limit=20" \
  --data "offset=0" | jq .
```

---

## 3️⃣ Contrôle du moteur de recherche (FULLTEXT vs LIKE)

- Forcer FULLTEXT :

```bash
curl -sS -G $HDR "$BASE" --data-urlencode "q=ambient" --data "ft=1" | jq .
```

- Forcer LIKE (désactiver FULLTEXT) :

```bash
curl -sS -G $HDR "$BASE" --data-urlencode "q=ambient" --data "ft=0" | jq .
```

> Si `ft` n’est pas fourni : FULLTEXT auto si `len(q) ≥ 2`, sinon LIKE.  
> Si FULLTEXT échoue (index absent) ou retourne 0 résultat, l’API retombe sur LIKE automatiquement.

---

## 4️⃣ Filtrer par tags (côté requête)

**CSV (slugs)** :

```bash
curl -sS -G $HDR "$BASE" --data "tag=techno,deep-house" | jq .
```

**Tableau** :

```bash
curl -sS -G $HDR "$BASE" \
  --data-urlencode "tag[]=techno" \
  --data-urlencode "tag[]=deep-house" | jq .
```

**Mélanger slug et ID** :

```bash
curl -sS -G $HDR "$BASE" --data "tag=id:42,techno" | jq .
```

**Mode de match** (`any` par défaut) :

```bash
curl -sS -G $HDR "$BASE" \
  --data "tag=techno,deep-house" \
  --data "tag_mode=all" | jq .
```

---

## 5️⃣ Scopes (sections fonctionnelles)

### 🔹 Tous (par défaut)

```bash
curl -sS -G $HDR "$BASE" --data "scope=all" --data "limit=20" | jq .
```

### 🔹 Récents

```bash
curl -sS -G $HDR "$BASE" --data "scope=recent" --data "limit=20" | jq .
```

### 🔹 Populaires (likes décroissants)

```bash
curl -sS -G $HDR "$BASE" --data "scope=popular" --data "limit=20" | jq .
```

### 🔹 Mes likes *(auth requise)*

```bash
curl -sS -G $HDR "$BASE" --data "scope=likes" | jq .
```

### 🔹 Mes récents *(auth requise)*

```bash
curl -sS -G $HDR "$BASE" --data "scope=myrecent" | jq .
```

> 🔐 Pour les scopes protégés (`likes`, `myrecent`, `playlists`), ta session doit être active (cookie PHP).  
> Exemple avec cookies après un login préalable :  
> `curl -sS -b cookies.txt -c cookies.txt -G $HDR "$BASE" --data "scope=likes"`

---

## 6️⃣ `withTags` (payload tags par morceau)

Par défaut (souvent OFF) :

```bash
curl -sS -G $HDR "$BASE" --data "q=techno" | jq .
```

Inclure `tags[]` pour chaque morceau :

```bash
curl -sS -G $HDR "$BASE" --data "q=techno" --data "withTags=1" | jq .
```

---

## 7️⃣ Shuffle déterministe (`shuffle=1` + `seed`)

Avec seed fourni :

```bash
curl -sS -G $HDR "$BASE" \
  --data "shuffle=1" \
  --data "seed=123456" \
  --data "limit=30" | jq .
```

Seed auto (si ton API le fait) :

```bash
curl -sS -G $HDR "$BASE" --data "shuffle=1" --data "limit=30" | jq .
```

> Le `seed` renvoyé dans la réponse permet de rejouer exactement le même ordre.

---

## 8️⃣ Pagination manuelle

```bash
# Page 1
curl -sS -G $HDR "$BASE" --data "limit=20" --data "offset=0" | jq .

# Page 2
curl -sS -G $HDR "$BASE" --data "limit=20" --data "offset=20" | jq .
```

---

## 9️⃣ Combinaisons avancées

```bash
curl -sS -G $HDR "$BASE" \
  --data-urlencode "q=live" \
  --data "tag=techno,id:7" \
  --data "tag_mode=all" \
  --data "scope=popular" \
  --data "shuffle=1" \
  --data "seed=98765" \
  --data "withTags=1" \
  --data "limit=25" \
  --data "offset=50" | jq .
```

---

## 🧠 Bonus : helper Bash

```bash
audious() {
  curl -sS -G -H 'Accept: application/json' "$BASE" "$@" | jq .
}

audious --data-urlencode q='kas:st' --data limit=20
audious --data tag=techno,deep-house --data tag_mode=all
audious --data scope=popular --data limit=20
audious --data shuffle=1 --data seed=42 --data limit=50
audious --data-urlencode q='ambient' --data ft=0
audious --data-urlencode q='techno' --data withTags=1
```

---

## 📎 Annexe — `scope=playlists` (optionnel / legacy)

> `scope=playlists` est **supporté par `search.php`**, mais le front “Playlists” d’Audious **n’utilise pas `search.php`** (il passe par `api/playlists.php` et `api/playlist_songs.php`).  
> Ce scope reste utile si tu veux une vue “morceaux présents dans au moins une de mes playlists” directement via `search.php`.

### 🔹 Morceaux présents dans mes playlists *(auth requise)*

```bash
curl -sS -b cookies.txt -c cookies.txt -G $HDR "$BASE" \
  --data "scope=playlists" \
  --data "limit=20" | jq .
```

### 🔹 Recherche texte limitée à mes playlists *(auth requise)*

```bash
curl -sS -b cookies.txt -c cookies.txt -G $HDR "$BASE" \
  --data-urlencode "q=live" \
  --data "scope=playlists" | jq .
```

### 🔹 Filtrer par tags + playlists *(auth requise)*

```bash
curl -sS -b cookies.txt -c cookies.txt -G $HDR "$BASE" \
  --data "scope=playlists" \
  --data "tag=techno,acid" \
  --data "tag_mode=any" | jq .
```

---

> 💡 Installe `jq` pour une sortie lisible : `sudo apt install jq`

# 🎧 `CURL_SEARCH.md`

Commandes `curl` pour interagir avec **`public/api/search.php`** d’Audious : recherche texte, filtres de tags, scopes (popular/recent/likes/playlists/myrecent), tri, shuffle déterministe, etc.

> ℹ️ Rappels clés (spécifiques à ton API)  
> - Paramètre de **portée** : `scope=all|likes|playlists|recent|myrecent|popular` (par défaut `all`).  
> - **Auth requise** (session) pour `likes`, `playlists`, `myrecent` → sinon `401`.  
> - **Recherche** : FULLTEXT auto si `q` ≥ 2 caractères (ou force via `ft=1|0`), fallback automatique vers LIKE si 0 résultat.  
> - **Tags** : `tag=…`, `tags=…` (CSV) ou `tag[]=` (tableau). Supporte `id:42` et les *slugs*. `tag_mode=any|all` (défaut `any`).  
> - **Shuffle** : actif uniquement si `shuffle=1` **et** `seed` fourni (entier).  
> - **Limits** : `limit` ∈ [1,100], `offset` ≥ 0.

---

## ⚙️ Préparation

```bash
BASE="https://audious.one/api/search.php"
HDR="-H Accept:application/json"
```

---

## 1️⃣ Récupérer un morceau par ID

```bash
curl -sS $HDR "$BASE?songId=123" | jq .
```

Retourne 1 enregistrement (avec `tags[]`, `liked`, `total_likes`, etc.).

---

## 2️⃣ Recherche texte simple (+ pagination)

```bash
curl -sS -G $HDR "$BASE"   --data-urlencode "q=kas:st"   --data "limit=20"   --data "offset=0" | jq .
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
> S’il n’y a **aucun résultat** en FULLTEXT, l’API **retombe** sur LIKE automatiquement.

---

## 4️⃣ Filtrer par tags

**CSV (slugs)** :
```bash
curl -sS -G $HDR "$BASE" --data "tag=techno,deep-house" | jq .
```

**Tableau** :
```bash
curl -sS -G $HDR "$BASE"   --data-urlencode "tag[]=techno"   --data-urlencode "tag[]=deep-house" | jq .
```

**Mélanger slug et ID** :
```bash
curl -sS -G $HDR "$BASE" --data "tag=id:42,techno" | jq .
```

**Mode de match** (`any` par défaut) :
```bash
curl -sS -G $HDR "$BASE"   --data "tag=techno,deep-house"   --data "tag_mode=all" | jq .
```

---

## 5️⃣ Scopes (sections fonctionnelles)

### 🔹 Tous (par défaut)
```bash
curl -sS -G $HDR "$BASE" --data "scope=all" --data "limit=20" | jq .
```

### 🔹 Récents (ajouts récents → `s.id DESC`)
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

### 🔹 Mes playlists *(auth requise)*
```bash
curl -sS -G $HDR "$BASE" --data "scope=playlists" | jq .
```

### 🔹 Mes récents *(auth requise)*
```bash
curl -sS -G $HDR "$BASE" --data "scope=myrecent" | jq .
```

> 🔐 Pour les scopes protégés (`likes`, `playlists`, `myrecent`), **ta session doit être active** (cookie PHP).  
> Exemple d’utilisation de cookies avec `curl` après un login préalable :  
> `curl -sS -b cookies.txt -c cookies.txt -G $HDR "$BASE" --data "scope=likes"`

---

## 6️⃣ Shuffle déterministe (avec `seed`)

```bash
curl -sS -G $HDR "$BASE"   --data "shuffle=1"   --data "seed=123456"   --data "limit=30" | jq .
```

> ⚠️ Si `seed` est **absent**, le `shuffle` est ignoré (ordre par défaut selon `scope`/recherche).

---

## 7️⃣ Pagination manuelle

```bash
# Page 1
curl -sS -G $HDR "$BASE" --data "limit=20" --data "offset=0" | jq .

# Page 2
curl -sS -G $HDR "$BASE" --data "limit=20" --data "offset=20" | jq .
```

---

## 8️⃣ Combinaisons avancées

```bash
curl -sS -G $HDR "$BASE"   --data-urlencode "q=live"   --data "tag=techno,id:7"   --data "tag_mode=all"   --data "scope=popular"   --data "shuffle=1"   --data "seed=98765"   --data "limit=25"   --data "offset=50" | jq .
```

---

## 🧠 Bonus : helper Bash

```bash
audious() {
  curl -sS -G -H 'Accept: application/json' "$BASE" "$@" | jq .
}

# Exemples
audious --data-urlencode q='kas:st' --data limit=20
audious --data tag=techno,deep-house --data tag_mode=all
audious --data scope=popular --data limit=20
audious --data shuffle=1 --data seed=42 --data limit=50
audious --data-urlencode q='ambient' --data ft=0
```

---

> 💡 Installe `jq` pour une sortie lisible : `sudo apt install jq`.

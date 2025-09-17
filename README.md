# MovieWatch — Django REST + Vue.js

A small full-stack app to **search movies**, **like/unlike** titles, and **write/read reviews**.  
Backend is **Django + Django REST Framework**; frontend is **Vue 3 + Axios** using the **OMDb API** for movie data.

---

## ✨ Features

- **Search** movies by title via OMDb (API key in `src/env.js`).
- **Like / Unlike** movies (persisted in Django DB).
- **Liked Movies** page shows everything you’ve liked.
- **Movie Detail** page with **reviews** (create & list).
- Clean JSON API, easy to integrate with other UIs.

---

## 🧱 Tech Stack

- **Backend:** Django, Django REST Framework (DRF)
- **Frontend:** Vue 3 (Composition API), Axios, Vue Router
- **DB:** SQLite (default) — swap to Postgres/MySQL as needed
- **External:** OMDb API (https://www.omdbapi.com/)

---

## 🚀 Quickstart

### Backend (Django + DRF)

**Prereqs:** Python 3.10+ recommended

```bash
cd backend
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

pip install django djangorestframework django-cors-headers
python manage.py migrate
python manage.py runserver 8000
```

**Django settings changes (summary):**
- `INSTALLED_APPS`: add `'rest_framework'`, `'corsheaders'`, and your app (e.g. `'movies'`).
- `MIDDLEWARE`: add `'corsheaders.middleware.CorsMiddleware'` **at the top**.
- For local dev, either:
  - `CORS_ALLOW_ALL_ORIGINS = True` (dev only), or
  - `CORS_ALLOWED_ORIGINS = ["http://localhost:5173", "http://localhost:8080"]` (depending on your Vue dev port).
- If you use SessionAuth/CSRF, configure CSRF properly for Axios (or exempt views for dev only).

### 2) Frontend (Vue)

**Prereqs:** Node 18+ recommended

```bash
cd frontend
npm install
```

Create `src/env.js`:

```js
// src/env.js
export default {
  apikey: "YOUR_OMDB_API_KEY"
};
```

Run the dev server (Vue CLI: `npm run serve`, Vite: `npm run dev`):

```bash
npm run dev
```

By default, the frontend code expects the API at `http://localhost:8000/`.

---

## 🔌 API Reference (current code)

Base URL: `http://localhost:8000/api/v1/`

> **Models**
>
> - `LikedMovie`: `{ movie_id: string (unique), title: string, poster_url: string }`
> - `Review`: `{ id, movie_id: string, text: string, created_at: datetime }`

### Likes

- **List liked movies**
  - `GET /api/v1/likes/`
  - **Response:** `200 OK` → `[{ movie_id, title, poster_url }, ...]`

- **Like a movie** *(function view)*  
  - `POST /api/v1/likes/`  
  - **Body (JSON):**
    ```json
    { "movie_id": "tt0848228", "title": "The Avengers", "poster_url": "https://..." }
    ```
  - **Response:** `200 OK` with `{"status":"liked"}` or `{"status":"already liked"}`

- **Unlike a movie** *(function view)*  
  - `DELETE /api/v1/unlike/<movie_id>/`
  - **Response:** `200 OK` with `{"status":"unliked"}` (or `404` if not found)


### Reviews

- **List reviews for a movie**
  - The code maps: `GET /api/v1/reviews/<movie_id>/`
  - However, `get_queryset` currently reads **query params**, not path kwargs.
  - **Work now (as coded):** `GET /api/v1/reviews/<movie_id>/` returns **all** reviews.
  - **Intended:** filter by `movie_id`. See fix below.

- **Create a review**
  - `POST /api/v1/postreviews/`
  - **Body (JSON):**
    ```json
    { "movie_id": "tt0848228", "text": "Loved the team-up!" }
    ```
  - **Response:** `201 Created` (from DRF generic) with created review JSON *(frontend expects 201)*

---

## 🖥️ Frontend Behavior (Vue)

- **Home.vue**
  - Search OMDb: `GET http://www.omdbapi.com/?apikey=${env.apikey}&s=${search}`
  - Like/Unlike via:
    - `POST http://localhost:8000/api/v1/likes/` (expects `201 Created`)
    - `DELETE http://localhost:8000/api/v1/unlike/${imdbID}/` (expects `204 No Content`)
  - Tracks liked state in a local `likedMovies` array.

- **LikedMovies.vue**
  - Fetches liked movies via `GET /api/v1/likes/` and renders cards.

- **MovieDetail.vue**
  - Fetches movie details via OMDb by `i=<id>`.
  - Lists reviews via `GET /api/v1/reviews/<id>` *(see fix)*.
  - Creates reviews via `POST /api/v1/postreviews/`.

---

## 🧪 Example cURL

```bash
# Like a movie
curl -X POST http://localhost:8000/api/v1/likes/ \
  -H "Content-Type: application/json" \
  -d '{"movie_id":"tt0848228","title":"The Avengers","poster_url":"https://..."}'

# List liked movies
curl http://localhost:8000/api/v1/likes/

# Unlike a movie
curl -X DELETE http://localhost:8000/api/v1/unlike/tt0848228/

# Post a review
curl -X POST http://localhost:8000/api/v1/postreviews/ \
  -H "Content-Type: application/json" \
  -d '{"movie_id":"tt0848228","text":"Loved the team-up!"}'

# (Intended) list reviews for a movie (see fixes)
curl http://localhost:8000/api/v1/reviews/tt0848228/
```

---


- OMDb API for public movie data.
- Django REST Framework for rapid API scaffolding.

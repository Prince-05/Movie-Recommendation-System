# 🎬 Movie Recommender

A full-stack movie recommendation web app powered by **TF-IDF content-based filtering** + **TMDB API**, built with FastAPI and Streamlit — deployed on [Render](https://render.com).


🔗 **Live Demo:** [https://movie-recommendation-system-v5nfhvg6cm2szgom3f7ytc.streamlit.app/](https://movie-recommendation-system-v5nfhvg6cm2szgom3f7ytc.streamlit.app/)
---

## ✨ Features

- 🔍 **Keyword Search** — Search movies by title with real-time TMDB suggestions and a dropdown autocomplete
- 🖼️ **Poster Grid** — Browse trending, popular, top-rated, now playing, and upcoming movies on the home feed
- 📄 **Movie Details** — View poster, backdrop, genres, release date, and overview for any movie
- 🤖 **TF-IDF Recommendations** — Content-based similar movies computed from a local dataset using cosine similarity on TF-IDF vectors
- 🎭 **Genre Recommendations** — TMDB Discover API results filtered by the movie's primary genre
- ⚡ **Fast & Cached** — Short-TTL Streamlit caching on API calls; async HTTPX on the backend

---

## 🏗️ Architecture

```
┌─────────────────────┐        HTTP        ┌──────────────────────────┐
│   Streamlit (UI)    │ ◄────────────────► │   FastAPI (Backend)      │
│   app.py            │                    │   main.py                │
└─────────────────────┘                    └──────────┬───────────────┘
                                                      │
                                         ┌────────────┴────────────┐
                                         │                         │
                                  ┌──────▼──────┐        ┌─────────▼──────┐
                                  │  TMDB API   │        │  Local Pickles │
                                  │ (posters,   │        │  df.pkl        │
                                  │  search,    │        │  tfidf_matrix  │
                                  │  discover)  │        │  indices.pkl   │
                                  └─────────────┘        └────────────────┘
```

**Frontend:** Streamlit single-file app with session-state routing (home ↔ details), URL query params, and a responsive poster grid.

**Backend:** FastAPI with async HTTPX for TMDB calls, pickle-loaded TF-IDF resources at startup, and CORS enabled for local Streamlit.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | [Streamlit](https://streamlit.io) |
| Backend | [FastAPI](https://fastapi.tiangolo.com) |
| HTTP Client | [HTTPX](https://www.python-httpx.org) (async) |
| Recommendations | Scikit-learn TF-IDF + Cosine Similarity |
| Movie Data | [TMDB API](https://developer.themoviedb.org/docs) |
| Deployment | [Render](https://render.com) |

---

## 📁 Project Structure

```
movie-recommender/
├── app.py               # Streamlit frontend
├── main.py              # FastAPI backend
├── df.pkl               # Processed movie DataFrame
├── indices.pkl          # Title → matrix index mapping
├── tfidf_matrix.pkl     # Sparse TF-IDF matrix (scipy)
├── tfidf.pkl            # Fitted TF-IDF vectorizer
├── .env                 # TMDB_API_KEY (not committed)
└── requirements.txt
```

---

## ⚙️ Setup & Installation

### 1. Clone the repo

```bash
git clone https://github.com/your-username/movie-recommender.git
cd movie-recommender
```

### 2. Install dependencies

```bash
pip install fastapi uvicorn streamlit httpx pandas numpy python-dotenv scikit-learn
```

### 3. Add your TMDB API key

Create a `.env` file in the root directory:

```env
TMDB_API_KEY=your_tmdb_api_key_here
```

> Get a free API key at [https://www.themoviedb.org/settings/api](https://www.themoviedb.org/settings/api)

### 4. Prepare the pickle files

Make sure the following files are present in the project root (generated from your dataset notebook/script):

- `df.pkl` — pandas DataFrame with at least a `title` column
- `indices.pkl` — dict or pandas Series mapping `title → row index`
- `tfidf_matrix.pkl` — scipy sparse matrix (rows = movies, cols = TF-IDF features)
- `tfidf.pkl` — fitted `TfidfVectorizer` object

### 5. Run the backend

```bash
uvicorn main:app --reload --port 8000
```

### 6. Run the frontend

In a separate terminal:

```bash
streamlit run app.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `GET` | `/home` | Home feed (trending, popular, etc.) |
| `GET` | `/tmdb/search` | TMDB keyword search (multiple results) |
| `GET` | `/movie/id/{tmdb_id}` | Movie details by TMDB ID |
| `GET` | `/recommend/genre` | Genre-based recommendations |
| `GET` | `/recommend/tfidf` | TF-IDF recommendations (title query) |
| `GET` | `/movie/search` | Bundle: details + TF-IDF + genre recs |

### Example: Search

```
GET /tmdb/search?query=inception
```

### Example: Recommendations bundle

```
GET /movie/search?query=Inception&tfidf_top_n=12&genre_limit=12
```

---

## 🤖 How Recommendations Work

### TF-IDF (Content-Based)

1. At startup, the app loads a pre-computed TF-IDF matrix from `tfidf_matrix.pkl`
2. Given a movie title, the corresponding row vector is extracted
3. Cosine similarity is computed against all other rows using a matrix multiply: `scores = tfidf_matrix @ query_vector.T`
4. The top-N results (excluding the query movie itself) are returned as similar titles
5. Each title is enriched with a poster and TMDB ID via a TMDB search call

### Genre (TMDB Discover)

1. The movie's primary genre ID is fetched from TMDB
2. TMDB's `/discover/movie` endpoint is queried with `sort_by=popularity.desc` for that genre
3. The source movie itself is filtered out from the results

---

## 🚀 Deployment on Render

Both the FastAPI backend and Streamlit frontend can be deployed as separate **Web Services** on Render.

### Backend (FastAPI)

- **Build Command:** `pip install -r requirements.txt`
- **Start Command:** `uvicorn main:app --host 0.0.0.0 --port $PORT`
- **Environment Variable:** `TMDB_API_KEY=your_key`

### Frontend (Streamlit)

- **Build Command:** `pip install -r requirements.txt`
- **Start Command:** `streamlit run app.py --server.port $PORT --server.address 0.0.0.0`
- **Environment Variable:** Update `API_BASE` in `app.py` to your Render backend URL

```python
# app.py
API_BASE = "https://your-backend.onrender.com"
```

---


## 📝 License

MIT License — feel free to fork and build on this!

---

## 🙏 Acknowledgements

- Movie data powered by [The Movie Database (TMDB)](https://www.themoviedb.org/)
- Recommendation logic inspired by classic content-based filtering approaches

> *This product uses the TMDB API but is not endorsed or certified by TMDB.*

[def]: https://movie-recommendation-system-v5nfhvg6cm2szgom3f7ytc.streamlit.app/
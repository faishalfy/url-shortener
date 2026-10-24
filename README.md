# URL Shortener

A small Flask service that turns long URLs into short links.

## What you need

- Docker Desktop with Docker Compose
- An internet connection for the first build, to download the images and Python packages

The app uses Flask, PostgreSQL to store URLs, and Redis to cache lookups. Docker Compose builds and runs all three services.

## How it works

1. Send a long URL to the Flask app.
2. The app saves it in PostgreSQL. The database ID is encoded as a short code.
3. Redis caches the short code and its URL.
4. When someone opens a short link, the app checks Redis first. If it is not cached, it looks it up in PostgreSQL and caches it.
5. The app redirects to the original URL.

## How to use it

Start the services from the project directory:

```bash
docker compose up --build -d
```

Create a short link:

```bash
curl -X POST http://localhost:5001/shorten \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com"}'
```

The response includes a `short_url` and `short_code`. Open the `short_url` in a browser to follow the redirect.

## Rate limit

Each client IP can make up to 10 requests to `/shorten`. Each accepted request refreshes a 60-second Redis counter. Once the limit is reached, further requests return HTTP `429` with `{"error":"Rate limit exceeded. Try again later."}` until the counter expires. Requests from the same IP share the limit, and invalid requests count too.

Check running services and stop them with:

```bash
docker compose ps
docker compose down
```

PostgreSQL data is kept in a Docker volume when you stop the services.

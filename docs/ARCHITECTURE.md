# PWDE Architecture

## Plain-language overview

The Data Explorer has four main pieces:

1. **Browser/front end** — the Vue/Vuetify application downloaded by a user's browser.
2. **Apache web server** — serves the front-end files over HTTPS and provides the secure `/api/` doorway.
3. **PostgREST API** — translates approved API requests into database queries.
4. **PREP PostgreSQL database** — stores the underlying data.

The CARTO basemap is a separate external service used by the browser for mapping tiles.

## Architecture diagram

```mermaid
flowchart LR
    B[User's browser]
    A[Apache / HTTPS<br/>data.prepestuaries.org]
    FE[Data Explorer<br/>/data-explorer/]
    API[/api/ reverse proxy]
    P[PostgREST<br/>internal :3001]
    DB[(PREP PostgreSQL<br/>database)]
    C[CARTO basemap service]

    B -->|HTTPS| A
    A --> FE
    B -->|HTTPS /api/...| API
    API -->|internal HTTP| P
    P --> DB
    B -->|HTTPS map tiles| C
```

## Key idea

The browser never needs to know that PostgREST is running on port 3001. Apache provides the secure public interface, while the backend service remains internal.

## September 2026 API problem

The old production bundle used:

```text
http://data.prepestuaries.org:3001
```

from an HTTPS page. Browsers blocked those requests as mixed active content.

The production bundle now uses:

```text
/api
```

which keeps the browser-to-server connection HTTPS.

## CARTO

The Data Explorer uses CARTO for its basemap. The production API key is supplied at build time and is not committed to Git.

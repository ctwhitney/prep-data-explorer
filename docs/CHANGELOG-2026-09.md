# September 2026 Change Summary

## Overview

Two production issues affecting the Piscataqua Watershed Data Explorer were addressed:

1. The CARTO basemap required updated authentication.
2. The Data Explorer API requests were blocked because the HTTPS application was requesting data over HTTP.

## CARTO fix

The basemap configuration was changed to use CARTO's authenticated raster basemap endpoint. The production CARTO key is supplied through `VITE_CARTO_API_KEY` at build time and is restricted to the PREP domain pattern.

The credential is intentionally not stored in Git.

## HTTPS/API fix

The original production bundle used:

```text
http://data.prepestuaries.org:3001
```

The Data Explorer itself is served over HTTPS. Browsers therefore blocked requests for `prep_stations`, `prep_results`, and `prep_variables` as mixed active content.

### The temporary/bandaid idea

During troubleshooting, a separate Apache proxy route for the Data Explorer API was considered. This would have provided another HTTPS URL that Apache could forward to port 3001.

Investigation showed that this was unnecessary because the active HTTPS Apache virtual host already had:

```text
/api/ → PostgREST :3001
```

### Final fix

The production API base was changed to:

```text
/api
```

The browser now uses HTTPS:

```text
https://data.prepestuaries.org/api/prep_stations
```

Apache forwards the request internally to PostgREST.

This resolved the mixed-content problem without adding another public proxy route.

## Deployment

The new Vite assets were staged first. A full application backup and an `index.html` rollback copy were retained. The new `index.html` was copied last, making it the activation point.

Old hashed assets were retained so rollback could be performed by restoring the previous `index.html`.

## Result

The Data Explorer front end, API requests, and CARTO basemap were all restored to normal production operation.

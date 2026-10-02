# SOP-01 — Infrastructure Access and Data Explorer Architecture

## Purpose

This SOP explains the infrastructure a maintainer needs to understand before rebuilding or deploying the PREP Data Explorer. It intentionally documents connection concepts and locations without publishing credentials or private keys.

## 1. Access prerequisites

A maintainer needs:

- Appropriate authorization to administer the PREP server/application.
- SSH access to the AWS/Bitnami server.
- The approved PuTTY private key (`.ppk`) stored securely outside Git.
- Permission to use `sudo` for server maintenance.
- Access to the PREP database/PostgREST service as required by the application.
- Git access to the `prep-data-explorer` repository.

Never place the `.ppk` private key, database passwords, or API keys in the repository.

## 2. Production server

The Data Explorer is hosted on the PREP Bitnami server. The production application directory is:

```text
/opt/bitnami/apps/dataexplorer/
```

The live static entry point is:

```text
/opt/bitnami/apps/dataexplorer/index.html
```

Compiled assets are in:

```text
/opt/bitnami/apps/dataexplorer/assets/
```

## 3. Apache

Before modifying Apache, identify the active virtual host:

```bash
sudo /opt/bitnami/apache2/bin/apachectl -S
```

For the current production configuration, the active PREP virtual-host configuration is:

```text
/mnt/django/PREP/conf/httpd-vhosts.conf
```

Do not assume that files under `/opt/bitnami/apache2/conf/extra/` are the active Data Explorer virtual host.

## 4. HTTPS and the API

The public site is served over HTTPS:

```text
https://data.prepestuaries.org/
```

The Data Explorer is available at:

```text
https://data.prepestuaries.org/data-explorer/
```

The browser-facing API path is:

```text
https://data.prepestuaries.org/api/
```

Apache reverse-proxies `/api/` to the internal PostgREST service on port 3001.

A basic API health check is:

```bash
curl -I https://data.prepestuaries.org/api/prep_stations
```

A successful response should be HTTP 200.

## 5. PostgREST

PostgREST provides the HTTP API used by the Data Explorer. The backend service listens on port 3001.

An internal server-side test is:

```bash
curl -I http://127.0.0.1:3001/prep_stations
```

The browser should not use this direct HTTP URL in production. Browser requests should use `/api/`.

## 6. Database relationship

Conceptually:

```text
Browser
  |
  | HTTPS /api/
  v
Apache
  |
  | internal HTTP
  v
PostgREST :3001
  |
  v
PREP PostgreSQL database
```

PostgREST exposes approved database tables/views/functions as HTTP resources. The Data Explorer consumes those API resources; it does not connect directly to PostgreSQL from browser JavaScript.

## 7. Troubleshooting

### Site loads but data does not

Check the browser console/network panel. If requests point to:

```text
http://data.prepestuaries.org:3001/
```

the production front end was built with the old API URL.

Check the live bundle and environment configuration.

### Browser reports mixed active content

The browser is probably attempting HTTP API requests from an HTTPS page. Verify that the production build uses:

```text
/api
```

and that Apache has the `/api/` reverse proxy.

### `/api/prep_stations` returns an error

Test both layers:

```bash
curl -I https://data.prepestuaries.org/api/prep_stations
curl -I http://127.0.0.1:3001/prep_stations
```

If the internal PostgREST endpoint works but `/api/` does not, investigate Apache. If both fail, investigate PostgREST/database infrastructure.

### Apache configuration changes

Always back up the relevant configuration before editing and run:

```bash
sudo /opt/bitnami/apache2/bin/apachectl -t
```

before reloading/restarting Apache.

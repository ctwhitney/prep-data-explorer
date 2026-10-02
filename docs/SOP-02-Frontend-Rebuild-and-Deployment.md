# SOP-02: Front-End Rebuild and Production Deployment

## Purpose

This procedure describes the verified process for rebuilding and
deploying the PWDE front end. It assumes approved access to the
production server. See `SOP-01-Infrastructure-Access.md` for
infrastructure notes.

## 1. Local repository

Before making changes:

``` powershell
git status
git pull
```

Do not add unrelated files to the maintenance commit. In particular, the
2022 eelgrass GeoJSON is an unrelated untracked file and should remain
untracked.

## 2. Install dependencies

Use the lockfile for a reproducible installation:

``` powershell
npm ci
```

Do not run `npm audit fix`, force dependency upgrades, or upgrade npm as
part of a routine rebuild unless there is a separate reason to do so.

## 3. Check production configuration

Production uses:

``` text
VITE_API_URL=/api
BASE_URL=/data-explorer/
```

The production CARTO API key is supplied through environment
configuration.

Do not commit `.env.production` or other files containing production
secrets.

The production browser-facing API path should be `/api`, not:

``` text
http://data.prepestuaries.org:3001
```

Development may use a different API URL.

## 4. Validate and build

Run:

``` powershell
npm run type-check
npm run build-only
```

The production files are written to:

``` text
.\dist\
```

### Verify the compiled API path

The old direct API URL must not be present:

``` powershell
Select-String -Path .\dist\assets\*.js -Pattern "data.prepestuaries.org:3001"
```

This should return no matches.

## 5. Back up production

On the server:

``` bash
cd /opt/bitnami/apps
sudo tar -czf dataexplorer-backup-$(date +%Y%m%d-%H%M%S).tar.gz dataexplorer
```

Before replacing `index.html`:

``` bash
sudo cp -p /opt/bitnami/apps/dataexplorer/index.html \
  /opt/bitnami/apps/dataexplorer/index.html.backup-YYYYMMDD-description
```

The full archive provides a rollback copy of the application directory.
The `index.html` copy provides a convenient quick rollback/reference
point.

## 6. Stage assets

Using PuTTY's `pscp.exe` from Windows PowerShell, the general form is:

``` powershell
& "C:\Program Files\PuTTY\pscp.exe" `
  -i "C:\path\to\prepserver.ppk" `
  ".\dist\assets\*" `
  "bitnami@SERVER:/opt/bitnami/apps/dataexplorer/assets/"
```

Replace the placeholder key path, server address, and other connection
details with the approved institutional values.

Do not commit private key paths, passwords, or credentials.

Upload assets before replacing `index.html`.

## 7. Verify staged assets

On the server:

``` bash
ls -lh /opt/bitnami/apps/dataexplorer/assets/
```

Asset filenames are generated with hashes and will change between
builds.

## 8. Replace `index.html` last

After all referenced assets are present, copy the new `dist/index.html`
into the production application directory.

The important sequence is:

``` text
1. Back up production
2. Upload assets
3. Verify assets
4. Replace index.html last
```

A static front-end cutover does not require an Apache restart.

## 9. Verify the live application

Check the live HTML:

``` bash
curl -s https://data.prepestuaries.org/data-explorer/ | grep -E "index-[0-9a-f]+\.js|index-[0-9a-f]+\.css"
```

Check the public API:

``` bash
curl -I https://data.prepestuaries.org/api/prep_stations
```

Then verify in a browser that:

-   the application loads
-   data queries work
-   no mixed-content API errors appear
-   the CARTO basemap loads
-   other map layers load as expected

## 10. Rollback

If the deployment causes a problem, restore the previous production
application from the verified backup archive.

Do not run rollback commands until the correct backup and current
application state have been confirmed.

If only the entry-point HTML needs to be restored and the existing
assets are known to be valid, the saved `index.html.backup-*` file can
provide a faster recovery path.

## 11. September 2026 notes

The September 2026 deployment fixed two independent issues.

### CARTO basemap authentication

The CARTO basemap configuration was changed to use the authenticated
raster endpoint. The production API key is supplied through environment
configuration and is not committed to Git.

### HTTPS/API mixed content

The production application previously used:

``` text
http://data.prepestuaries.org:3001
```

Because the Data Explorer is served over HTTPS, browsers blocked those
requests as mixed active content.

The production configuration was changed to:

``` text
VITE_API_URL=/api
```

The existing Apache `/api/` reverse proxy forwards those requests
internally to PostgREST on port 3001.

A separate API proxy route was considered during troubleshooting.
Investigation showed that the active HTTPS Apache virtual host already
contained the required `/api/` proxy, so no additional production route
was needed.

## 12. Apache changes are not part of a routine rebuild

A normal static front-end deployment does not require an Apache restart
or configuration change.

If API routing appears broken, first verify the active Apache
configuration and existing `/api/` proxy using
`SOP-01-Infrastructure-Access.md`.

Do not create a second API proxy route without first confirming that the
existing active route is unavailable or insufficient.

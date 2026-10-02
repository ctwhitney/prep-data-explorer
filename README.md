Piscataqua Watershed Data Explorer
----------------------------------

*Prepared for*: [Piscataqua Region Estuaries Partnership (PREP)](https://prepestuaries.org/)

*By*: Jeffrey D Walker, PhD ([Walker Environmental Research LLC](https://walkerenvres.com)) in collaboration with Laura Diemer, Luke Frankel ([FB Environmental](https://www.fbenvironmental.com/))

Links:
- **Live Website**: https://data.prepestuaries.org/data-explorer/
- **Development Website**: http://walkerenvres-prep.s3-website-us-east-1.amazonaws.com/
- **Original Developer Documentation**: https://github.com/walkerjeffd/prep-data-explorer/wiki

## Overview

This repository contains the source code for the Piscataqua Watershed Data Explorer (PWDE). The PWDE is a client-side web application developed using Vue.js, Vuetify, and Vite. Data are loaded from a PostgREST API connected to the PREP Database.

The primary goals of the application are to provide a user-friendly interface for exploring and downloading water-quality data throughout the PREP region.

See the [original Project Wiki](https://github.com/walkerjeffd/prep-data-explorer/wiki) for additional background on how the web application retrieves data from the PREP database and API.

## Architecture

The application has three main layers:

1. **Front end** — the Vue/Vuetify application served from `/data-explorer/`.
2. **API proxy** — Apache serves the application over HTTPS and proxies browser requests to `/api/` to the internal PostgREST service.
3. **Database API** — PostgREST provides the API used to query the PREP PostgreSQL database.

In production, the browser should communicate with the API through the HTTPS `/api/` path. The browser should **not** connect directly to PostgREST on port 3001.

A simplified architecture diagram is available in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Project Setup

### Requirements

The application was developed using Node.js 18 or higher. The repository includes a `.tool-versions` file documenting the Node.js version used by the original development environment.

Node.js can be installed from [nodejs.org](https://nodejs.org/).

### Install Dependencies

For a reproducible installation from the repository lockfile, use:

```sh
npm ci
```

### Configuration

The application uses Vite environment files to configure different environments. The primary variables are:

```text
VITE_API_URL="URL to the database API"
BASE_URL="Base URL for the application"
```

Typical production values are:

```text
VITE_API_URL=/api
BASE_URL=/data-explorer/
```

Environment files containing production credentials or API keys should not be committed to the repository. In particular, `.env.production` may contain the production CARTO API key and is intentionally excluded by `.gitignore`.

A safe template is provided in [`.env.example`](.env.example).

> **Security:** Never commit passwords, database credentials, private keys, production API keys, or other secrets to this repository.

## Development Server

Run the Vite development server with:

```sh
npm run dev
```

Then navigate to the URL provided by Vite in your browser.

Development configuration is kept separate from the production configuration. Do not change the production API path simply to accommodate local development.

## Production Build

Before deploying a production build, run the type checker:

```sh
npm run type-check
```

Then create the production build:

```sh
npm run build-only
```

The resulting files are placed in the `dist/` directory.

The repository also supports the standard build command:

```sh
npm run build
```

which runs the project's configured build workflow.

### Verifying the Production API Configuration

The production build should use the relative `/api` path rather than the direct PostgREST address.

For example, after building, the compiled JavaScript should not contain:

```text
http://data.prepestuaries.org:3001
```

The production application should instead contain requests using:

```text
/api/...
```

This allows the browser to remain entirely on HTTPS while Apache handles the internal connection to PostgREST.

## Deployment

The production application is hosted separately from this Git repository. The current production installation is served from:

```text
/opt/bitnami/apps/dataexplorer/
```

The public application is:

```text
https://data.prepestuaries.org/data-explorer/
```

The production Apache configuration provides the browser-facing `/api/` reverse proxy to the PostgREST service.

**Do not deploy by simply copying `dist/` to an arbitrary server location or by creating a new API proxy route without first checking the active Apache configuration.**

For the verified production deployment procedure, see:

- [`docs/SOP-01-Infrastructure-Access.md`](docs/SOP-01-Infrastructure-Access.md)
- [`docs/SOP-02-Frontend-Rebuild-and-Deployment.md`](docs/SOP-02-Frontend-Rebuild-and-Deployment.md)

The infrastructure access SOP is intentionally separate from the front-end rebuild SOP because server access, Apache configuration, and database/API infrastructure are distinct from the Vue application itself.

## Production Maintenance

When updating the production application:

1. Make and test source/configuration changes locally.
2. Run `npm ci` when dependencies need to be installed from the lockfile.
3. Run `npm run type-check`.
4. Run `npm run build-only`.
5. Verify the generated production assets.
6. Back up the current production deployment.
7. Stage the new assets on the production server.
8. Verify the staged files.
9. Replace the production `index.html` last so that the live HTML does not reference assets that have not yet been uploaded.
10. Test the application in a browser, including the API and map layers.

The deployment SOP documents the current procedure in more detail, including rollback considerations.

## Recent Maintenance History

### September 2026 — CARTO basemap authentication

The CARTO basemap configuration was updated to use the authenticated CARTO raster endpoint and a production API key supplied through the environment configuration.

The API key is intentionally kept out of Git.

### September 2026 — HTTPS/API path

The production application previously attempted to connect directly to:

```text
http://data.prepestuaries.org:3001
```

Because the Data Explorer is served over HTTPS, modern browsers blocked those requests as mixed active content.

The production configuration was changed to:

```text
VITE_API_URL=/api
```

The existing Apache `/api/` reverse proxy then forwards those requests internally to PostgREST on port 3001.

A separate API proxy route was considered during troubleshooting, but investigation showed that the active HTTPS Apache virtual host already provided the required `/api/` proxy. The existing route was therefore used rather than introducing another production endpoint.

See [`docs/CHANGELOG-2026-09.md`](docs/CHANGELOG-2026-09.md) for more detail.

## Documentation

Additional maintenance documentation is stored in the `docs/` directory:

- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — plain-language architecture overview and diagram.
- [`docs/SOP-01-Infrastructure-Access.md`](docs/SOP-01-Infrastructure-Access.md) — production server, Apache, and API/database infrastructure notes.
- [`docs/SOP-02-Frontend-Rebuild-and-Deployment.md`](docs/SOP-02-Frontend-Rebuild-and-Deployment.md) — front-end rebuild, staging, deployment, verification, and rollback procedure.
- [`docs/CHANGELOG-2026-09.md`](docs/CHANGELOG-2026-09.md) — record of the September 2026 production fixes and deployment changes.

Some institutional infrastructure details are still being reconstructed from historical notes and meeting/development records. Where a procedure has not yet been independently verified, the documentation should be updated rather than filled in with assumptions.

## Repository and Environment Files

Production and local environment files may contain credentials or API keys and should remain untracked.

The repository `.gitignore` excludes:

```text
.env.development
.env.production
.env.local
.env.*.local
```

The example environment file is safe to commit because it contains placeholders rather than real credentials.

Do not add production build output, credentials, or unrelated data files to a maintenance commit unless there is a specific reason to version them.

## License

This application is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). See [`LICENSE`](LICENSE) for more information.

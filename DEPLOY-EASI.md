# Deploy Human Atlas to the EASI/BASI droplet

The production static build is served at **https://human-atlas.pivotventures.tech**.

Caddy on the EASI/BASI droplet uses `file_server` from `/opt/basi/human-atlas-static`, the same pattern as `meetings-static`. Hostname, TLS, and Caddyfile changes live in Pivot-Ventures/rag-platform.

## Requirements

- Node.js **22.13** or newer
- SSH access to the BASI droplet as `root`

## Build

From the repository root:

```bash
npm ci && npm run build
```

Output is `dist/`. Vite `base` is `/`, so a dedicated hostname can serve the site at the host root.

`public/ATTRIBUTION.md` is copied into `dist/ATTRIBUTION.md` (BodyParts3D, © The Database Center for Life Science, CC BY 4.0). Keep that file in the published tree.

Optional typecheck before shipping:

```bash
npm run check
```

## Sync to the droplet

```bash
npm ci && npm run build
rsync -az --delete dist/ root@<BASI_DROPLET>:/opt/basi/human-atlas-static/
```

Replace `<BASI_DROPLET>` with the droplet hostname or IP. Trailing slashes matter: they sync the contents of `dist/` into the destination directory.

After rag-platform points Caddy at this directory, the site is available at https://human-atlas.pivotventures.tech.

## Hosting notes

- Catalogue and mesh paths are origin-root (`/models/atlas.json`, `/models/body-*.bin`). Serve this build at the host root, not a URL subpath.
- `.bin.gz` files are fetched by the app and decoded in the browser when the payload is still gzip. Serve them as regular static files.
- There is no client-side router: `file_server` of `index.html` at `/` is enough.
- Vercel remains an optional preview host via `vercel.json`. The upstream live reference is https://human-atlas-seven.vercel.app.

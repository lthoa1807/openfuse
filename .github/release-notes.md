### Release assets

- `openfuse-web-<version>.tar.gz` — static web build. Serve the extracted folder from any static host. `EXPO_PUBLIC_API_URL` is inlined at build time, so set it and rebuild to point the bundle at your API.
- `openfuse-server-<version>.tar.gz` — compiled API server (`dist/`) with the manifests it was built from. Extract it, run `pnpm install --prod --frozen-lockfile`, then `node dist/apps/server/src/index.js`. In a container set `HOST=0.0.0.0`, `WORKSPACE_MODE=live` and `DATA_DIR=/data`; the directory holds the PGlite store, documents and the signing key, so keep it private and back it up.
- `SHA256SUMS` — checksums for both archives.

### Container images

```sh
docker pull ghcr.io/@OWNER@/openfuse-browser-worker:<version>
docker pull ghcr.io/@OWNER@/openfuse-computer:<version>
```

Both are built from this tag and are also tagged `latest`. Run the browser worker with a random `WORKER_TOKEN` of at least 32 characters; commands for the computer image run as a nonroot user with no host mounts.

Configuration, deployment and boundaries are in [README.md](https://github.com/@OWNER@/openfuse#readme).

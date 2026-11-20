# API server image. Build context is the repository root:
#   docker build -f apps/server/Dockerfile -t openfuse-server .
#
# Two stages so the published image carries production dependencies only.
# syntax=docker/dockerfile:1

FROM node:24-bookworm-slim AS build
WORKDIR /repo
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY patches ./patches
COPY apps/mobile/package.json apps/mobile/package.json
COPY apps/worker/package.json apps/worker/package.json
RUN npm install --global pnpm@11.19.0 \
    && pnpm install --frozen-lockfile --ignore-scripts
COPY tsconfig.json tsconfig.build.json ./
COPY apps/server ./apps/server
COPY packages ./packages
RUN pnpm build:server

FROM node:24-bookworm-slim AS runtime
ARG TARGETARCH
WORKDIR /app
# curl + docker CLI: the computer feature runs sibling containers from this host.
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl \
    && rm -rf /var/lib/apt/lists/* \
    && case "${TARGETARCH:-amd64}" in arm64) arch=aarch64 ;; *) arch=x86_64 ;; esac \
    && curl -fsSL "https://download.docker.com/linux/static/stable/${arch}/docker-27.5.1.tgz" \
       | tar xz --strip-components=1 -C /usr/local/bin docker/docker \
    && docker --version
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY patches ./patches
COPY apps/mobile/package.json apps/mobile/package.json
COPY apps/worker/package.json apps/worker/package.json
RUN npm install --global pnpm@11.19.0 \
    && pnpm install --prod --frozen-lockfile \
    && pnpm store prune \
    && npm remove --global pnpm
COPY --from=build /repo/dist ./dist
ENV NODE_ENV=production \
    HOST=0.0.0.0 \
    PORT=8787 \
    DATA_DIR=/data/openfuse
EXPOSE 8787
# Keep the data directory private: it holds the PGlite store, documents and the
# signing key.
VOLUME ["/data"]
CMD ["node", "dist/apps/server/src/index.js"]

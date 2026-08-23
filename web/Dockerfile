# Multi-stage: build the React app, then serve it with nginx.
# Self-hosters never need Node locally — `docker compose up` builds everything.
#
# --platform=$BUILDPLATFORM pins the build stage to the host's native arch even when
# cross-building for other targets (e.g. amd64 host building an arm64 image). The build
# output (static JS/CSS/HTML) is arch-independent, so there's no reason to run it under
# QEMU — and QEMU-emulated npm installs are known to corrupt esbuild/rollup's platform-
# specific native binaries, which is what breaks `vite build` with unrelated-looking
# module-resolution errors.
FROM --platform=$BUILDPLATFORM node:22-alpine AS build
WORKDIR /app
COPY frontend/package.json frontend/package-lock.json* ./
RUN npm ci 2>/dev/null || npm install
COPY frontend/ ./
RUN npm run build

# Exercise media (img/gif), baked into the image at build time.
# On Railway (or any host without shared volumes between services), this replaces
# the separate `media` compose service + shared bind mount.
FROM alpine/git AS media
RUN mkdir -p /out/img /out/gif \
    && git clone --depth 1 https://github.com/hasaneyldrm/exercises-dataset /tmp/ds \
    && cp /tmp/ds/images/*.jpg /out/img/ \
    && cp /tmp/ds/videos/*.gif /out/gif/

FROM nginx:alpine
COPY web/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html

COPY --from=media /out/img /usr/share/nginx/html/img
COPY --from=media /out/gif /usr/share/nginx/html/gif
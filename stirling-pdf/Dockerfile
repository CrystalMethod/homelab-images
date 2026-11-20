# Stirling-PDF runtime image without the Brotli decoder.
#
# Stirling-PDF bundles org.brotli:dec 0.1.2. Apache HttpClient 5 advertises
# `Accept-Encoding: br` whenever that class is on the classpath, and Spring
# peeks one byte of every response body before reading the rest. The decoder
# loses buffered bytes in that sequence (google/brotli#1612), so every
# Brotli-compressed response up to 16 KiB is truncated. Cloudflare answers the
# Authentik token endpoint with Brotli, which breaks OIDC login with
# `invalid_token_response` / `Unexpected end-of-input`.
#
# Removing the decoder makes HttpClient negotiate gzip instead. Nothing else is
# changed: configuration, volumes and entrypoint stay as shipped upstream.
#
# To build locally:
#   docker build -t stirling-pdf:dev stirling-pdf/
#
# CI builds the image and pushes it to ghcr.io/crystalmethod/stirling-pdf (see
# .github/workflows).

# renovate: datasource=docker depName=docker.io/stirlingtools/stirling-pdf
FROM docker.io/stirlingtools/stirling-pdf:3.4.0-fat@sha256:0bc9e784a2345acecb3f5cbc52b3fc331427be45c98e6f5455bb20273374ba5d

RUN rm -- /app/lib/dec-0.1.2.jar

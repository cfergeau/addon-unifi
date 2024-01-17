ARG BUILD_FROM=ghcr.io/hassio-addons/ubuntu-base:9.0.3
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

ARG BUILD_ARCH=amd64

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base system
# hadolint ignore=DL3009
RUN \
    apt-get update \
    && apt-get upgrade -y \
    && apt-cache show binutils \
    && apt-cache show logrotate \
    && apt-get install -y --no-install-recommends \
        libc-bin=2.35-0ubuntu3.6 \
        binutils=2.38-4ubuntu2.5 \
        logrotate=3.19.0-1ubuntu1.1 \
        openjdk-17-jre-headless=17*

RUN \
    pkgname=libssl1.1_1.1.1f-1ubuntu2.21
    curl -L -O "http://security.ubuntu.com/ubuntu/pool/main/o/openssl/${pkgname}_amd64.deb" \
    && curl -L -O "http://ports.ubuntu.com/pool/main/o/openssl/${pkgname}_arm64.deb" \
    && if [[ "${BUILD_ARCH}" = "aarch64" ]]; then ARCH="arm64"; fi \
    && if [[ "${BUILD_ARCH}" = "amd64" ]]; then ARCH="amd64"; fi \
    && dpkg -i ${pkgname}_${ARCH}.deb \
    && rm -f ${pkgname}*

# install mongodb-org-server. Only the 6.0 and 7.0 repositories contain 'mongodb-org-server' packages
# beware of the licence change, previous mongodb version was 3.6.9 released under the AGPL, 7.0 is released under the non-free SSPL
# https://en.wikipedia.org/wiki/MongoDB#Licensing
# hadolint ignore=DL3008, DL3009
RUN \
    curl -J -L -o /etc/apt/keyrings/mongodb-server-4.4.gpg https://pgp.mongodb.com/server-4.4.pub \
    && echo "deb [ arch=amd64,arm64 signed-by=/etc/apt/keyrings/mongodb-server-4.4.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" >/etc/apt/sources.list.d/mongodb-org-4.4.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends mongodb-org-server

# need to ignore the mongodb-org-server dependency the unifi package requires 5.x or older, and we installed 7.x
RUN \
    && curl -J -L -o /tmp/unifi.deb \
        "https://dl.ui.com/unifi/8.0.28/unifi_sysvinit_all.deb" \
    && dpkg -i /tmp/unifi.deb

RUN \
    apt-get clean \
    && rm -fr \
        /tmp/* \
        /var/cache/* \
        /var/lib/apt/lists/* \
        /var/log/*.log \
        /var/log/apt

# Copy root filesystem
COPY rootfs /

# Health check
HEALTHCHECK --start-period=5m \
    CMD curl --insecure --fail https://localhost:8443 || exit 1

# Build arguments
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}

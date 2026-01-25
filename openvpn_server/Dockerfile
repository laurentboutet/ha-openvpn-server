ARG BUILD_FROM=ghcr.io/hassio-addons/base:15.0.1
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base system
RUN \
    apk add --no-cache \
        openvpn=2.6.8-r0 \
        easy-rsa=3.1.7-r0 \
        bash=5.2.21-r0 \
        iptables=1.8.10-r3 \
        ip6tables=1.8.10-r3 \
        tzdata=2024a-r0 \
        openssl=3.1.4-r6 \
        jq=1.7.1-r0 \
    && rm -rf /tmp/* /var/cache/apk/*

# Copy root filesystem
COPY rootfs /

# Set permissions
RUN chmod a+x /etc/services.d/openvpn/run \
    && chmod a+x /etc/services.d/openvpn/finish \
    && chmod a+x /usr/bin/openvpn-* \
    && mkdir -p /data/openvpn

# Labels
LABEL \
    io.hass.name="OpenVPN Server" \
    io.hass.description="Full-featured OpenVPN server with certificate management" \
    io.hass.arch="aarch64|amd64|armv7" \
    io.hass.type="addon" \
    io.hass.version="${BUILD_VERSION}"

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

ARG BUILD_FROM
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base system
RUN \
    apk add --no-cache \
        openvpn \
        easy-rsa \
        bash \
        iptables \
        ip6tables \
        tzdata \
        openssl \
        jq \
        curl \        
    && rm -rf /tmp/* /var/cache/apk/*

# Copy root filesystem
COPY rootfs /

# Set permissions
RUN chmod a+x /etc/services.d/*/run \
    && chmod a+x /etc/services.d/*/finish \
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

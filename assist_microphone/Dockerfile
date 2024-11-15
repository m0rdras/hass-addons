ARG BUILD_FROM
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /usr/src
ARG WYOMING_SATELLITE_VERSION
ENV PIP_BREAK_SYSTEM_PACKAGES=1
ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive

# Combine RUN commands and use buildkit cache mount for apt and pip
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    --mount=type=cache,target=/root/.cache/pip \
    apt-get update \
    && apt-get install -y --no-install-recommends \
        netcat-traditional \
        libasound2-plugins \
        alsa-utils \
        python3 \
        python3-pip \
    \
    && pip3 install --no-cache-dir -U \
        setuptools \
        wheel \
    && pip3 install --no-cache-dir \
        "wyoming-satellite[webrtc] @ https://github.com/rhasspy/wyoming-satellite/archive/refs/tags/v${WYOMING_SATELLITE_VERSION}.tar.gz" \
    \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY sounds/ ./sounds/

WORKDIR /
COPY rootfs /

HEALTHCHECK --start-period=10m \
    CMD echo '{ "type": "describe" }' \
        | nc -w 1 localhost 10700 \
        | grep -q "assist" \
        || exit 1

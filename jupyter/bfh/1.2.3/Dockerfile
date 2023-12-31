# syntax = docker/dockerfile:1.3 or higher
# VULN_SCAN_TIME=2023-07-03_05:15:54

# The build stage
# ---------------
# This stage is building Python wheels for use in later stages by using a base
# image that has more pre-requisites to do so, such as a C++ compiler.
#
# NOTE: If the image version is updated, also update it in ci/refreeze and
#       hub's Dockerfile!
#
FROM python:3.11-bullseye as build-stage

# Build wheels
#
# We set pip's cache directory and expose it across build stages via an
# ephemeral docker cache (--mount=type=cache,target=${PIP_CACHE_DIR}). We use
# the same technique for the directory /tmp/wheels.
#
COPY requirements.txt requirements.txt
ARG PIP_CACHE_DIR=/tmp/pip-cache
RUN --mount=type=cache,target=${PIP_CACHE_DIR} \
  pip install build && \
  pip wheel --wheel-dir=/tmp/wheels -r requirements.txt

# The final stage
# ---------------
#
FROM python:3.11-slim-bullseye
ENV DEBIAN_FRONTEND=noninteractive

ENV NB_USER=jovyan \
  NB_UID=1000 \
  HOME=/home/jovyan
RUN adduser \
  --disabled-password \
  --gecos "Default user" \
  --uid ${NB_UID} \
  --home ${HOME} \
  --force-badname \
  ${NB_USER}

# Install system dependencies
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      dnsutils \
      iputils-ping \
      tini \
      git \
      wget \
      curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Node.js >= 18.0.0 from NodeSource
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs

# Check Node version (this is optional, but can be useful for debugging)
RUN node -v

# install wheels built in the build-stage
COPY requirements.txt /tmp/requirements.txt
ARG PIP_CACHE_DIR=/tmp/pip-cache
RUN --mount=type=cache,target=${PIP_CACHE_DIR} \
  --mount=type=cache,from=build-stage,source=/tmp/wheels,target=/tmp/wheels \
  pip install \
  --find-links=/tmp/wheels/ \
  -r /tmp/requirements.txt

RUN jupyter lab build --dev-build=False --minimize=False

WORKDIR ${HOME}
USER ${NB_USER}

EXPOSE 8888
ENTRYPOINT ["tini", "--"]
CMD ["jupyter", "lab"]
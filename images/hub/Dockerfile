# syntax = docker/dockerfile:1.3
# VULN_SCAN_TIME=2024-09-16_05:06:48


# The build stage
# ---------------
# This stage is building Python wheels for use in later stages by using a base
# image that has more pre-requisites to do so, such as a C++ compiler.
#
# NOTE: If the image version is updated, also update it in ci/refreeze and
#       singleuser-sample's Dockerfile!
#
FROM python:3.11-10-bookworm as build-stage

# Build wheels
#
# We set pip's cache directory and expose it across build stages via an
# ephemeral docker cache (--mount=type=cache,target=${PIP_CACHE_DIR}). We use
# the same technique for the directory /tmp/wheels.
#
RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y --no-install-recommends \
        # requirement for pycurl
        libcurl4 \
        # requirement for using postgres database
        libpq5 \
        # requirement for using a local sqlite database
        sqlite3 \
        tini \
 && rm -rf /var/lib/apt/lists/*

COPY requirements.txt requirements.txt
ARG PIP_CACHE_DIR=/tmp/pip-cache
RUN --mount=type=cache,target=${PIP_CACHE_DIR} \
    pip wheel \
        --wheel-dir=/tmp/wheels \
        # pycurl 7.45.3 has wheels, but they aren't portable
        # https://github.com/pycurl/pycurl/issues/834
        --no-binary pycurl \
        -r requirements.txt \
        # Additional wheels for default-stage. Updates below should be repeated
        # in default-stage.
        #
        py-spy

RUN --mount=type=cache,target=${PIP_CACHE_DIR} \
    pip install --no-index --find-links=/tmp/wheels --target /opt/python-packages -r requirements.txt \
    --no-cache-dir

# The final stage - slim version
# ------------------------------
# This stage is built and published as quay.io/jupyterhub/k8s-hub-slim. It is meant to
# provide no non-essential packages.
#

# The final stage - slim version
# ------------------------------
# This stage is built and published as quay.io/jupyterhub/k8s-hub-slim. It is meant to
# provide no non-essential packages.
#
FROM 201959883603.dkr.ecr.us-east-2.amazonaws.com/mdaca/base-images/ironbank-ubuntu-python:22.04_3.11 as final

# Copy the installed Python packages from the build stage
COPY --from=build-stage /opt/python-packages /usr/local/lib/python3.11/site-packages

# Add user details
ARG NB_USER=jovyan \
    NB_UID=1000 \
    HOME=/home/jovyan
RUN adduser --disabled-password --gecos "Default user" --uid ${NB_UID} --home ${HOME} --force-badname ${NB_USER}

# Install essential system packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    dnsutils \
    git \
    less \
    vim \
    libpq5 \
    sqlite3 \
    tini && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /srv/jupyterhub
RUN chown ${NB_USER}:${NB_USER} /srv/jupyterhub

# Switch to non-root user
USER ${NB_USER}

# Expose port and define entrypoint and command for JupyterHub
EXPOSE 8081
ENTRYPOINT ["tini", "--"]
CMD ["jupyterhub", "--config", "/usr/local/etc/jupyterhub/jupyterhub_config.py"]

# JupyterHub runtime image.
#
# This image provides the JupyterHub runtime only (notebook base + JupyterHub,
# OAuthenticator, DockerSpawner and JupyterLab-Git). JupyterHub *configuration*
# is deliberately NOT baked in: `jupyterhub_config.py` is app-owned by the
# homelab repository and is mounted into the container at runtime. Keeping
# configuration out of the image means a config change does not require a
# rebuild and re-push.
#
# To build locally:
#   docker build -t jupyterhub:dev jupyterhub/
#
# CI builds multi-architecture images and pushes them to
# ghcr.io/crystalmethod/jupyterhub (see .github/workflows).

# renovate: datasource=docker depName=quay.io/jupyter/scipy-notebook
FROM quay.io/jupyter/scipy-notebook:2026-10-19@sha256:5d35c495b8544caa2b6bcf6634493e6003004a6b6388019533c6dca32c35658e

# renovate: datasource=pypi depName=jupyterhub versioning=pep440
ARG JUPYTERHUB_VERSION=6.1.1
# renovate: datasource=github-tags depName=jupyterhub/oauthenticator
ARG OAUTHENTICATOR_VERSION=17.4.0
# renovate: datasource=pypi depName=dockerspawner versioning=pep440
ARG DOCKERSPAWNER_VERSION=14.0.0
# renovate: datasource=npm depName=configurable-http-proxy
ARG CONFIGURABLE_HTTP_PROXY_VERSION=5.3.0
# renovate: datasource=pypi depName=jupyterlab-git versioning=pep440
ARG JUPYTERLAB_GIT_VERSION=0.55.0
# renovate: datasource=pypi depName=jupyter-ai versioning=pep440
ARG JUPYTER_AI_VERSION=3.2.0

USER root

RUN apt-get update \
    && apt-get install --yes --no-install-recommends \
        git \
        nodejs \
        npm \
    && npm install --global "configurable-http-proxy@${CONFIGURABLE_HTTP_PROXY_VERSION}" \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --no-cache-dir \
    "jupyterhub==${JUPYTERHUB_VERSION}" \
    "oauthenticator==${OAUTHENTICATOR_VERSION}" \
    "dockerspawner==${DOCKERSPAWNER_VERSION}" \
    "jupyterlab-git==${JUPYTERLAB_GIT_VERSION}" \
    "jupyter-ai==${JUPYTER_AI_VERSION}" \
    "langchain-openai"

USER jovyan
WORKDIR /home/jovyan

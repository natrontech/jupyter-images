
# ARG OWNER=jupyter
# ARG BASE_CONTAINER=$OWNER/scipy-notebook
# FROM $BASE_CONTAINER
FROM tensorflow/tensorflow:latest-gpu-jupyter

LABEL maintainer="Natron Tech <support@natron.io>"
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install Tensorflow with pip
RUN fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
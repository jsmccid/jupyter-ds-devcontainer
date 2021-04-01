# [Choice] Python version: 3, 3.8, 3.7, 3.6
ARG VARIANT=4d9c9bd9ced0
FROM jupyter/scipy-notebook:${VARIANT}

# add new packages at bottom in from jupyter

# vvvvvvvvvvvvv From python3 devcontainer tempalte vvvvvvvvvvvvvvvvvv
ENV PYTHONUNBUFFERED 1

# Update args in docker-compose.yaml to set the UID/GID of the "vscode" user.
# from ARG USER_UID=1000 ARG USER_GID=$USER_UID'

# copy args to equivalents for now
ARG NB_UID=1000
ARG NB_GID=1000
ARG NB_USER="jovyan"

ARG USERNAME=$NB_USER
ARG USER_UID=$NB_UID
ARG USER_GID=$NB_GID

USER root
# RUN if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then groupmod --gid $USER_GID $USERNAME && usermod --uid $USER_UID --gid $USER_GID jovyan; fi
# replacement from vscode docs https://code.visualstudio.com/docs/remote/containers-advanced
# !!!! problem with code, is it needed? !!!!!

# insterted > create usergroup for username to match with devcontainer structures
RUN DEBIAN_FRONTEND=noninteractive \
    && groupadd ${NB_USER} \
    && groupmod --gid $USER_GID $USERNAME \
    && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
    && chown -R $USER_UID:$USER_GID /home/$USERNAME

# ----- reset to non root user ----
 
USER $NB_UID

# [Optional] If your requirements rarely change, uncomment this section to add them to the image.
# COPY requirements.txt /tmp/pip-tmp/
# RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt \
#    && rm -rf /tmp/pip-tmp

# vvvvvvvvvvvv from miniconda devcontainer vvvvvvvvvvvvv
# https://github.com/microsoft/vscode-dev-containers/blob/master/containers/python-3-miniconda/.devcontainer/base.Dockerfile

ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="true"

# --- switch to root ----
USER root

COPY .devcontainer/library-scripts/common-debian.sh /tmp/library-scripts/
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# [Optional] Uncomment to install a different version of Python than the default
# RUN conda install -y python=3.6

# Setup default python tools in a venv via pipx to avoid conflicts
ENV PIPX_HOME=/usr/local/py-utils \
    PIPX_BIN_DIR=/usr/local/py-utils/bin
ENV PATH=${PATH}:${PIPX_BIN_DIR}
COPY .devcontainer/library-scripts/python-debian.sh /tmp/library-scripts/
RUN bash /tmp/library-scripts/python-debian.sh "none" "/opt/conda" "${PIPX_HOME}" "${USERNAME}" "false" \ 
    && apt-get clean -y && rm -rf /tmp/library-scripts

# [Option] Install Node.js
ARG INSTALL_NODE="true"
ARG NODE_VERSION="none"
ENV NVM_DIR=/usr/local/share/nvm
ENV NVM_SYMLINK_CURRENT=true \
    PATH=${NVM_DIR}/current/bin:${PATH}
COPY .devcontainer/library-scripts/node-debian.sh /tmp/library-scripts/
RUN if [ "$INSTALL_NODE" = "true" ]; then bash /tmp/library-scripts/node-debian.sh "${NVM_DIR}" "${NODE_VERSION}" "${USERNAME}"; fi \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# Copy environment.yml (if found) to a temp locaition so we update the environment. Also
# copy "noop.txt" so the COPY instruction does not fail if no environment.yml exists.
COPY environment.yml* .devcontainer/noop.txt /tmp/conda-tmp/
RUN if [ -f "/tmp/conda-tmp/environment.yml" ]; then /opt/conda/bin/conda env update -n base -f /tmp/conda-tmp/environment.yml; fi \
    && rm -rf /tmp/conda-tmp

# ----- reset to non root user ----

USER root

# vvvvvvv from R devcontainer vvvvvvvvvvv
# https://github.com/microsoft/vscode-dev-containers/blob/master/containers/r/.devcontainer/Dockerfile
# need to fix langauge server to work, get from https://hub.docker.com/r/rocker/r-ver

# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install \
#         python3-pip \
#         libgit2-dev \
#         libcurl4-openssl-dev \
#         libssl-dev \
#         libxml2-dev \
#     && apt-get autoremove -y && apt-get clean -y \
#     && python3 -m pip --no-cache-dir install radian \
#     && install2.r --error --skipinstalled --repos ${CRAN} --ncpus -1 \
#         devtools \
#         languageserver \
#     && rm -rf /tmp/downloaded_packages

# USER $NB_UID


# !!!!!  need to add from Julia devcontainer !!!!!
# https://github.com/julia-vscode/julia-devcontainer

# vvvvvvvvvvvv from jupyter/docker-stacks vvvvvvvvvvvv

# uncomment to apt install additional packages here
# USER root
# RUN apt-get update && \
#     apt-get install -y --no-install-recommends \
#     <additional-package-name> && \
#     apt-get clean && rm -rf /var/lib/apt/lists/*



USER $NB_UID

# install additional conda packages here
RUN conda install --quiet --yes \
#   '<additional-package-name>'
    'pandas-profiling=2.11.*' && \
    conda clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"


USER $NB_UID

WORKDIR $HOME
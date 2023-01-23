# You can pick any Debian/Ubuntu-based image. 😊
FROM mcr.microsoft.com/dotnet/sdk:6.0-focal

# Change Ubuntu mirror
RUN sed -i -e 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list

# Configure apt
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1

# [Option] Install zsh
ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"
# [Option] Enable non-root Docker access in container
ARG ENABLE_NONROOT_DOCKER="true"
# [Option] Use the OSS Moby Engine instead of the licensed Docker Engine
ARG USE_MOBY="true"
# [Option] Engine/CLI Version
ARG DOCKER_VERSION="latest"

# Enable new "BUILDKIT" mode for Docker CLI
ENV DOCKER_BUILDKIT=1

# Install needed packages and setup non-root user. Use a separate RUN statement to add your
# own dependencies. A user of "automatic" attempts to reuse an user ID if one already exists.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY library-scripts/*.sh /tmp/library-scripts/
RUN apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-in-docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "${USERNAME}" "${USE_MOBY}" "${DOCKER_VERSION}" \
    # Clean up
    && apt-get autoremove -y && apt-get clean -y

VOLUME [ "/var/lib/docker" ]

# Setting the ENTRYPOINT to docker-init.sh will start up the Docker Engine
# inside the container "overrideCommand": false is set in devcontainer.json.
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]

# [Choice] Node.js version: none, lts/*, 16, 14, 12, 10
ARG NODE_VERSION="16"
ENV NVM_DIR=/usr/local/share/nvm
ENV NVM_SYMLINK_CURRENT=true \
    PATH=${NVM_DIR}/current/bin:${PATH}
RUN bash /tmp/library-scripts/node-debian.sh "${NVM_DIR}" "${NODE_VERSION}" "${USERNAME}" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Remove library scripts for final image
RUN rm -rf /tmp/library-scripts

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends build-essential curl wget file git software-properties-common apt-transport-https ca-certificates vim-nox \
    && apt-get -y install --no-install-recommends python3-pip dos2unix \
    && apt-get -y install --no-install-recommends dotnet-sdk-5.0 \
    && apt-get -y install --no-install-recommends python-is-python3

COPY custom-scripts/*.sh /tmp/custom-scripts/
RUN bash /tmp/custom-scripts/install-curl.sh
RUN bash /tmp/custom-scripts/install-tcping.sh
RUN bash /tmp/custom-scripts/install-pwsh.sh

RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/cmdhistory/.bash_history" \
    && echo $SNIPPET >> "/root/.bashrc" \
    # [Optional] If you have a non-root user
    && mkdir /cmdhistory \
    && touch /cmdhistory/.bash_history \
    && chown -R $USERNAME.$USERNAME /cmdhistory \
    && echo $SNIPPET >> "/home/$USERNAME/.bashrc"

RUN mkdir -p /home/linuxbrew/.linuxbrew \
    && chown -R $USERNAME.$USERNAME /home/linuxbrew

USER vscode
RUN mkdir -p /home/$USERNAME/.config
RUN mkdir -p /home/$USERNAME/.cargo
RUN mkdir -p /home/$USERNAME/.rustup
RUN mkdir -p /home/$USERNAME/.vscode-server
RUN bash /tmp/custom-scripts/install-omz-plugins.sh

# Install typescript. eslint
ARG NODE_MODULES="eslint tslint-to-eslint-config typescript"
RUN umask 0002 && npm install -g ${NODE_MODULES} \
    && npm cache clean --force > /dev/null 2>&1

ARG EXTRA_NODE_VERSION=18
RUN bash -c "source ${NVM_DIR}/nvm.sh && nvm install ${EXTRA_NODE_VERSION} && nvm use default && node --version"

## Install PIP packages
RUN pip install --user --no-warn-script-location httpie pylint

## Rust - $HOME/.zshenv
RUN SNIPPET='. $HOME/.cargo/env' \
    && echo $SNIPPET >> "/home/$USERNAME/.zshenv"

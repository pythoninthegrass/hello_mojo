# syntax=docker/dockerfile:1.6

FROM ubuntu:22.04

ENV TZ=America/Chicago
# standardise on locale, don't generate .pyc, enable tracebacks on seg faults
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONFAULTHANDLER 1

# avoid stuck build due to user prompt
ARG DEBIAN_FRONTEND=noninteractive

RUN <<EOF
#!/usr/bin/env bash
# install dependencies
apt -qq update && apt -qq install \
    --no-install-recommends -y \
    apt-transport-https \
    build-essential \
    ca-certificates \
    curl \
    gcc \
    git \
    gnupg \
    icu-devtools \
    libc6 \
    libicu-dev \
    libicu70 \
    libncurses-dev \
    libpq-dev \
    libtinfo-dev \
    libxml2 \
    libxml2-dev \
    python3-pip \
    python3.10 \
    python3.10-venv \
    sudo \
    tzdata \
    vim \
    wget \
    && rm -rf /var/lib/apt/lists/*
EOF

RUN <<EOF
#!/usr/bin/env bash
conda init

python3 -m pip install \
    jupyterlab \
    ipykernel \
    matplotlib \
    ipywidgets
EOF

ARG AUTH_KEY=${AUTH_KEY}

RUN <<EOF
#!/usr/bin/env bash
keyring_location=/usr/share/keyrings/modular-installer-archive-keyring.gpg
curl -1sLf 'https://dl.modular.com/bBNWiLZX5igwHXeu/installer/gpg.0E4925737A3895AD.key' |  gpg --dearmor >> ${keyring_location}
curl -1sLf 'https://dl.modular.com/bBNWiLZX5igwHXeu/installer/config.deb.txt?distro=debian&codename=wheezy' > /etc/apt/sources.list.d/modular-installer.list
apt -qq update
apt -qq install -y modular
EOF

RUN modular auth $AUTH_KEY
RUN modular install mojo

WORKDIR /app

ARG MODULAR_HOME="/usr/local/bin/.modular"
ENV PATH="$PATH:$MODULAR_HOME/pkg/packages.modular.com_mojo/bin"

USER mojo

RUN jupyter labextension disable "@jupyterlab/apputils-extension:announcements"

EXPOSE 8888

CMD ["sleep", "infinity"]

# CMD ["jupyter", "lab", "--ip='*'", "--NotebookApp.token=''", "--NotebookApp.password=''", "--allow-root", "--no-browser"]

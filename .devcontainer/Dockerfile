#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM python:3

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Golang
# gcc for cgo
RUN apt-get update && apt-get install -y --no-install-recommends \
    g++ \
    gcc \
    libc6-dev \
    make \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.13

RUN set -eux; \
    \
    # this "case" statement is generated via "update.sh"
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
    amd64) goRelArch='linux-amd64'; goRelSha256='68a2297eb099d1a76097905a2ce334e3155004ec08cdea85f24527be3c48e856' ;; \
    armhf) goRelArch='linux-armv6l'; goRelSha256='931906d67cae1222f501e7be26e0ee73ba89420be0c4591925901cb9a4e156f0' ;; \
    arm64) goRelArch='linux-arm64'; goRelSha256='e2a61328101eff3b9c1ba47ecfec5eb2fdc3eb35d8c27d505737ba98bfcb197b' ;; \
    i386) goRelArch='linux-386'; goRelSha256='519b3e6ae4db011b93b60e6fabb055773ae6448355b6909a6befef87e02d98f5' ;; \
    ppc64el) goRelArch='linux-ppc64le'; goRelSha256='807b036bb058061b6090635e2a8612aaf301895dce70a773bbcd67fa1e57337c' ;; \
    s390x) goRelArch='linux-s390x'; goRelSha256='b7122795910b70b68e4118d0d34685a30925f4dd861c065cf20b699a7783807a' ;; \
    *) goRelArch='src'; goRelSha256='3fc0b8b6101d42efd7da1da3029c0a13f22079c0c37ef9730209d8ec665bf122'; \
    echo >&2; echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; echo >&2 ;; \
    esac; \
    \
    url="https://golang.org/dl/go${GOLANG_VERSION}.${goRelArch}.tar.gz"; \
    wget -O go.tgz "$url"; \
    echo "${goRelSha256} *go.tgz" | sha256sum -c -; \
    tar -C /usr/local -xzf go.tgz; \
    rm go.tgz; \
    \
    if [ "$goRelArch" = 'src' ]; then \
    echo >&2; \
    echo >&2 'error: UNIMPLEMENTED'; \
    echo >&2 'TODO install golang-any from jessie-backports for GOROOT_BOOTSTRAP (and uninstall after build)'; \
    echo >&2; \
    exit 1; \
    fi; \
    \
    export PATH="/usr/local/go/bin:$PATH"; \
    go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"

RUN go get github.com/mdempsky/gocode \
    && go get github.com/uudashr/gopkgs/cmd/gopkgs \
    && go get github.com/ramya-rao-a/go-outline \
    && go get github.com/acroca/go-symbols \
    && go get golang.org/x/tools/cmd/guru \
    && go get golang.org/x/tools/cmd/gorename \
    && go get github.com/cweill/gotests/... \
    && go get github.com/fatih/gomodifytags \
    && go get github.com/josharian/impl \
    && go get github.com/davidrjenni/reftools/cmd/fillstruct \
    && go get github.com/haya14busa/goplay/cmd/goplay \
    && go get github.com/godoctor/godoctor \
    && go get github.com/go-delve/delve/cmd/dlv \
    && go get github.com/stamblerre/gocode \
    && go get github.com/rogpeppe/godef \
    && go get github.com/sqs/goreturns \
    && go get golang.org/x/lint/golint

# Python
ENV PYTHONUNBUFFERED 1

# Uncomment the following COPY line and the corresponding lines in the `RUN` command if you wish to
# include your requirements in the image itself. It is suggested that you only do this if your
# requirements rarely (if ever) change.
# COPY requirements.txt /tmp/pip-tmp/

# Install docker
RUN apt-get update \
    && apt-get -y install docker.io

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git iproute2 procps lsb-release groff-base \
    #
    # Install python packages
    && pip install boto3==1.9.233 \
    && pip install botocore==1.12.233 \
    && pip install pylint \
    && pip install awscli \
    && pip install aws-sam-cli \
    #
    # Update Python environment based on requirements.txt
    # && pip --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt \
    # && rm -rf /tmp/pip-tmp \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=

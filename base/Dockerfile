FROM ubuntu:20.04
MAINTAINER tmpnix@alech.de

ARG buildprefix=/tmp
ARG NIX_VERSION=2.3.9

ENV TZ=America/New_York
RUN apt-get update
# Set timezone to work around an Ubuntu 20.04 issue getting stuck at tzdata configuration
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime  && echo $TZ > /etc/timezone
RUN apt-get install -y pkg-config build-essential libbz2-dev libsqlite3-dev libcurl4-openssl-dev liblzma-dev brotli libseccomp-dev libssl-dev libboost-context-dev xz-utils libbrotli-dev python3 wget libc++-dev wget apt-utils autoconf autoconf-archive jq libarchive-dev zlib1g-dev libgc-dev

# Download and install editline
RUN wget https://github.com/troglobit/editline/releases/download/1.16.0/editline-1.16.0.tar.xz
RUN tar xf editline-1.16.0.tar.xz
RUN cd editline-1.16.0 && ./configure --prefix=/usr && make && make install

# Download and compile NixOS
RUN wget https://nixos.org/releases/nix/nix-${NIX_VERSION}/nix-${NIX_VERSION}.tar.xz
RUN tar xf nix-${NIX_VERSION}.tar.xz
RUN cd nix-${NIX_VERSION} && ./configure --prefix=/usr/local --with-store-dir=${buildprefix}/nix/store --localstatedir=${buildprefix}/nix/var && make && make install

# Add user and setup nixOS
RUN useradd tmpnix --create-home --user-group --no-log-init --shell=/bin/bash
RUN mkdir -p ${buildprefix}/nix
RUN chown tmpnix ${buildprefix}/nix
USER tmpnix
RUN echo "${buildprefix}" > /home/tmpnix/.buildprefix
RUN mkdir /home/tmpnix/.config
RUN /usr/local/bin/nix-store --init
ENV NIX_CONF_DIR /home/tmpnix/.config
RUN echo 'build-use-chroot = false' > /home/tmpnix/.config/nix.conf
RUN echo 'build-users-group = ' >> /home/tmpnix/.config/nix.conf
RUN /usr/local/bin/nix-channel --add https://nixos.org/channels/nixos-18.09 nixpkgs
RUN /usr/local/bin/nix-channel --add https://nixos.org/channels/nixpkgs-unstable unstable
RUN /usr/local/bin/nix-channel --update nixpkgs
RUN /usr/local/bin/nix-channel --update unstable
COPY tmpnix.py /home/tmpnix
ENTRYPOINT ["/home/tmpnix/tmpnix.py"]
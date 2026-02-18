FROM archlinux:latest
LABEL maintainer="Khas-flomesh<khas@flomesh.io>"

ARG VERSION
ENV VERSION=${VERSION:-latest}

ARG REVISION
ENV REVISION=${REVISION:-1}

ARG COMMIT_ID
ENV CI_COMMIT_SHA=${COMMIT_ID}

ARG COMMIT_DATE
ENV CI_COMMIT_DATE=${COMMIT_DATE}

ARG PIPY_GUI
ENV PIPY_GUI=${PIPY_GUI:-OFF}

ARG PIPY_STATIC
ENV PIPY_STATIC=${PIPY_STATIC:-OFF}

ARG BUILD_TYPE
ENV BUILD_TYPE=${BUILD_TYPE:-Release}

COPY pipy.tar.gz /pipy.tar.gz
COPY PKGBUILD /PKGBUILD

RUN pacman -Syu --noconfirm \
    && pacman -S --noconfirm --needed base-devel clang cmake git perl zlib chrpath \
    && git config --global user.email "build@flomesh.io" \
    && git config --global user.name "Build"

# makepkg refuses to run as root, create a builder user
RUN useradd -m builder \
    && mkdir -p /home/builder/build \
    && cp /pipy.tar.gz /home/builder/build/pipy.tar.gz \
    && cp /PKGBUILD /home/builder/build/PKGBUILD \
    && chown -R builder:builder /home/builder/build

USER builder
RUN cd /home/builder/build \
    && git config --global user.email "build@flomesh.io" \
    && git config --global user.name "Build" \
    && export PIPY_GUI=${PIPY_GUI} \
    && export PIPY_STATIC=${PIPY_STATIC} \
    && export BUILD_TYPE=${BUILD_TYPE} \
    && export CI_COMMIT_SHA=${CI_COMMIT_SHA} \
    && export CI_COMMIT_TAG=${VERSION}-${REVISION} \
    && export CI_COMMIT_DATE="${CI_COMMIT_DATE}" \
    && makepkg --noconfirm --skipchecksums

USER root
RUN mkdir -p /pacman \
    && cp /home/builder/build/*.pkg.tar.zst /pacman/

FROM debian:bookworm
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
COPY control /control

RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential clang cmake git perl zlib1g-dev fakeroot chrpath ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && git config --global user.email "build@flomesh.io" \
    && git config --global user.name "Build"

RUN mkdir -p /build && cd /build \
    && tar xzf /pipy.tar.gz \
    && cd pipy \
    && mkdir build && cd build \
    && CC=clang CXX=clang++ cmake \
         -DPIPY_GUI=${PIPY_GUI} \
         -DPIPY_CODEBASES=${PIPY_GUI} \
         -DPIPY_STATIC=${PIPY_STATIC} \
         -DCMAKE_BUILD_TYPE=${BUILD_TYPE} .. \
    && make -j$(nproc)

RUN set -e \
    && ARCH=$(dpkg --print-architecture) \
    && PKG_DIR=/build/pipy-pkg \
    && mkdir -p ${PKG_DIR}/DEBIAN \
    && mkdir -p ${PKG_DIR}/usr/local/bin \
    && mkdir -p ${PKG_DIR}/etc/pipy \
    # control file
    && sed "s/@ARCH@/${ARCH}/g" /control > ${PKG_DIR}/DEBIAN/control \
    # preinst: create pipy user/group
    && cat > ${PKG_DIR}/DEBIAN/preinst <<'PREINST' \
#!/bin/sh
set -e
getent group pipy >/dev/null || groupadd -r pipy
getent passwd pipy >/dev/null || useradd -r -g pipy -G pipy -d /etc/pipy -s /usr/sbin/nologin -c "pipy" pipy
PREINST
    && chmod 755 ${PKG_DIR}/DEBIAN/preinst \
    # prerm: stop service
    && cat > ${PKG_DIR}/DEBIAN/prerm <<'PRERM' \
#!/bin/sh
set -e
if [ "$1" = "remove" ]; then
    systemctl --no-reload disable pipy.service >/dev/null 2>&1 || true
    systemctl stop pipy.service >/dev/null 2>&1 || true
fi
PRERM
    && chmod 755 ${PKG_DIR}/DEBIAN/prerm \
    # postrm: remove user on purge
    && cat > ${PKG_DIR}/DEBIAN/postrm <<'POSTRM' \
#!/bin/sh
set -e
if [ "$1" = "purge" ]; then
    userdel pipy 2>/dev/null || true
fi
POSTRM
    && chmod 755 ${PKG_DIR}/DEBIAN/postrm \
    # binary
    && cp /build/pipy/bin/pipy ${PKG_DIR}/usr/local/bin/pipy \
    && chrpath --delete ${PKG_DIR}/usr/local/bin/pipy || true \
    && chmod 755 ${PKG_DIR}/usr/local/bin/pipy \
    # build deb
    && mkdir -p /deb \
    && fakeroot dpkg-deb --build ${PKG_DIR} /deb/pipy-${VERSION}-${REVISION}-${ARCH}.deb

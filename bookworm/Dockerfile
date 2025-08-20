FROM postgres:17.6-bookworm AS plv8_build

ENV PLV8_BRANCH=r3.2
ENV PLV8_VERSION=3.2.4
ENV PG_MAJOR=17

ARG BIGINT_GRACEFUL_VALUE='BIGINT_GRACEFUL=1'
ARG BIGINT_GRACEFUL
ARG BIGINT_GRACEFUL_FLAG=${BIGINT_GRACEFUL:+$BIGINT_GRACEFUL_VALUE}

RUN buildDependencies="build-essential \
    ca-certificates \
    curl \
    git-core \
    postgresql-server-dev-${PG_MAJOR} \
    libtinfo5 \
    pkg-config \
    clang \
    binutils \
    libstdc++-12-dev \
    cmake" \
    && apt-get update && apt-get install -y --no-install-recommends ${buildDependencies}
	
RUN mkdir -p /tmp/build \
  && cd /tmp/build \
  && git clone --branch ${PLV8_BRANCH} --single-branch --depth 1 https://github.com/plv8/plv8 \
  && cd plv8 \
  && git submodule update --init \
  && cd deps/v8-cmake \
  && git checkout master \
  && cd ../.. \
  && make ${BIGINT_GRACEFUL_FLAG} \
  && strip plv8-${PLV8_VERSION}.so \
  && make install

RUN rm -rf /root/.vpython_cipd_cache /root/.vpython-root \
  && apt-get clean \
  && apt-get remove -y ${buildDependencies} \
  && apt-get autoremove -y \
  && rm -rf /tmp/build /var/lib/apt/lists/*


FROM docker.io/bitnami/minideb:bookworm

ARG DOWNLOADS_URL="downloads.bitnami.com/files/stacksmith"
ARG EXTRA_LOCALES
ARG TARGETARCH
ARG WITH_ALL_LOCALES="no"

LABEL com.vmware.cp.artifact.flavor="sha256:c50c90cfd9d12b445b011e6ad529f1ad3daea45c26d20b00732fae3cd71f6a83" \
      org.opencontainers.image.base.name="docker.io/bitnami/minideb:bookworm" \
      org.opencontainers.image.created="2025-08-14T16:42:42Z" \
      org.opencontainers.image.description="Application packaged by Broadcom, Inc." \
      org.opencontainers.image.documentation="https://github.com/bitnami/containers/tree/main/bitnami/postgresql/README.md" \
      org.opencontainers.image.source="https://github.com/bitnami/containers/tree/main/bitnami/postgresql" \
      org.opencontainers.image.title="postgresql" \
      org.opencontainers.image.vendor="Broadcom, Inc." \
      org.opencontainers.image.version="17.6.0"

ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-12" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]
# Install required system packages and dependencies
RUN install_packages ca-certificates curl libbsd0 libbz2-1.0 libedit2 libffi8 libgcc-s1 libgmp10 libgnutls30 libhogweed6 libicu72 libidn2-0 libldap-2.5-0 liblz4-1 liblzma5 libmd0 libnettle8 libp11-kit0 libpcre3 libreadline8 libsasl2-2 libsqlite3-0 libssl3 libstdc++6 libtasn1-6 libtinfo6 libunistring2 libuuid1 libxml2 libxslt1.1 libzstd1 locales procps zlib1g
RUN --mount=type=secret,id=downloads_url,env=SECRET_DOWNLOADS_URL \
    DOWNLOADS_URL=${SECRET_DOWNLOADS_URL:-${DOWNLOADS_URL}} ; \
    mkdir -p /tmp/bitnami/pkg/cache/ ; cd /tmp/bitnami/pkg/cache/ || exit 1 ; \
    COMPONENTS=( \
      "postgresql-17.6.0-0-linux-${OS_ARCH}-debian-12" \
    ) ; \
    for COMPONENT in "${COMPONENTS[@]}"; do \
      if [ ! -f "${COMPONENT}.tar.gz" ]; then \
        curl -SsLf "https://${DOWNLOADS_URL}/${COMPONENT}.tar.gz" -O ; \
        curl -SsLf "https://${DOWNLOADS_URL}/${COMPONENT}.tar.gz.sha256" -O ; \
      fi ; \
      sha256sum -c "${COMPONENT}.tar.gz.sha256" ; \
      tar -zxf "${COMPONENT}.tar.gz" -C /opt/bitnami --strip-components=2 --no-same-owner ; \
      rm -rf "${COMPONENT}".tar.gz{,.sha256} ; \
    done
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives
RUN chmod g+rwX /opt/bitnami
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true
RUN update-locale LANG=C.UTF-8 LC_MESSAGES=POSIX && \
    DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales && \
    echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
RUN uninstall_packages curl

COPY rootfs /
RUN /opt/bitnami/scripts/postgresql/postunpack.sh
RUN update-locale LANG=C.UTF-8 LC_MESSAGES=POSIX && \
    DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales && \
    echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
RUN /opt/bitnami/scripts/locales/generate-locales.sh
ENV APP_VERSION="17.6.0" \
    BITNAMI_APP_NAME="postgresql" \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    NSS_WRAPPER_LIB="/opt/bitnami/common/lib/libnss_wrapper.so" \
    PATH="/opt/bitnami/postgresql/bin:$PATH"

ARG PG_MAJOR=17
ARG PLV8_VERSION=3.2.4
COPY --from=plv8_build /usr/share/postgresql/${PG_MAJOR}/extension/ /opt/bitnami/postgresql/share/extension/
COPY --from=plv8_build /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so /opt/bitnami/postgresql/lib/plv8-${PLV8_VERSION}.so

VOLUME [ "/bitnami/postgresql", "/docker-entrypoint-initdb.d", "/docker-entrypoint-preinitdb.d" ]

EXPOSE 5432

USER 1001
ENTRYPOINT [ "/opt/bitnami/scripts/postgresql/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/postgresql/run.sh" ]

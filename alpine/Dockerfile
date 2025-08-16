FROM postgres:17.6-alpine AS plv8_build

ENV PLV8_BRANCH=r3.2
ENV PLV8_VERSION=3.2.4
ENV PG_MAJOR=17

ARG BIGINT_GRACEFUL_VALUE='BIGINT_GRACEFUL=1'
ARG BIGINT_GRACEFUL
ARG BIGINT_GRACEFUL_FLAG=${BIGINT_GRACEFUL:+$BIGINT_GRACEFUL_VALUE}

RUN apk update \
    && apk add --no-cache --virtual .v8-build \
  libstdc++-dev \
  binutils \
  gcc \
  libc-dev \
  g++ \
  patch \  
  ca-certificates \
  curl \
  make \
  libbz2 \
  linux-headers \
  cmake \
  clang19-libs \
  clang19 \
  llvm19 \
  ncurses-libs \
  zlib-dev \
  git \
  python3

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

RUN apk del --no-network .v8-build; \
  rm -rf /tmp/* /var/tmp/*


FROM postgres:17.6-alpine
ENV PLV8_VERSION=3.2.4
COPY --from=plv8_build /usr/local/share/postgresql/extension/ /usr/local/share/postgresql/extension/
COPY --from=plv8_build /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so

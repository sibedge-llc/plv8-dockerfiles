FROM postgres:15.4-alpine AS plv8_build

ENV PLV8_VERSION=3.2.0
ENV PG_MAJOR=15

ARG BIGINT_GRACEFUL_VALUE='BIGINT_GRACEFUL=1'
ARG BIGINT_GRACEFUL
ARG BIGINT_GRACEFUL_FLAG=${BIGINT_GRACEFUL:+$BIGINT_GRACEFUL_VALUE}

RUN apk update \
    && apk add --no-cache --virtual .v8-build \
  libstdc++-dev \
  binutils \
  libgomp \
  libatomic \
  gmp \
  isl25 \
  mpfr4 \
  mpc1 \
  gcc \
  musl-dev \
  libc-dev \
  g++ \
  ca-certificates \
  brotli-libs \
  nghttp2-libs \
  libcurl \
  curl \
  make \
  libbz2 \
  perl \
  linux-headers \
  libexpat \
  mpdecimal \
  sqlite-libs \
  clang15-libs \
  clang15 \
  llvm15 \
  lld-libs \
  lld \
  cmake \
  ncurses \
  ncurses-libs \
  zlib-dev \
  git \
  python3

RUN mkdir -p /tmp/build \
  && curl -o /tmp/build/v$PLV8_VERSION.tar.gz -SL "https://github.com/plv8/plv8/archive/refs/tags/v${PLV8_VERSION}.tar.gz" \
  && cd /tmp/build \
  && tar -xzf /tmp/build/v$PLV8_VERSION.tar.gz -C /tmp/build/
  
RUN cd /tmp/build/plv8-$PLV8_VERSION/deps \
  && git clone https://github.com/bnoordhuis/v8-cmake.git
  
RUN cd /tmp/build/plv8-$PLV8_VERSION \
  && git init \
  && make ${BIGINT_GRACEFUL_FLAG} \
  && make install \
  && strip /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so

RUN apk del --no-network .v8-build; \
  rm -rf /tmp/* /var/tmp/*


FROM postgres:15.4-alpine
ENV PLV8_VERSION=3.2.0
COPY --from=plv8_build /usr/local/share/postgresql/extension/ /usr/local/share/postgresql/extension/
COPY --from=plv8_build /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so

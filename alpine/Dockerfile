FROM postgres:17.0-alpine AS plv8_build

ENV PLV8_VERSION=3.2.3
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
  ca-certificates \
  curl \
  make \
  libbz2 \
  linux-headers \
  cmake \
  clang15-libs \
  clang15 \
  llvm15 \
  ncurses-libs \
  zlib-dev \
  git \
  python3

RUN mkdir -p /tmp/build \
  && curl -o /tmp/build/v$PLV8_VERSION.tar.gz -SL "https://github.com/plv8/plv8/archive/refs/tags/v${PLV8_VERSION}.tar.gz" \
  && cd /tmp/build \
  && tar -xzf /tmp/build/v$PLV8_VERSION.tar.gz -C /tmp/build/
  
RUN cd /tmp/build/plv8-$PLV8_VERSION/deps \
  && git clone https://github.com/bnoordhuis/v8-cmake.git \
  && cd ./v8-cmake \
  && git checkout beb327f02f4a7e200b9ec
  
RUN cd /tmp/build/plv8-$PLV8_VERSION \
  && git init \
  && make ${BIGINT_GRACEFUL_FLAG} \
  && make install \
  && strip /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so

RUN apk del --no-network .v8-build; \
  rm -rf /tmp/* /var/tmp/*


FROM postgres:17.0-alpine
ENV PLV8_VERSION=3.2.3
COPY --from=plv8_build /usr/local/share/postgresql/extension/ /usr/local/share/postgresql/extension/
COPY --from=plv8_build /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so /usr/local/lib/postgresql/plv8-${PLV8_VERSION}.so

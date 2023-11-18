FROM debian:bookworm-slim AS plv8_build

ENV PLV8_VERSION=3.2.0
ENV PG_MAJOR=15

ARG BIGINT_GRACEFUL_VALUE='BIGINT_GRACEFUL=1'
ARG BIGINT_GRACEFUL
ARG BIGINT_GRACEFUL_FLAG=${BIGINT_GRACEFUL:+$BIGINT_GRACEFUL_VALUE}

RUN buildDependencies="build-essential \
    ca-certificates \
    curl \
    git-core \
    gpp \
    cpp \
	gnupg dirmngr \
    pkg-config \
    apt-transport-https \
    cmake \
    libc++-dev \
    libncurses5 \
    libc++abi-dev \
	libstdc++-12-dev \
	wget \
	zlib1g-dev \
	libtinfo5" \
    runtimeDependencies="libc++1" \
    && apt-get update && apt-get install -y --no-install-recommends ${buildDependencies} ${runtimeDependencies}
	
RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt bookworm-pgdg main 15" > /etc/apt/sources.list.d/pgdg.list' \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -

RUN apt-get update && apt-get install -y postgresql-$PG_MAJOR postgresql-server-dev-$PG_MAJOR

	
RUN mkdir -p /tmp/build \
  && curl -o /tmp/build/v$PLV8_VERSION.tar.gz -SL "https://github.com/plv8/plv8/archive/refs/tags/v${PLV8_VERSION}.tar.gz" \
  && cd /tmp/build \
  && tar -xzf /tmp/build/v$PLV8_VERSION.tar.gz -C /tmp/build/
  
RUN cd /tmp/build/plv8-$PLV8_VERSION/deps \
  && git clone https://github.com/bnoordhuis/v8-cmake.git \ 
  && cd /tmp/build/plv8-$PLV8_VERSION \
  && git init \
  && make ${BIGINT_GRACEFUL_FLAG} \
  && make install \
  && strip /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so
RUN rm -rf /root/.vpython_cipd_cache /root/.vpython-root \
  && apt-get clean \
  && apt-get remove -y ${buildDependencies} \
  && apt-get autoremove -y \
  && rm -rf /tmp/build /var/lib/apt/lists/*

FROM postgres:15.5-bookworm
RUN mkdir -p /usr/lib/postgresql/${PG_MAJOR}/lib/
ENV PLV8_VERSION=3.2.0
COPY --from=plv8_build /usr/share/postgresql/${PG_MAJOR}/extension/ /usr/share/postgresql/${PG_MAJOR}/extension/
COPY --from=plv8_build /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so

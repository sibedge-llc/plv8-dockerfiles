FROM postgres:13.7 AS plv8_build

ENV PLV8_VERSION=3.0.0
ENV PG_MAJOR=13
RUN buildDependencies="build-essential \
    ca-certificates \
    curl \
    git-core \
    python \
    python3 \
    python3-pip \
    gpp \
    cpp \
    pkg-config \
    apt-transport-https \
    cmake \
    libc++-dev \
    libncurses5 \
    libc++abi-dev \
    postgresql-server-dev-$PG_MAJOR" \
    runtimeDependencies="libc++1" \
    &&  apt-get update && apt-get install -y --no-install-recommends ${buildDependencies} ${runtimeDependencies}
RUN mkdir -p /tmp/build \
  && curl -o /tmp/build/v$PLV8_VERSION.tar.gz -SL "https://github.com/plv8/plv8/archive/refs/tags/v${PLV8_VERSION}.tar.gz" \
  && cd /tmp/build \
  && tar -xzf /tmp/build/v$PLV8_VERSION.tar.gz -C /tmp/build/ \
  && cd /tmp/build/plv8-$PLV8_VERSION \
  && make static \
  && make install \
  && strip /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so
RUN rm -rf /root/.vpython_cipd_cache /root/.vpython-root \
  && apt-get clean \
  && apt-get remove -y ${buildDependencies} \
  && apt-get autoremove -y \
  && rm -rf /tmp/build /var/lib/apt/lists/*

FROM postgres:13.7
RUN mkdir -p /usr/lib/postgresql/${PG_MAJOR}/lib/
ENV PLV8_VERSION=3.0.0
RUN runtimeDependencies="libc++1" \
  && apt-get update \
  && apt-get install -y --no-install-recommends ${runtimeDependencies}
COPY --from=plv8_build /usr/share/postgresql/${PG_MAJOR}/extension/ /usr/share/postgresql/${PG_MAJOR}/extension/
COPY --from=plv8_build /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so

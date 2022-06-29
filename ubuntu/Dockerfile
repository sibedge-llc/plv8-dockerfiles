FROM debian:bookworm-slim AS plv8_build

ENV PLV8_VERSION=3.1.1
ENV PG_MAJOR=14

RUN buildDependencies="build-essential \
    postgresql-14 \
	postgresql-server-dev-14 \
    ca-certificates \
    curl \
    git-core \
    gpp \
    cpp \
    pkg-config \
    apt-transport-https \
    cmake \
    libc++-dev \
    libncurses5 \
    libc++abi-dev \
	wget \
	zlib1g-dev \
	ninja-build \
    postgresql-server-dev-$PG_MAJOR" \
    runtimeDependencies="libc++1" \
    &&  apt-get update && apt-get install -y --no-install-recommends ${buildDependencies} ${runtimeDependencies}

ENV PATH="${PATH}:/Python-3.9.13"
	
RUN wget https://www.python.org/ftp/python/3.9.13/Python-3.9.13.tgz \
    && tar zxf Python-3.9.13.tgz \
    && cd Python-3.9.13/ \
    && ./configure --prefix=/usr/local \
    && make \
    && make install	\
	&& cd ./Tools/scripts \
	&& wget https://bootstrap.pypa.io/get-pip.py \
	&& python get-pip.py 
	
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

FROM ubuntu/postgres:14-22.04_edge
RUN mkdir -p /usr/lib/postgresql/${PG_MAJOR}/lib/
ENV PLV8_VERSION=3.1.1
COPY --from=plv8_build /usr/share/postgresql/${PG_MAJOR}/extension/ /usr/share/postgresql/${PG_MAJOR}/extension/
COPY --from=plv8_build /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so

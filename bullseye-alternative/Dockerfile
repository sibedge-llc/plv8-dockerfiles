FROM postgres:14.5 AS plv8_build

ARG PLV8_VERSION=3.0.0
ARG PG_MAJOR=14
ARG V8_VERSION=8.6.405

ARG BIGINT_GRACEFUL_VALUE='BIGINT_GRACEFUL=1'
ARG BIGINT_GRACEFUL
ARG BIGINT_GRACEFUL_FLAG=${BIGINT_GRACEFUL:+$BIGINT_GRACEFUL_VALUE}

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
	clang \
	lld \
    libc++-dev \
    libncurses5 \
    libc++abi-dev \
    postgresql-server-dev-$PG_MAJOR" \
    runtimeDependencies="libc++1" \
    &&  apt-get update && apt-get install -y --no-install-recommends ${buildDependencies} ${runtimeDependencies}
RUN mkdir -p /tmp/build/plv8-${PLV8_VERSION}/build/v8/build \
  && curl -o /tmp/build/plv8-${PLV8_VERSION}/build/v8.tar.gz "https://chromium.googlesource.com/v8/v8/+archive/${V8_VERSION}.tar.gz" \
  && tar -xzf /tmp/build/plv8-${PLV8_VERSION}/build/v8.tar.gz -C /tmp/build/plv8-${PLV8_VERSION}/build/v8
  
RUN curl -o /tmp/build/plv8-${PLV8_VERSION}/build/chromium.tar.gz "https://chromium.googlesource.com/chromium/src/build/+archive/183d29ca633d95300aef4dd4bb66f058ff106615.tar.gz" \
  && tar -xzf /tmp/build/plv8-${PLV8_VERSION}/build/chromium.tar.gz -C /tmp/build/plv8-${PLV8_VERSION}/build/v8/build

RUN mkdir -p /tmp/build/plv8-${PLV8_VERSION}/build/v8/base/trace_event/common \
  && curl -o /tmp/build/plv8-${PLV8_VERSION}/build/common.tar.gz "https://chromium.googlesource.com/chromium/src/base/trace_event/common/+archive/23ef5333a357fc7314630ef88b44c3a545881dee.tar.gz" \
  && tar -xzf /tmp/build/plv8-${PLV8_VERSION}/build/common.tar.gz -C /tmp/build/plv8-${PLV8_VERSION}/build/v8/base/trace_event/common

RUN mkdir -p /tmp/build/plv8-${PLV8_VERSION}/build/v8/buildtools \
  && curl -o /tmp/build/plv8-${PLV8_VERSION}/build/buildtools.tar.gz "https://chromium.googlesource.com/chromium/src/buildtools/+archive/b00ad0af636401e5eb4b5d0ab01b65164dca1914.tar.gz" \
  && tar -xzf /tmp/build/plv8-${PLV8_VERSION}/build/buildtools.tar.gz -C /tmp/build/plv8-${PLV8_VERSION}/build/v8/buildtools

RUN mkdir -p /tmp/build/plv8-${PLV8_VERSION}/build/v8/third_party/zlib \
  && curl -o /tmp/build/plv8-${PLV8_VERSION}/build/zlib.tar.gz "https://chromium.googlesource.com/chromium/src/third_party/zlib/+archive/d53accfbd0382a98ad7378045631866449b5f92e.tar.gz" \
  && tar -xzf /tmp/build/plv8-${PLV8_VERSION}/build/zlib.tar.gz -C /tmp/build/plv8-${PLV8_VERSION}/build/v8/third_party/zlib

RUN mkdir -p /tmp/build/plv8-${PLV8_VERSION}/build/v8/third_party/googletest/src \
  && curl -o /tmp/build/plv8-${PLV8_VERSION}/build/googletest.tar.gz "https://chromium.googlesource.com/external/github.com/google/googletest/+archive/4fe018038f87675c083d0cfb6a6b57c274fb1753.tar.gz" \
  && tar -xzf /tmp/build/plv8-${PLV8_VERSION}/build/googletest.tar.gz -C /tmp/build/plv8-${PLV8_VERSION}/build/v8/third_party/googletest/src

COPY Makefile.debian /tmp/build/plv8-$PLV8_VERSION/
COPY linux64 /tmp/build/plv8-$PLV8_VERSION/build/v8/buildtools/linux64/
COPY gclient_args.gni /tmp/build/plv8-$PLV8_VERSION/build/v8/build/config/

RUN cd /tmp/build \
  && git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

RUN ln -s /usr/lib/llvm-11/bin/llvm-ar /usr/bin/llvm-ar

RUN cd /tmp/build/plv8-${PLV8_VERSION}/build/v8 \
  && sed -ri -e 's/"(src|test)\/inspector:.*",?//' -e 's/"(tools|test):gn_all",?//' -e 's/"test\/(cctest|unittests):.*",?//' -e 's/"\/\/build\/(win:default_exe_manifest|config:exe_and_shlib_deps)",?//' BUILD.gn \
  && sed -ri 's/(set_sources_assignment_filter)\(.*\)/\1([])/' build/config/BUILDCONFIG.gn \
  && sed -ri 's/\((V8_OS_LINUX).*[^=\)]\)?/\1/g' src/base/platform/condition-variable.cc \
  && sed -ri -e 's/^(#if) V8_LIBC_GLIBC.*/\1 0/' -e '/const char k(MangledSymbolPrefix|SymbolCharacters)/{N;/;/d}' src/base/debug/stack_trace_posix.cc \
  && sed -ri -e '/^#include <execinfo.h>s*/d' -e 's/^#ifdef SYMBOLIZE_FUNCTION.*/#if 0/' src/codegen/external-reference-table.cc \
  && ./buildtools/linux64/gn gen out '--script-executable=python3' '--args=       target_cpu="x64"       target_os="linux"       symbol_level=0             is_cfi=false       is_clang=true       clang_base_path="/usr" clang_use_chrome_plugins=false       use_sysroot=false       use_custom_libcxx=false       use_custom_libcxx_for_host=false       use_glib=false       use_gold=false       use_lld=true       use_thin_lto=false thin_lto_enable_optimizations=false       fatal_linker_warnings=false       treat_warnings_as_errors=false             v8_target_cpu="x64"       is_debug=false              v8_enable_disassembler=false       v8_symbol_level=0             v8_static_library=false       is_component_build=false v8_monolithic=true v8_use_external_startup_data=false                    v8_use_siphash=false       v8_win64_unwinding_info=false       v8_enable_backtrace=true       v8_enable_gdbjit=false       v8_enable_vtunejit=false       v8_enable_vtunetracemark=false       v8_enable_handle_zapping=false       v8_enable_google_benchmark=false       v8_enable_i18n_support=false       v8_enable_object_print=false       v8_typed_array_max_size_in_heap=64       v8_enable_pointer_compression=true v8_enable_31bit_smis_on_64bit_arch=true            '

ENV PATH="${PATH}:/tmp/build/depot_tools"

RUN curl -o /tmp/build/v$PLV8_VERSION.tar.gz -SL "https://github.com/plv8/plv8/archive/refs/tags/v${PLV8_VERSION}.tar.gz" \
  && cd /tmp/build \
  && tar -xzf /tmp/build/v$PLV8_VERSION.tar.gz -C /tmp/build/ \
  && cd /tmp/build/plv8-$PLV8_VERSION \
  && make -f Makefile.debian ${BIGINT_GRACEFUL_FLAG} static \
  && make -f Makefile.debian install \
  && strip /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so
RUN rm -rf /root/.vpython_cipd_cache /root/.vpython-root \
  && apt-get clean \
  && apt-get remove -y ${buildDependencies} \
  && apt-get autoremove -y \
  && rm -rf /tmp/build /var/lib/apt/lists/*


FROM postgres:14.5

ARG PG_MAJOR=14
ARG PLV8_VERSION=3.0.0

RUN mkdir -p /usr/lib/postgresql/${PG_MAJOR}/lib/

RUN runtimeDependencies="libc++1" \
  && apt-get update \
  && apt-get install -y --no-install-recommends ${runtimeDependencies}

COPY --from=plv8_build /usr/share/postgresql/${PG_MAJOR}/extension/ /usr/share/postgresql/${PG_MAJOR}/extension/
COPY --from=plv8_build /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so

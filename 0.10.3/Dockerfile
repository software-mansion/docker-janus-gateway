FROM ubuntu:bionic AS build
ENV DEBIAN_FRONTEND=noninteractive

# Create workdir
RUN mkdir /build

# Install global build dependencies 
RUN \
  apt-get update && \
  apt-get install -y \
    git \
    pkg-config \
    libtool \
    automake

# Install build dependencies of libnice
RUN \
  apt-get update && \
  apt-get install -y \
	  libssl-dev \
    libglib2.0-dev \
    gtk-doc-tools

# Build libnice from sources as one shipped with ubuntu:bionic is too old, at least 0.1.15 is required
RUN \
  cd /build && \
  git clone --branch 0.1.16 https://gitlab.freedesktop.org/libnice/libnice.git && \
  cd libnice && \
  sh ./autogen.sh && \
  ./configure --prefix=/usr/local && \
  make -j$(nproc) && \
  make install

# Install build dependencies of libsrtp
RUN \
  apt-get update && \
  apt-get install -y \
	  libssl-dev

# Build libsrtp from sources as one shipped with ubuntu:bionic does not support AES-GCM profiles
# This needs to use /usr or /usr/local as a prefix, see https://github.com/meetecho/janus-gateway/issues/2019
RUN \
  cd /build && \
  git clone --branch v2.3.0 https://github.com/cisco/libsrtp.git && \
  cd libsrtp && \
  ./configure --prefix=/usr/local --enable-openssl && \
  make -j$(nproc) shared_library && \
  make install 

# Build usrsctp from sources
RUN \
  cd /build && \
  git clone https://github.com/sctplab/usrsctp && \
  cd usrsctp && \
  git reset --hard 579e6dea765c593acaa8525f6280b85868c866fc && \
  ./bootstrap && \
  ./configure --prefix=/usr/local && \ 
  make -j$(nproc) && \ 
  make install

# Install build dependencies of libwebsockets
RUN \
  apt-get update && \
  apt-get install -y \
	  cmake

# Build libwebsockets from sources as one shipped with ubuntu:bionic is too old, at least 2.4 is required
RUN \
  cd /build && \
  git clone --branch v2.4-stable https://libwebsockets.org/repo/libwebsockets && \
  cd libwebsockets && \
  mkdir build && \
  cd build && \
  cmake -DLWS_MAX_SMP=1 -DCMAKE_INSTALL_PREFIX:PATH=/usr/local -DCMAKE_C_FLAGS="-fpic" .. && \
  make -j$(nproc) && \
  make install

# Install build dependencies of librabbitmq-c
RUN \
  apt-get update && \
  apt-get install -y \
	  cmake

# Build librabbitmq-c from sources as one shipped with ubuntu:bionic is too old
RUN \
  cd /build && \
  git clone --branch v0.10.0 https://github.com/alanxz/rabbitmq-c.git && \
  cd rabbitmq-c && \
  git submodule init && \
  git submodule update && \
  mkdir build && \
  cd build && \
  cmake -DCMAKE_INSTALL_PREFIX=/usr/local .. && \
  make -j$(nproc) && \
  make install

# Install build dependencies of janus-gateway
RUN \
  apt-get update && \
  apt-get install -y \
	  libssl-dev \
    libglib2.0-dev \
    libmicrohttpd-dev \
    libjansson-dev \
    libsofia-sip-ua-dev \ 
	  libopus-dev \
    libogg-dev \
    libcurl4-openssl-dev \
    liblua5.3-dev \
	  libconfig-dev \
    gengetopt

# Build janus-gateway from sources
RUN \
  cd /build && \
  git clone --branch v0.10.3 https://github.com/meetecho/janus-gateway.git 
RUN cd /build/janus-gateway && \
  sh autogen.sh && \
  ./configure --prefix=/usr/local \
    --disable-all-transports \
    --enable-websockets \
    --enable-rabbitmq \
    --disable-all-handlers \
    --enable-rabbitmq-event-handler \
    --disable-all-loggers
RUN cd /build/janus-gateway && \
  make -j$(nproc) && \
  make install && \
  make configs

# Install dependencies of dockerize
RUN \
  apt-get update && \
  apt-get install -y \
    wget

# Install dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz


FROM ubuntu:bionic

# Install runtime dependencies of janus-gateway
RUN \
  apt-get update && \
  apt-get install -y \
	  libssl1.1 \
    libglib2.0-0 \
    libmicrohttpd12 \
    libjansson4 \
    libsofia-sip-ua-glib3 \ 
	  libopus0 \
    libogg0 \
    libcurl4 \
    liblua5.3-0 \
	  libconfig9 && \
 rm -rf /var/lib/apt/lists/*

# Copy all things that were built
COPY --from=build /usr/local /usr/local
ADD templates /templates

# Copy entrypoint
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Do not run as root unless necessary
RUN groupadd -g 999 app && useradd -r -u 999 -g app app

# Start the gateway
ENV LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib/x86_64-linux-gnu
CMD /entrypoint.sh

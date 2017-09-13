FROM ubuntu:16.04
MAINTAINER Fabian Köster <mail@fabian-koester.com>

EXPOSE 80

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    apache2 \
    build-essential \
    ca-certificates \
    cmake \
    curl \
    g++ \
    libapache2-mod-php \
    libboost-dev \
    libboost-filesystem-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libbz2-dev \
    libexpat1-dev \
    libgeos-dev \
    libgeos++-dev \
    libpq-dev \
    libproj-dev \
    libxml2-dev \
    php-db \
    php \
    php-intl \
    php-pgsql \
    php-pear \
    postgresql-client \
    postgresql-server-dev-9.5 \
    postgresql-9.5-postgis-2.2 \
    postgresql-contrib-9.5 \
    zlib1g-dev \
 && rm -rf /var/lib/apt/lists/*

# Install gosu
ENV GOSU_VERSION 1.9
RUN set -x \
    && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
    && curl -L -o /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
    && curl -L -o /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true

ARG WEBROOT=/var/www/html/
RUN groupadd -r nominatim && useradd -r -g nominatim nominatim
RUN mkdir -p ${WEBROOT} /app && chmod -R 755 ${WEBROOT} /app && chown -R nominatim:nominatim ${WEBROOT} /app

WORKDIR /app
ARG NOMINATIM_VERSION=3.0.1
ARG NOMINATIM_SHA256=30c19ba03bae0b23bd789758e41849da65b5a2104e69f072d4c754cb9b509c57
RUN gosu nominatim curl -o nominatim.tar.bz2 http://www.nominatim.org/release/Nominatim-${NOMINATIM_VERSION}.tar.bz2 \
  && echo "${NOMINATIM_SHA256} nominatim.tar.bz2" | sha256sum -c \
  && gosu nominatim tar -xf nominatim.tar.bz2 \
  && rm *.tar.bz2 \
  && gosu nominatim mkdir build \
  && cd build \
  && gosu nominatim cmake ../Nominatim-${NOMINATIM_VERSION} \
  && gosu nominatim make

RUN gosu nominatim cp -R build/website/ ${WEBROOT}/nominatim \
 && gosu nominatim cp -R build/settings ${WEBROOT}

COPY entrypoint.sh wait-for-it.sh ./
COPY local.php ./build/settings/local.php
COPY apache.conf /etc/apache2/sites-available/000-default.conf
COPY apache2-foreground /usr/local/bin

VOLUME /importdata
VOLUME /app/build/module
CMD /app/entrypoint.sh

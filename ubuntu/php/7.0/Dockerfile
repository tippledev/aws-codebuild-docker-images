# Copyright 2017-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#
# Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License.
# A copy of the License is located at
#
#    http://aws.amazon.com/asl/
#
# or in the "license" file accompanying this file.
# This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied.
# See the License for the specific language governing permissions and limitations under the License.
#

FROM ubuntu:14.04.5

ENV DOCKER_BUCKET="download.docker.com" \
    DOCKER_VERSION="17.09.0-ce" \
    DOCKER_CHANNEL="stable" \
    DOCKER_SHA256="a9e90a73c3cdfbf238f148e1ec0eaff5eb181f92f35bdd938fd7dab18e1c4647" \
    DIND_COMMIT="3b5fac462d21ca164b3778647420016315289034" \
    DOCKER_COMPOSE_VERSION="1.16.1"

# Building git from source code:
#   Ubuntu's default git package is built with broken gnutls. Rebuild git with openssl.
##########################################################################
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       wget=1.15-* python=2.7.5-* python2.7-dev=2.7.6-* fakeroot=1.20-* ca-certificates \
       tar=1.27.1-* gzip=1.6-* zip=3.0-* autoconf=2.69-* automake=1:1.14.1-* \
       bzip2=1.0.6-* file=1:5.14-* g++=4:4.8.2-* gcc=4:4.8.2-* imagemagick=8:6.7.7.10-* \
       libbz2-dev=1.0.6-* libc6-dev=2.19-* libcurl4-openssl-dev=7.35.0-* libdb-dev=1:5.3.21~* \
       libevent-dev=2.0.21-stable-* libffi-dev=3.1~rc1+r3.0.13-* libgeoip-dev=1.6.0-* libglib2.0-dev=2.40.2-* \
       libjpeg-dev=8c-* libkrb5-dev=1.12+dfsg-* liblzma-dev=5.1.1alpha+20120614-* \
       libmagickcore-dev=8:6.7.7.10-* libmagickwand-dev=8:6.7.7.10-* libmysqlclient-dev=5.5.60-* \
       libncurses5-dev=5.9+20140118-* libpng12-dev=1.2.50-* libpq-dev=9.3.22-* libreadline-dev=6.3-* \
       libsqlite3-dev=3.8.2-* libssl-dev=1.0.1f-* libtool=2.4.2-* libwebp-dev=0.4.0-* \
       libxml2-dev=2.9.1+dfsg1-* libxslt1-dev=1.1.28-* libyaml-dev=0.1.4-* make=3.81-* \
       patch=2.7.1-* xz-utils=5.1.1alpha+20120614-* zlib1g-dev=1:1.2.8.dfsg-* unzip=6.0-* curl=7.35.0-* \
       e2fsprogs=1.42.9-* iptables=1.4.21-* xfsprogs=3.1.9ubuntu2 xz-utils=5.1.1alpha+20120614-* \
    && apt-get install -y -qq less=458-* groff=1.22.2-* \
    && apt-get -qy build-dep git=1:1.9.1 \
    && apt-get -qy install libcurl4-openssl-dev=7.35.0-* git-man=1:1.9.1-* liberror-perl=0.17-* \
    && mkdir -p /usr/src/git-openssl \
    && cd /usr/src/git-openssl \
    && apt-get source git=1:1.9.1 \
    && cd $(find -mindepth 1 -maxdepth 1 -type d -name "git-*") \
    && sed -i -- 's/libcurl4-gnutls-dev/libcurl4-openssl-dev/' ./debian/control \
    && sed -i -- '/TEST\s*=\s*test/d' ./debian/rules \
    && dpkg-buildpackage -rfakeroot -b \
    && find .. -type f -name "git_*ubuntu*.deb" -exec dpkg -i \{\} \; \
    && rm -rf /usr/src/git-openssl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Install Docker
RUN set -x \
    && curl -fSL "https://${DOCKER_BUCKET}/linux/static/${DOCKER_CHANNEL}/x86_64/docker-${DOCKER_VERSION}.tgz" -o docker.tgz \
    && echo "${DOCKER_SHA256} *docker.tgz" | sha256sum -c - \
    && tar --extract --file docker.tgz --strip-components 1  --directory /usr/local/bin/ \
    && rm docker.tgz \
    && docker -v \
# set up subuid/subgid so that "--userns-remap=default" works out-of-the-box
    && addgroup dockremap \
    && useradd -g dockremap dockremap \
    && echo 'dockremap:165536:65536' >> /etc/subuid \
    && echo 'dockremap:165536:65536' >> /etc/subgid \
    && wget "https://raw.githubusercontent.com/docker/docker/${DIND_COMMIT}/hack/dind" -O /usr/local/bin/dind \
    && curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/dind /usr/local/bin/docker-compose \
# Ensure docker-compose works
    && docker-compose version

# Install dependencies by all python images equivalent to buildpack-deps:jessie
# on the public repos.

RUN wget "https://bootstrap.pypa.io/get-pip.py" -O /tmp/get-pip.py \
    && python /tmp/get-pip.py \
    && pip install awscli \
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME /var/lib/docker

COPY dockerd-entrypoint.sh /usr/local/bin/

ENV GPG_KEYS 1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763 6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3
ENV SRC_DIR="/usr/src" \
    PHP_VERSION=7.0.27 \
    PHP_DOWNLOAD_SHA="4b2bc823e806dbf7b62fe0b92b0d14b0c6e03f88c3fc5d96278416c54ce11f6c" \
    PHPPATH="/php" \
    PHP_INI_DIR="/usr/local/etc/php" \
    PHP_CFLAGS="-fstack-protector -fpic -fpie -O2" \
    PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ENV PHP_SRC_DIR="$SRC_DIR/php" \
    PHP_CPPFLAGS="$PHP_CFLAGS" \
    PHP_URL="https://secure.php.net/get/php-$PHP_VERSION.tar.xz/from/this/mirror" \
    PHP_ASC_URL="https://secure.php.net/get/php-$PHP_VERSION.tar.xz.asc/from/this/mirror"

# Install PHP
RUN set -xe; \
    mkdir -p $SRC_DIR; \
    cd $SRC_DIR; \
    wget -O php.tar.xz "$PHP_URL"; \
    echo "$PHP_DOWNLOAD_SHA *php.tar.xz" | sha256sum -c -; \
    wget -O php.tar.xz.asc "$PHP_ASC_URL"; \
    export GNUPGHOME="$(mktemp -d)"; \
    for key in $GPG_KEYS; do \
        ( gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" \
          || gpg --keyserver pgp.mit.edu --recv-keys "$key" \
          || gpg --keyserver keyserver.pgp.com --recv-keys "$key" ); \
    done; \
    gpg --batch --verify php.tar.xz.asc php.tar.xz; \
    rm -rf "$GNUPGHOME"; \

    set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends libedit-dev=3.1-*; \
    rm -rf /var/lib/apt/lists/*; \
    apt-get clean; \

    export \
        CFLAGS="$PHP_CFLAGS" \
        CPPFLAGS="$PHP_CPPFLAGS" \
        LDFLAGS="$PHP_LDFLAGS" \
    ; \
    mkdir -p $PHP_SRC_DIR; \
    tar -Jxf $SRC_DIR/php.tar.xz -C $PHP_SRC_DIR --strip-components=1; \
    cd $SRC_DIR/php; \
    gnuArch="$(dpkg-architecture -qDEB_BUILD_GNU_TYPE)"; \
    debMultiarch="$(dpkg-architecture -qDEB_BUILD_MULTIARCH)"; \

    # https://bugs.php.net/bug.php?id=74125
    if [ ! -d /usr/include/curl ]; then \
        ln -sT "/usr/include/$debMultiarch/curl" /usr/local/include/curl; \
    fi; \
    ./configure \
        --build="$gnuArch" \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        --disable-cgi \
    # --enable-ftp is included here because ftp_ssl_connect() needs ftp to be compiled statically (see https://github.com/docker-library/php/issues/236)
        --enable-ftp \
    # --enable-mbstring is included here because otherwise there's no way to get pecl to use it properly (see https://github.com/docker-library/php/issues/195)
        --enable-mbstring \
    # --enable-mysqlnd is included here because it's harder to compile after the fact than extensions are (since it's a plugin for several extensions, not an extension in itself)
        --enable-mysqlnd \
    # https://wiki.php.net/rfc/argon2_password_hash (7.2+)
        --with-password-argon2 \
        --with-curl \
        --with-libedit \
        --with-openssl \
        --with-zlib \
    # bundled pcre does not support JIT on s390x

    # https://manpages.debian.org/stretch/libpcre3-dev/pcrejit.3.en.html#AVAILABILITY_OF_JIT_SUPPORT
    $(test "$gnuArch" = 's390x-linux-gnu' && echo '--without-pcre-jit') \
        --with-libdir="lib/$debMultiarch" \
    ${PHP_EXTRA_CONFIGURE_ARGS:-} \
    ; \
    make -j "$(nproc)"; \
    make test; \
    make install; \
    find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; \
    make clean; \
    cd /; \
    rm -rf $PHP_SRC_DIR; \

    # reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
    find /usr/local -type f -executable -exec ldd '{}' ';' \
        | awk '/=>/ { print $(NF-1) }' \
        | sort -u \
        | xargs -r dpkg-query --search \
        | cut -d: -f1 \
        | sort -u \
        | xargs -r apt-mark manual \
    ; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    php --version; \
    pecl update-channels; \
    rm -rf /tmp/pear ~/.pearrc; \

    # Increase the memory size, default is 128M
    mkdir "$PHP_INI_DIR"; \
    mkdir "$PHP_INI_DIR/conf.d"; \
    touch "$PHP_INI_DIR/conf.d/memory.ini" \
    && echo "memory_limit = 1G;" >> "$PHP_INI_DIR/conf.d/memory.ini";

ENV PATH="$PHPPATH/bin:/usr/local/php/bin:$PATH"

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

WORKDIR $PHPPATH

ENTRYPOINT ["dockerd-entrypoint.sh"]

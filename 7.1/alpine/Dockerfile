FROM php:7.1-alpine 

# persistent / runtime deps
ENV PHPIZE_DEPS \
		autoconf \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkgconf \
		re2c
RUN apk add --no-cache --virtual .persistent-deps \
		ca-certificates \
		curl \
		tar \
		xz
		

# Apply stack smash protection to functions using local buffers and alloca()
# Make PHP's main executable position-independent (improves ASLR security mechanism, and has no performance impact on x86_64)
# Enable optimization (-O2)
# Enable linker optimization (this sorts the hash buckets to improve cache locality, and is non-default)
# Adds GNU HASH segments to generated executables (this is used if present, and is much faster than sysv hash; in this configuration, sysv hash is also generated)
# https://github.com/docker-library/php/issues/272
ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ENV PHPGEARMAN_URL="https://github.com/wcgallego/pecl-gearman/archive/gearman-2.0.3.tar.gz"

COPY docker-php-source /usr/local/bin/
COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

RUN set -xe \
	&& apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		curl-dev \
		libedit-dev \
		libxml2-dev \
		openssl-dev \
		sqlite-dev \
	\
	&& export CFLAGS="$PHP_CFLAGS" \
		CPPFLAGS="$PHP_CPPFLAGS" \
                LDFLAGS="$PHP_LDFLAGS" \
        && docker-php-ext-install soap
	\
	&& mkdir -p /usr/src \
	&& cd /usr/src \
	\
	&& wget -O phpgearman.tar.gz "$PHPGEARMAN_URL" \
	&& mkdir -p /usr/src/phpgearman
	&& tar -xzf /usr/src/phpgearman.tar.gz -C /usr/src/phpgearman
	&& cd /usr/src/phpgearman
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
	&& make -j "$(nproc)" \
	&& make install \
	&& make clean \
	&& cd / \
	&& rm -rf /usr/src/phpgearman \
	\
	&& apk del .build-deps


ENTRYPOINT ["docker-php-entrypoint"]
CMD ["php", "-a"]

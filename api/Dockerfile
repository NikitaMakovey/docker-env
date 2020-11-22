ARG PHP_VERSION=7.4

FROM php:${PHP_VERSION}-fpm-alpine

RUN apk add --no-cache \
		acl \
        fcgi \
        file \
        gettext \
        git \
        vim \
        jpegoptim \
        optipng \
        pngquant \
	;

ARG APCU_VERSION=5.1.18

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		icu-dev \
		libzip-dev \
		postgresql-dev \
		zlib-dev \
		libxml2-dev \
		imagemagick-dev \
        freetype-dev \
        libpng-dev \
        libjpeg-turbo-dev \
	; \
	\
	docker-php-ext-configure zip; \
	docker-php-ext-configure gd --with-freetype --with-jpeg; \
	docker-php-ext-install -j$(nproc) \
		intl \
		pdo_pgsql \
		zip \
		xml \
		soap \
		exif \
		gd \
	; \
	pecl install \
		apcu-${APCU_VERSION} \
		redis \
		imagick \
	; \
	pecl clear-cache; \
	docker-php-ext-enable \
		apcu \
		opcache \
		redis \
		imagick \
	; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache --virtual .api-phpexts-rundeps $runDeps; \
	\
	apk del .build-deps

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN ln -s $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini
COPY ../docker/php/conf.d/custom.dev.ini $PHP_INI_DIR/conf.d/custom.ini

ENV COMPOSER_ALLOW_SUPERUSER=1
ENV PATH="${PATH}:/root/.composer/vendor/bin"

WORKDIR /srv/api

ARG APP_ENV=local

COPY . ./

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint

ENTRYPOINT ["docker-entrypoint"]
CMD ["php-fpm"]

FROM php:7.4-fpm

# persistent dependencies
RUN set -eux; \
	apt-get update; \
	apt-get install -y nginx; \
	apt-get install -y --no-install-recommends \
# Ghostscript is required for rendering PDF previews
		ghostscript \
	; \
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libicu-dev \
		libjpeg-dev \
		libmagickwand-dev \
		libpng-dev \
		libwebp-dev \
		libzip-dev \
	; \
	\
	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg \
		--with-webp \
	; \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		exif \
		gd \
		intl \
		mysqli \
		pdo_mysql \
		zip \
	; \
# https://pecl.php.net/package/imagick
	pecl install imagick-3.6.0 redis; \
	docker-php-ext-enable imagick redis mysqli; \
	rm -r /tmp/pear; \
	\
# some misbehaving extensions end up outputting to stdout 🙈 (https://github.com/docker-library/wordpress/issues/669#issuecomment-993945967)
	out="$(php -r 'exit(0);')"; \
	[ -z "$out" ]; \
	err="$(php -r 'exit(0);' 3>&1 1>&2 2>&3)"; \
	[ -z "$err" ]; \
	\
	extDir="$(php -r 'echo ini_get("extension_dir");')"; \
	[ -d "$extDir" ]; \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$extDir"/*.so \
		| awk '/=>/ { so = $(NF-1); if (index(so, "/usr/local/") == 1) { next }; gsub("^/(usr/)?", "", so); print so }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*; \
	\
	! { ldd "$extDir"/*.so | grep 'not found'; }; \
# check for output like "PHP Warning:  PHP Startup: Unable to load dynamic library 'foo' (tried: ...)
	err="$(php --version 3>&1 1>&2 2>&3)"; \
	[ -z "$err" ]

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN set -eux; \
	docker-php-ext-enable opcache; \
	{ \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini
# https://wordpress.org/support/article/editing-wp-config-php/#configure-error-logging
RUN { \
# https://www.php.net/manual/en/errorfunc.constants.php
# https://github.com/docker-library/wordpress/issues/420#issuecomment-517839670
		echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
		echo 'display_errors = Off'; \
		echo 'display_startup_errors = Off'; \
		echo 'log_errors = On'; \
		echo 'error_log = /dev/stderr'; \
		echo 'log_errors_max_len = 1024'; \
		echo 'ignore_repeated_errors = On'; \
		echo 'ignore_repeated_source = Off'; \
		echo 'html_errors = Off'; \
	} > /usr/local/etc/php/conf.d/error-logging.ini

RUN { \
		echo '#!/usr/bin/env bash'; \
		echo ''; \
		echo 'run_user_name="${APACHE_RUN_USER:-www-data}"'; \
		echo 'run_group_name="${APACHE_RUN_GROUP:-www-data}"'; \
		echo 'run_user_id="${APACHE_RUN_UID:-1000}"'; \
		echo 'run_group_id="${APACHE_RUN_GID:-1000}"'; \
		echo ''; \
		echo 'if [ -z "$(getent group ${run_group_name})" ]; then'; \
		echo '    groupadd -g ${run_group_id} ${run_group_name}'; \
		echo 'fi'; \
		echo ''; \
		echo 'if [ -z "$(getent passwd ${run_user_name})" ]; then'; \
		echo '    useradd -u ${run_user_id} -g ${run_group_id} -s /bin/bash -m ${run_user_name}'; \
		echo 'fi'; \
		echo ''; \
		echo 'exec "$@"'; \
	} > /usr/local/bin/docker-entrypoint.sh \
	&& chmod +x /usr/local/bin/docker-entrypoint.sh

RUN curl -fSL 'http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz' -o ioncube.tar.gz \
	&& tar -xf ioncube.tar.gz -C /opt/ --strip-components=1 ioncube/ioncube_loader_lin_7.4.so \
	&& rm -r ioncube.tar.gz \
	&& mv /opt/ioncube_loader_lin_7.4.so /opt/ioncube.so \
	&& echo "zend_extension=/opt/ioncube.so" > /usr/local/etc/php/conf.d/00-ioncube.ini

RUN curl -sS https://getcomposer.org/installer | php -- \
	--install-dir=/usr/local/bin --filename=composer \
    && composer self-update --2

ARG MULTIRUN_URL=https://github.com/nicolas-van/multirun/releases/download/1.1.3/multirun-x86_64-linux-gnu-1.1.3.tar.gz
RUN curl -L $MULTIRUN_URL | tar -xzv -C /usr/sbin/

VOLUME /var/www/html

# Configure Nginx
COPY nginx-default.conf /etc/nginx/sites-available/default

EXPOSE 80

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["multirun", "-v", "nginx -g \"daemon off;\"", "php-fpm -F"]

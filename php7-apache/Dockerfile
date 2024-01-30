FROM php:7.4-apache

RUN set -eux; \
	apt-get update && \
	apt-get install -y --no-install-recommends \
		ghostscript \
		libfreetype6-dev \
		libicu-dev \
		libjpeg-dev \
		libmagickwand-dev \
		libpng-dev \
		libwebp-dev \
		libzip-dev \
		less mariadb-client \
	&& \
	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg \
		--with-webp \
	&& \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		exif \
		gd \
		intl \
		mysqli \
		opcache \
		pdo pdo_mysql \
		zip \
	&& \
	pecl install \
		imagick \
		redis \
	&& \
	docker-php-ext-enable \
		imagick \
		opcache \
		redis \
	&& \
	rm -rf /var/lib/apt/lists/* /tmp/pear; \
	! { ldd "$(php -r 'echo ini_get(\"extension_dir\");')/*.so" | grep 'not found'; } && \
	{ \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini && \
	{ \
		echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
		echo 'display_errors = Off'; \
		echo 'display_startup_errors = Off'; \
		echo 'log_errors = On'; \
		echo 'error_log = /dev/stderr'; \
		echo 'log_errors_max_len = 1024'; \
		echo 'ignore_repeated_errors = On'; \
		echo 'ignore_repeated_source = Off'; \
		echo 'html_errors = Off'; \
	} > /usr/local/etc/php/conf.d/error-logging.ini && \
	a2enmod rewrite headers expires remoteip && \
	{ \
		echo 'RemoteIPHeader X-Forwarded-For'; \
		echo 'RemoteIPInternalProxy 10.0.0.0/8'; \
		echo 'RemoteIPInternalProxy 172.16.0.0/12'; \
		echo 'RemoteIPInternalProxy 192.168.0.0/16'; \
		echo 'RemoteIPInternalProxy 169.254.0.0/16'; \
		echo 'RemoteIPInternalProxy 127.0.0.0/8'; \
	} > /etc/apache2/conf-available/remoteip.conf && \
	a2enconf remoteip && \
	find /etc/apache2 -type f -name '*.conf' -exec sed -ri 's/([[:space:]]*LogFormat[[:space:]]+"[^"]*)%h([^"]*")/\1%a\2/g' '{}' +

RUN curl -fSL 'http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz' -o ioncube.tar.gz \
		&& tar -xf ioncube.tar.gz -C /opt/ --strip-components=1 ioncube/ioncube_loader_lin_7.4.so \
		&& rm -r ioncube.tar.gz \
		&& mv /opt/ioncube_loader_lin_7.4.so /opt/ioncube.so \
		&& echo "zend_extension=/opt/ioncube.so" > /usr/local/etc/php/conf.d/00-ioncube.ini \
	&& \
	curl -sS https://getcomposer.org/installer | php -- \
		--install-dir=/usr/local/bin --filename=composer \
		&& composer self-update --2 \
	&& \
	curl -L -o /usr/local/bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
		&& chmod +x /usr/local/bin/wp \
	&& \
	GOCROND_VERSION=23.12.0 \
		&& curl -L -o /usr/local/bin/go-crond https://github.com/webdevops/go-crond/releases/download/${GOCROND_VERSION}/go-crond.linux.amd64 \
		&& chmod +x /usr/local/bin/go-crond \
	&& \
	{ \
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
		echo '[ -e /etc/crontab ] && /usr/local/bin/go-crond /etc/crontab &'; \
		echo ''; \
		echo 'if [ -n "$APACHE_DOCUMENT_ROOT" ]; then'; \
		echo '    sed -i "s#DocumentRoot\s\+.*#DocumentRoot $APACHE_DOCUMENT_ROOT#" /etc/apache2/sites-available/*.conf'; \
		echo 'fi'; \
		echo ''; \
		echo 'exec "$@"'; \
	} > /usr/local/bin/docker-entrypoint.sh \
	&& chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["apache2-foreground"]

FROM wordpress:php7.4-apache

RUN set -eux; \
    pecl install -o -f redis; \
    docker-php-ext-enable redis; \
    rm -r /tmp/pear; \
    a2enmod headers; \
    docker-php-ext-install -j "$(nproc)" \
        pdo pdo_mysql mysqli \
	; \
    docker-php-ext-enable mysqli

# ioncube loader
RUN curl -fSL 'http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz' -o ioncube.tar.gz \
    && mkdir -p ioncube \
    && tar -xf ioncube.tar.gz -C ioncube --strip-components=1 \
    && rm ioncube.tar.gz \
    && mv ioncube/ioncube_loader_lin_7.4.so /var/www/ioncube_loader_lin_7.4.so \
    && rm -r ioncube \
    && echo "zend_extension = /var/www/ioncube_loader_lin_7.4.so" > \
    /usr/local/etc/php/conf.d/10-ioncube-loader.ini

# composer
RUN curl -S https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer self-update

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
    } > /usr/local/bin/docker-entrypoint.sh



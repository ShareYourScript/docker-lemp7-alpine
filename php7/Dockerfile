FROM php:7.0-fpm-alpine

# docker-entrypoint.sh dependencies
RUN apk add --no-cache \
# in theory, docker-entrypoint.sh is POSIX-compliant, but priority is a working, consistent image
    bash \
    zip \
    unzip \
# BusyBox sed is not sufficient for some of our sed expressions
    sed

# install the PHP extensions we need
RUN set -ex; \
  \
  apk add --no-cache --virtual .build-deps \
    libjpeg-turbo-dev \
    libpng-dev \
  ; \
  \
  docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
  docker-php-ext-install gd mysqli opcache zip; \
  \
  runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
      | tr ',' '\n' \
      | sort -u \
      | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
  )"; \
  apk add --virtual .wordpress-phpexts-rundeps $runDeps; \
  apk del .build-deps

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
    echo 'opcache.memory_consumption=256'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=2'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
  } > /usr/local/etc/php/conf.d/opcache-recommended.ini

VOLUME /var/www/html

RUN set -ex; \
  #chown -R xfs:xfs /var/www/html; \
  #sed -i 's/user = www-data/user = xfs/g' /usr/local/etc/php-fpm.d/www.conf; \
  #sed -i 's/group = www-data/user = xfs/g' /usr/local/etc/php-fpm.d/www.conf; \
  echo "pm.process_idle_timeout=10s" >> /usr/local/etc/php-fpm.d/www.conf; \
  echo "pm.max_requests=500" >> /usr/local/etc/php-fpm.d/www.conf; \
  echo "request_terminate_timeout = 300" >> /usr/local/etc/php-fpm.d/www.conf; \
  echo "php_admin_value[post_max_size] = 20M" >> /usr/local/etc/php-fpm.d/www.conf; \
  echo "php_admin_value[upload_max_filesize] = 20M" >> /usr/local/etc/php-fpm.d/www.conf; \
  echo "php_admin_value[memory_limit] = 256M" >> /usr/local/etc/php-fpm.d/www.conf

CMD ["php-fpm"]

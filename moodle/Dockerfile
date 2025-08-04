ARG PHP_VERSION=8.4
FROM php:${PHP_VERSION}-apache

ARG MOODLE_VERSION=5.0.1
ENV MOODLE_URL=https://download.moodle.org/download.php/direct/stable500/moodle-${MOODLE_VERSION}.tgz \
    MOODLE_PATH=/var/www/html

# Variable per al proxy de confiança (passada via .env)
ARG TRUSTED_PROXY
ENV TRUSTED_PROXY=${TRUSTED_PROXY}

# Instal·lar dependències i eines necessàries
RUN apt-get update && apt-get install -y \
    cron nano git curl zip unzip \
    libpng-dev libjpeg-dev libfreetype6-dev \
    libicu-dev libxml2-dev libzip-dev libxslt-dev \
    pkg-config libssl-dev gcc make autoconf \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd intl zip soap mysqli pdo pdo_mysql exif \
    && pecl install redis \
    && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Activar mòduls necessaris
RUN a2enmod rewrite remoteip

# Configuració mod_remoteip usant variable d'entorn
RUN echo "RemoteIPHeader X-Forwarded-For" > /etc/apache2/conf-available/remoteip.conf \
    && echo "RemoteIPTrustedProxy ${TRUSTED_PROXY}" >> /etc/apache2/conf-available/remoteip.conf \
    && a2enconf remoteip

# Descarregar i instal·lar Moodle
RUN set -e \
    && mkdir -p $MOODLE_PATH \
    && curl -fSL $MOODLE_URL -o /tmp/moodle.tgz \
    && tar -xzf /tmp/moodle.tgz --strip-components=1 -C $MOODLE_PATH \
    && rm /tmp/moodle.tgz

# Instal·lar el plugin Redis Lock
RUN cd $MOODLE_PATH/local \
    && git clone https://github.com/open-lms-open-source/moodle-local_redislock redislock \
    && chown -R www-data:www-data redislock

# Ajustar permisos de Moodle i moodledata
RUN chown -R www-data:www-data $MOODLE_PATH \
    && mkdir -p /var/www/moodledata \
    && chown -R www-data:www-data /var/www/moodledata

# Configuració PHP i Opcache
RUN { \
    echo "post_max_size = 128M"; \
    echo "upload_max_filesize = 128M"; \
    echo "max_execution_time = 300"; \
    echo "memory_limit = 512M"; \
    echo "max_input_vars = 5000"; \
    echo "opcache.enable=1"; \
    echo "opcache.memory_consumption=256"; \
    echo "opcache.max_accelerated_files=10000"; \
    echo "opcache.revalidate_freq=60"; \
    echo "opcache.validate_timestamps=1"; \
} > /usr/local/etc/php/conf.d/moodle.ini

EXPOSE 80
CMD ["apache2-foreground"]

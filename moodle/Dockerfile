FROM php:8.2-apache

# Variables
ENV MOODLE_VERSION=4.3 \
    MOODLE_URL=https://download.moodle.org/download.php/direct/stable500/moodle-5.0.1.tgz \
    MOODLE_PATH=/var/www/html

# Instalar dependencias del sistema y extensiones PHP necesarias
RUN apt-get update && apt-get install -y \
    libpng-dev libjpeg-dev libfreetype6-dev \
    libicu-dev libxml2-dev libzip-dev libxslt-dev \
    zip unzip git curl \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd intl zip soap mysqli pdo pdo_mysql \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Habilitar mod_rewrite en Apache
RUN a2enmod rewrite

# Descargar Moodle
RUN curl -L $MOODLE_URL | tar -xz --strip-components=1 -C $MOODLE_PATH

# Ajustar permisos
RUN chown -R www-data:www-data $MOODLE_PATH

# Configuración PHP recomendada
RUN { \
    echo "post_max_size = 128M"; \
    echo "upload_max_filesize = 128M"; \
    echo "max_execution_time = 300"; \
} > /usr/local/etc/php/conf.d/moodle.ini

EXPOSE 80

CMD ["apache2-foreground"]

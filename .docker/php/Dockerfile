FROM php:8.4-fpm

# Start as root
USER root

# Install dependencies for PHP and Node.js
RUN apt-get update \
    && apt-get install -y \
        git \
        unzip \
        zip \
        curl \
        gnupg2 \
        lsb-release \
        libcurl4-openssl-dev \
        libmagickwand-dev

# Install NVM and Node.js 20
ENV NVM_DIR=/usr/local/nvm
ENV NODE_VERSION=20
RUN mkdir -p $NVM_DIR \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default \
    # Install Yarn using npm
    && npm install -g yarn

# Add NVM to PATH
ENV NODE_PATH=$NVM_DIR/versions/node/v$NODE_VERSION/lib/node_modules
ENV PATH=$NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

# Install PHP extensions
RUN docker-php-ext-install \
    mysqli \
    pdo \
    pdo_mysql \
    curl \
    exif \
    fileinfo \
    && (php -m | grep -i imagick || pecl install imagick) \
    && docker-php-ext-enable exif fileinfo imagick

# Clean up to reduce image size
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && rm -f /var/log/lastlog /var/log/faillog

# Add a non-root user to prevent files being created with root permissions on the host machine
ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}

RUN groupmod -o -g ${PGID} www-data \
    && usermod -o -u ${PUID} -g www-data www-data

# Set permissions for Composer cache and project directory
RUN chown -R ${PUID}:${PGID} /usr/local/bin \
    && chmod -R 775 /usr/local/bin

# Install a specific version of Composer
ENV COMPOSER_VERSION=2.9.2
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=$COMPOSER_VERSION \
    && mkdir -p /var/www/.composer \
    && chown -R ${PUID}:${PGID} /var/www/.composer \
    && chmod -R 775 /var/www/.composer/

# Install WP CLI
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
    && chmod +x wp-cli.phar \
    && mv wp-cli.phar /usr/local/bin/wp

# Set working directory
WORKDIR /var/www

# Create entrypoint to set log permissions
RUN echo '#!/bin/bash\n\
mkdir -p /var/log\n\
chown -R www-data:www-data /var/log\n\
chmod -R 755 /var/log\n\
exec php-fpm\n\
' > /docker-entrypoint.sh && chmod +x /docker-entrypoint.sh

# Expose port 9000 and start php-fpm server
EXPOSE 9000
ENTRYPOINT ["/docker-entrypoint.sh"]

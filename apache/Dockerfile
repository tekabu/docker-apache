FROM php:7.3-apache

# ── System dependencies ───────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
        libpng-dev \
        libjpeg62-turbo-dev \
        libfreetype6-dev \
        libzip-dev \
        libicu-dev \
        libonig-dev \
        curl \
    && rm -rf /var/lib/apt/lists/*

# ── PHP extensions required by CodeIgniter 3 ─────────────────────────────────
RUN docker-php-ext-configure gd \
        --with-freetype-dir=/usr \
        --with-jpeg-dir=/usr \
    && docker-php-ext-install -j"$(nproc)" \
        pdo_mysql \
        mysqli \
        gd \
        intl \
        zip \
        mbstring \
        opcache

# ── Apache modules ────────────────────────────────────────────────────────────
RUN a2enmod rewrite headers expires deflate remoteip

# ── Security: hide server version ────────────────────────────────────────────
RUN { \
        echo 'ServerTokens Prod'; \
        echo 'ServerSignature Off'; \
    } >> /etc/apache2/conf-available/security.conf \
    && a2enconf security

# ── Replace default vhost ─────────────────────────────────────────────────────
RUN a2dissite 000-default.conf
COPY vhost.conf /etc/apache2/sites-available/app.conf
RUN a2ensite app.conf

# ── PHP production config ─────────────────────────────────────────────────────
COPY php.ini /usr/local/etc/php/conf.d/app.ini

# ── Permissions ───────────────────────────────────────────────────────────────
# The app dir is bind-mounted at runtime; ensure www-data can access it.
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data \
    && mkdir -p /var/www/public_html \
    && chown www-data:www-data /var/www/public_html

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
    CMD curl -f http://localhost/test.html || exit 1

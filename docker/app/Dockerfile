FROM php:8.1-apache

# iniファイルの読込
COPY ./docker/app/php.ini /usr/local/etc/php/

# Composerインストール
COPY --from=composer:2.6 /usr/bin/composer /usr/bin/composer

# Middlewareインストール
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    npm \
    gnupg \
    ca-certificates \
    curl \
    git \
    vim \
    zip \
    unzip \
    libzip-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libonig-dev \
    libssl-dev \
    default-libmysqlclient-dev \
    libmagickwand-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# configファイルの読込
COPY ./docker/app/apache2/000-default.conf /etc/apache2/sites-available/
COPY ./docker/app/apache2/default-ssl.conf /etc/apache2/sites-available/
COPY ./docker/app/apache2/fqdn.conf /etc/apache2/conf-available/
RUN a2enconf fqdn \
    && a2enmod rewrite

# Apacheを再起動
RUN service apache2 restart

# pdo_mysql拡張モジュールのインストール（mysqlとの接続が可能に）
RUN docker-php-ext-install pdo_mysql mysqli

# gd拡張モジュールのインストール (-j$(nproc)=使用可能なCPU数を指定)
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

# ext-exif拡張機能のインストール
RUN docker-php-ext-install exif

# Imagickのインストール
RUN pecl install imagick \
    && docker-php-ext-enable imagick

# npmパッケージ / vite のインストール
RUN npm install npm@9.5.1 -g cross-env \
    && npm install -g vite

# Xdebugインストール
RUN pecl install xdebug-3.2.2 \
    && docker-php-ext-enable xdebug
RUN chmod 777 /var/log/xdebug.log

# laravelに必要なmodRewriteの有効化
RUN mv /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled
RUN /bin/sh -c a2enmod rewrite

EXPOSE 5173

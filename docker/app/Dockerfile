FROM php:8.1-apache

# iniファイルの読込
COPY ./docker/app/php.ini /usr/local/etc/php/

# Composerインストール
COPY --from=composer:2.6 /usr/bin/composer /usr/bin/composer

# Middlewareインストール
# 上から順：
# gnupg:                GNU Privacy Guard、暗号化・署名の為のパッケージ（dockerイメージ構築時に必要）
# ca-certificates:      これも認証関係
# curl:                 データの送受信
# git:                  Git
# vim:                  Vimエディタ
# zip:                  zip関連ユーティリティ
# unzip:                zip解凍ユーティリティ
# libzip-dev:           zipアーカイブの読取・作成・変更する開発ヘッダ
# libfreetype6-dev:     画像処理ライブラリ拡張モジュール；TrueTypeフォントサポート(GD2版)->pngを含むので、libpngは不要
# libjpeg62-turbo-dev:  画像処理ライブラリ拡張モジュール；JPEGサポート(GD2版)
# libpng-dev:           画像処理ライブラリ拡張モジュール；PNGサポート
# libonig-dev:          Oniguruma正規表現ライブラリ開発ヘッダ
# libssl-dev:           OpenSSLライブラリ関連ヘッダ
# default-libmysqlclient-dev:   MySQL開発ヘッダ
RUN apt-get update \
    && apt-get install -y \
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
    default-libmysqlclient-dev

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

# nodejs / npmパッケージのインストール
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && rm -rf node_modules \
    && apt-get install -y nodejs \
    && apt-get update
RUN npm install npm@9.5.1 -g cross-env
# webpackのインストール
RUN npm install webpack

# Xdebugインストール
RUN pecl install xdebug-3.2.2 \
    && docker-php-ext-enable xdebug
RUN chmod 777 /var/log/xdebug.log

# laravelに必要なmodRewriteの有効化
RUN mv /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled
RUN /bin/sh -c a2enmod rewrite


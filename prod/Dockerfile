FROM php:8.1-apache

ARG USERNAME=user
ARG GROUPNAME=user
ARG UID=1000
ARG GID=1000

RUN groupadd -g $GID $GROUPNAME && \
useradd -m -u $UID -g $GID $USERNAME && \
usermod -aG www-data $USERNAME

RUN apt-get update && apt-get install --assume-yes --no-install-recommends --quiet \
  unzip \
  vim \
  libicu-dev \
  && docker-php-ext-install \
  pdo_mysql \
  intl \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Composerをセットアップ
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Node.jsをセットアップ
RUN curl -SL https://deb.nodesource.com/setup_lts.x | bash
RUN apt-get install -y nodejs && \
  npm install -g npm@latest

#apacheの設定をコピー
COPY ./sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf

# headers : リクエストヘッダ、レスポンスヘッダを制御するモジュールの有効化
# => Laravelを動作させるにはmod_rewriteを有効にして、.htaccessが使える状態にする必要がある
# rewrite : URLの書き換えを行うモジュールを有効化
# expires : ブラウザのキャッシュの有効期限を指定するモジュールを有効化
RUN a2enmod headers rewrite expires
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini

WORKDIR /var/www/html

COPY ./src /var/www/html

# 実行ユーザーがstorageに書き込めるようにする。
RUN chown www-data:www-data storage/ -R \
&& chmod -R g+rwx storage/ -R \
&& composer install

USER $USERNAME

EXPOSE 80
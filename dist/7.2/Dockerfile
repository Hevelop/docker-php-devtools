FROM hevelop/php:7.2-cli-stretch

RUN usermod -u 1000 www-data
RUN groupmod -g 1000 www-data

# Install dependencies
RUN apt-get update \
  && apt-get install -y \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libxslt1-dev \
    mysql-client \
    sudo \
    curl

# Configure the gd library
RUN docker-php-ext-configure \
  gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

# Install required PHP extensions

RUN docker-php-ext-install \
  dom \
  gd \
  intl \
  mbstring \
  pdo_mysql \
  xsl \
  zip \
  soap \
  bcmath \
  pcntl \
  sockets && docker-php-ext-enable pcntl

# Install Xdebug (but don't enable)
RUN pecl install -o -f xdebug

# replace shell with bash so we can source files
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

RUN apt-get update \
  && apt-get install --no-install-recommends -y cmake make gcc libc6-dev

WORKDIR /tmp
ENV TINI_VERSION 0.18.0
ADD https://github.com/krallin/tini/archive/v${TINI_VERSION}.tar.gz ./
RUN tar -xf v${TINI_VERSION}.tar.gz \
  && cmake tini-${TINI_VERSION} \
  && make \
  && mv /tmp/tini /usr/local/sbin/

# update the repository sources list
# and install dependencies
RUN apt-get update \
    && apt-get install -y curl gnupg2 net-tools \
    && apt-get -y autoclean

RUN mkdir -p /var/www/.npm && chown -R www-data:www-data /var/www/.npm
RUN mkdir -p /var/www/.nvm && chown -R www-data:www-data /var/www/.nvm
RUN mkdir -p /var/www/.config && chown -R www-data:www-data /var/www/.config

# Install Node, NVM, NPM and Grunt
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - \
  	&& apt-get install -y nodejs build-essential \
    && curl https://raw.githubusercontent.com/creationix/nvm/v0.16.1/install.sh | sh \
    && npm i -g grunt-cli yarn

RUN apt-get update \
        && apt-get install -y apt-transport-https \
        && curl https://packages.sury.org/php/apt.gpg -o /etc/apt/trusted.gpg.d/php.gpg \
        && echo "deb https://packages.sury.org/php/ stretch main" > /etc/apt/sources.list.d/php.list \
        && rm -rf /var/lib/apt/lists/* && apt-get update \
        && apt-get install --yes libsodium-dev \
        && docker-php-ext-install sodium

COPY ./entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /var/www/deploy

CMD ["php", "-A"]

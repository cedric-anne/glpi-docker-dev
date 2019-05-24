FROM php:7.3-apache-stretch

# Workaround to make apache use same UID/GID as host user.
ARG HOST_GROUP_ID=1000
RUN groupmod --gid ${HOST_GROUP_ID} www-data
ARG HOST_USER_ID=1000
RUN usermod --uid ${HOST_USER_ID} www-data

# Make www-data user home persistent for cache purpose.
RUN mkdir --parents /home/www-data \
  && chown www-data:www-data /home/www-data \
  && usermod --home /home/www-data www-data
VOLUME /home/www-data

RUN apt-get update \
  \
  # Install APCU PHP extension.
  && pecl install apcu \
  && docker-php-ext-enable apcu \
  && echo "apc.enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini \
  \
  # Install exif extension.
  && docker-php-ext-install exif \
  \
  # Install gd PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libfreetype6-dev libjpeg-dev libpng-dev \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install gd \
  \
  # Install imap PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libc-client-dev libkrb5-dev \
  && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
  && docker-php-ext-install imap \
  \
  # Install intl PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libicu-dev \
  && docker-php-ext-configure intl \
  && docker-php-ext-install intl \
  \
  # Install ldap PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libldap2-dev \
  && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
  && docker-php-ext-install ldap \
  \
  # Install mysqli PHP extension.
  && docker-php-ext-install mysqli \
  \
  # Install PDO MySQL PHP extension.
  && docker-php-ext-install pdo pdo_mysql \
  \
  # Install soap PHP extension (required for some plugins).
  && apt-get install --assume-yes --no-install-recommends --quiet libxml2-dev \
  && docker-php-ext-install soap \
  \
  # Install xmlrpc PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libxml2-dev \
  && docker-php-ext-install xmlrpc \
  \
  # Install cron service.
  && apt-get install --assume-yes --no-install-recommends --quiet cron \
  \
  # Install composer.
  && curl --silent --show-error https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
  \
  # Install nodejs and npm.
  && apt-get install --assume-yes --no-install-recommends --quiet gnupg \
  && curl --silent --location https://deb.nodesource.com/setup_10.x | bash - \
  && apt-get install --assume-yes --no-install-recommends --quiet nodejs \
  \
  # Install git and zip used by composer when fetching dependencies.
  && apt-get install --assume-yes --no-install-recommends --quiet git unzip \
  \
  # Install ssmtp to forward emails dedicated mail container.
  && apt-get install --assume-yes --no-install-recommends --quiet ssmtp \
  \
  # Install gettext used for translation files.
  && apt-get install --assume-yes --no-install-recommends --quiet gettext \
  \
  # Install python, pip and setuptools
  && apt-get install --assume-yes --no-install-recommends --quiet python python-pip \
  && pip install --quiet setuptools \
  \
  # Install dependencies for plugin release tool
  && apt-get install --assume-yes --no-install-recommends --quiet gpg python-lxml \
  && pip install --quiet gitdb gitpython pygithub termcolor \
  \
  # Install transifex client
  && apt-get install --assume-yes --no-install-recommends --quiet transifex-client \
  \
  # Clean sources list
  && rm -rf /var/lib/apt/lists/*

# Copy files to container.
COPY ./files /

# Install GLPI crontab.
RUN crontab -u www-data /etc/cron.d/glpi

# Make startup script executable and executes it as default command.
RUN chmod u+x /opt/startup.sh
CMD /opt/startup.sh

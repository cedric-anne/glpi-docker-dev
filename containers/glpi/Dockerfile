ARG BASE_IMAGE=php:apache-buster

FROM $BASE_IMAGE

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
  && if [ $PHP_MAJOR_VERSION -eq "5" ]; then \
    # For PHP 5.x, install apcu 4.0.x using PECL util
    pecl install apcu-4.0.11 \
    && docker-php-ext-enable apcu \
  ; elif [ $PHP_MAJOR_VERSION -eq "7" ]; then \
    # For PHP 7.x, install latest apcu version using PECL util
    pecl install apcu \
    && docker-php-ext-enable apcu \
  ; else \
    # For PHP 8+, install from PECL archive (pecl util is not available)
    mkdir -p /usr/src/php/ext/apcu \
    && curl -LsfS https://pecl.php.net/get/apcu | tar xvz -C "/usr/src/php/ext/apcu" --strip-components 1 \
    && docker-php-ext-install apcu \
  ; fi \
  && echo "apc.enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini \
  \
  # Install exif extension.
  && docker-php-ext-install exif \
  \
  # Install gd PHP extension.
  # GD extension configuration parameters changed on PHP 7.4
  # see https://www.php.net/manual/en/image.installation.php#image.installation
  && apt-get install --assume-yes --no-install-recommends --quiet libfreetype6-dev libjpeg-dev libpng-dev \
  && PHP_MAJOR_VERSION="$(echo $PHP_VERSION | cut -d '.' -f 1)" \
  && PHP_MINOR_VERSION="$(echo $PHP_VERSION | cut -d '.' -f 2)" \
  && if [ $PHP_MAJOR_VERSION -le "5" ] || ([ $PHP_MAJOR_VERSION -eq "7" ] && [ $PHP_MINOR_VERSION -le "3" ]); then \
    # For PHP <= 5.x or PHP 7 <= 7.3, use old parameters
    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  ; else \
    docker-php-ext-configure gd --with-freetype --with-jpeg \
  ; fi \
  && docker-php-ext-install gd \
  \
  # Install imap PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libc-client-dev libkrb5-dev \
  && PHP_OPENSSL=yes docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
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
  # Install pcntl PHP extension (required for composer-require-checker).
  && docker-php-ext-install pcntl \
  \
  # Install PDO MySQL PHP extension.
  && docker-php-ext-install pdo pdo_mysql \
  \
  # Install soap PHP extension (required for some plugins).
  && apt-get install --assume-yes --no-install-recommends --quiet libxml2-dev \
  && docker-php-ext-install soap \
  \
  # Install XMLRPC PHP extension.
  && if [ $PHP_MAJOR_VERSION -lt "8" ]; then \
    # For PHP < 8.x, install bundled extension
    apt-get install --assume-yes --no-install-recommends --quiet libxml2-dev \
    && docker-php-ext-install xmlrpc \
  #; else \
    # For PHP 8+, install from PECL archive (pecl util is not available)
    # uncomment when a release will be available
    # mkdir -p /usr/src/php/ext/xmlrpc \
    # && curl -LsfS https://pecl.php.net/get/xmlrpc | tar xvz -C "/usr/src/php/ext/xmlrpc" --strip 1 \
    # && docker-php-ext-enable xmlrpc \
  ; fi \
  \
  # Enable apache mods.
  && a2enmod rewrite \
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
  && (apt-get install --assume-yes --no-install-recommends --quiet ssmtp || echo "Unable to install ssmtp") \
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
COPY ./files/etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ./files/etc/cron.d/glpi /etc/cron.d/glpi
COPY ./files/etc/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf
COPY ./files/opt/startup.sh /opt/startup.sh
COPY ./files/usr/local/etc/php/php.ini /usr/local/etc/php/php.ini

# Install GLPI crontab.
RUN crontab -u www-data /etc/cron.d/glpi

WORKDIR /var/www/glpi

# Make startup script executable and executes it as default command.
RUN chmod u+x /opt/startup.sh
CMD /opt/startup.sh

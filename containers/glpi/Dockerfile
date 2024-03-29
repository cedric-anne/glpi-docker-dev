ARG BASE_IMAGE=php:apache-bullseye


#####
# Fetch composer latest build
#####
FROM composer:latest AS composer

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
  && PHP_MAJOR_VERSION="$(echo $PHP_VERSION | cut -d '.' -f 1)" \
  && PHP_MINOR_VERSION="$(echo $PHP_VERSION | cut -d '.' -f 2)" \
  \
  # Install bz2 extension (for marketplace).
  && apt-get install --assume-yes --no-install-recommends --quiet libbz2-dev \
  && docker-php-ext-install bz2 \
  \
  # Install exif extension.
  && docker-php-ext-install exif \
  \
  # Install gd PHP extension.
  # GD extension configuration parameters changed on PHP 7.4
  # see https://www.php.net/manual/en/image.installation.php#image.installation
  && apt-get install --assume-yes --no-install-recommends --quiet libfreetype6-dev libjpeg-dev libpng-dev \
  && if [ $PHP_MAJOR_VERSION -le "5" ] || ([ $PHP_MAJOR_VERSION -eq "7" ] && [ $PHP_MINOR_VERSION -le "3" ]); then \
    # For PHP <= 5.x or PHP 7 <= 7.3, use old parameters
    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  ; else \
    docker-php-ext-configure gd --with-freetype --with-jpeg \
  ; fi \
  && docker-php-ext-install gd \
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
  # Install memcached PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libmemcached-dev \
  && pecl install memcached \
  && docker-php-ext-enable memcached \
  \
  # Install mysqli PHP extension.
  && docker-php-ext-install mysqli \
  \
  # Install opcache PHP extension.
  && docker-php-ext-install opcache \
  \
  # Install pcntl PHP extension (required for composer-require-checker).
  && docker-php-ext-install pcntl \
  \
  # Install redis PHP extension.
  && pecl install redis \
  && docker-php-ext-enable redis \
  \
  # Install soap PHP extension (required for some plugins).
  && apt-get install --assume-yes --no-install-recommends --quiet libxml2-dev \
  && docker-php-ext-install soap \
  \
  # Install XMLRPC PHP extension.
  && apt-get install --assume-yes --no-install-recommends --quiet libxml2-dev \
  && if [ $PHP_MAJOR_VERSION -lt "8" ]; then \
    # For PHP < 8.x, install bundled extension
    docker-php-ext-install xmlrpc \
  ; else \
    # For PHP 8+, install from Github (extension should be available on PECL but is not)
    mkdir -p /tmp/xmlrpc \
    && curl -LsfS https://github.com/php/pecl-networking-xmlrpc/archive/0f782ffe52cebd0a65356427b7ab72d48b72d20c/xmlrpc-0f782ff.tar.gz | tar xvz -C "/tmp/xmlrpc" --strip 1 \
    && docker-php-ext-configure /tmp/xmlrpc --with-xmlrpc \
    && docker-php-ext-install /tmp/xmlrpc \
    && rm -rf /tmp/xmlrpc \
  ; fi \
  \
  # Install zip PHP extension (for marketplace).
  # Zip extension configuration parameters changed on PHP 7.3
  # see https://www.php.net/manual/en/zip.installation.php#zip.installation
  && apt-get install --assume-yes --no-install-recommends --quiet libzip-dev zip \
  && if [ $PHP_MAJOR_VERSION -le "5" ] || ([ $PHP_MAJOR_VERSION -eq "7" ] && [ $PHP_MINOR_VERSION -le "2" ]); then \
    # For PHP <= 5.x or PHP 7 <= 7.2, configure command must be done
    docker-php-ext-configure zip --with-libzip \
  ; fi \
  && docker-php-ext-install zip \
  \
  # Enable apache mods.
  && a2enmod rewrite \
  \
  # Install cron service.
  && apt-get install --assume-yes --no-install-recommends --quiet cron \
  \
  # Install nodejs and npm.
  && apt-get install --assume-yes --no-install-recommends --quiet gnupg \
  && mkdir -p /etc/apt/keyrings \
  && curl --fail --silent --show-error --location https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor --output /etc/apt/keyrings/nodesource.gpg \
  && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
  && apt-get update \
  && apt-get install --assume-yes --no-install-recommends --quiet nodejs \
  \
  # Install git and zip used by composer when fetching dependencies.
  && apt-get install --assume-yes --no-install-recommends --quiet git unzip \
  \
  # Install gettext used for translation files.
  && apt-get install --assume-yes --no-install-recommends --quiet gettext \
  \
  # Install dependencies for plugin release tool
  && apt-get install --assume-yes --no-install-recommends --quiet gpg python3 python3-git python3-gitdb python3-github python3-lxml python3-termcolor \
  && ln -s /usr/bin/python3 /usr/bin/python \
  \
  # Install transifex client
  && (cd /usr/local/bin/ && curl --silent --location https://raw.githubusercontent.com/transifex/cli/master/install.sh | bash) \
  \
  # Install nano, as one may use it to update system files.
  && apt-get install --assume-yes --no-install-recommends --quiet nano \
  \
  # Clean sources list
  && rm -rf /var/lib/apt/lists/*

# Copy composer binary
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Copy files to container.
COPY ./files/etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ./files/etc/cron.d/glpi /etc/cron.d/glpi
COPY ./files/etc/php/conf.d/glpi.ini $PHP_INI_DIR/conf.d/glpi.ini
COPY ./files/opt/startup.sh /opt/startup.sh

# Install GLPI crontab.
RUN crontab -u www-data /etc/cron.d/glpi

# Define GLPI environment variables
ENV \
  GLPI_ENVIRONMENT_TYPE=development

WORKDIR /var/www/glpi

# Make startup script executable and executes it as default command.
RUN chmod u+x /opt/startup.sh
CMD /opt/startup.sh

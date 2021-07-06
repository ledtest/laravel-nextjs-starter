FROM ubuntu:20.04

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin vim\
    && mkdir -p ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C300EE8C \
    && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php8.0-cli php8.0-dev \
       php8.0-pgsql php8.0-sqlite3 php8.0-gd \
       php8.0-curl php8.0-memcached \
       php8.0-imap php8.0-mysql php8.0-mbstring \
       php8.0-xml php8.0-zip php8.0-bcmath php8.0-soap \
       php8.0-intl php8.0-readline \
       php8.0-msgpack php8.0-igbinary php8.0-ldap \
       php8.0-redis \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get install -y nodejs \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn \
    && apt-get -y autoremove \
    && apt-get clean \
    && apt-get install -y zsh \
    && apt-get install -y wget \
    && apt-get install -y php-xdebug \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true

RUN apt-get update \
    && apt-get install -y python3-pip \
    && pip3 --version \
    && python3 -m pip install cx_Oracle --upgrade --user

ARG INSTALL_OCI8=true
ARG ORACLE_INSTANT_CLIENT_MIRROR=https://github.com/diogomascarenha/oracle-instantclient/raw/master/

ENV LD_LIBRARY_PATH="/opt/oracle/instantclient_12_1"
ENV OCI_HOME="/opt/oracle/instantclient_12_1"
ENV OCI_LIB_DIR="/opt/oracle/instantclient_12_1"
ENV OCI_INCLUDE_DIR="/opt/oracle/instantclient_12_1/sdk/include"
ENV OCI_VERSION=12

RUN if [ ${INSTALL_OCI8} = true ]; then \
    # Install wget
    apt-get install --no-install-recommends -yqq wget \
    # Install Oracle Instantclient
    && mkdir /opt/oracle \
        && cd /opt/oracle \
        && wget ${ORACLE_INSTANT_CLIENT_MIRROR}instantclient-basic-linux.x64-12.1.0.2.0.zip \
        && wget ${ORACLE_INSTANT_CLIENT_MIRROR}instantclient-sdk-linux.x64-12.1.0.2.0.zip \
        && unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
        && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
        && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
        && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
        && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
        && rm -rf /opt/oracle/*.zip \
        && apt-get update \
    # Install PHP extensions deps
    && apt-get install --no-install-recommends -yqq \
      libaio-dev \
      freetds-dev && \
    # Install PHP extensions
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.0.10; \
    else \
      echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8; \
    fi \
#        && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 \
#        && docker-php-ext-configure pdo_dblib --with-libdir=/lib/x86_64-linux-gnu \
#        && docker-php-ext-install \
#                pdo_oci \
#        && docker-php-ext-enable \
#                oci8 \
    && echo "extension=oci8.so" >> /etc/php/8.0/cli/php.ini \
    && php -m | grep oci8 \
  ;fi

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.0

# Xdebug settings.
COPY ./xdebug.ini /etc/php/8.0/mods-available/xdebug.ini

# Add admin user
RUN useradd -ms /bin/bash -u 1337 admin

# Make some ZSH configuration.
ENV TERM xterm
ENV ZSH_THEME agnoster

# Expose ports.
EXPOSE 8000
EXPOSE 9000
EXPOSE 3000

CMD ["php", "artisan", "serve", "--host=0.0.0.0"]

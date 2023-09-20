FROM php:8.1-fpm

ENV http_proxy "http://10.7.6.6:3128"
ENV https_proxy "http://10.7.6.6:3128"

#ARG UID
#EXPOSE $UID
RUN adduser -u 1000 --disabled-password --gecos "" appuser

#para almacenar claves privadas, para repos privados
#RUN mkdir /home/appuser/.ssh
#RUN chown -R appuser:appuser /home/appuser/

#no pregunta si queremos añadir conf para repos privados
#RUN echo "StrictHostKeyChecking no" >> /home/appuser/.ssh/config
#conf de la console
#RUN echo "export COLUMNS=300" >> /home/appuser/.bashrc

COPY ./php_config/php.ini.docker_install $PHP_INI_DIR/php.ini

RUN apt-get update\
    && apt-get install -y git acl openssl openssh-client wget zip vim librabbitmq-dev libssh-dev\
    && apt-get install -y libpng-dev zlib1g-dev libzip-dev libxml2-dev libicu-dev libxslt1-dev
RUN docker-php-ext-install intl pdo pdo_mysql zip gd soap bcmath sockets opcache xsl

RUN curl --insecure https://getcomposer.org/composer.phar -o /usr/bin/composer && chmod +x /usr/bin/composer
RUN composer self-update

#linters
RUN wget https://cs.symfony.com/download/php-cs-fixer-v2.phar -O php-cs-fixer
RUN chmod a+x php-cs-fixer
RUN mv php-cs-fixer /usr/local/bin/php-cs-fixer

RUN apt-get update && apt-get install -y gnupg2
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y yarn

# Nodejs update
#RUN curl -fsSL https://deb.nodesource.com/setup_current.x | bash -
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash -
RUN apt-get install -y nodejs

COPY ./xdebug.ini $PHP_INI_DIR/conf.d/
RUN if [ "${http_proxy}" != "" ]; then \
    # Needed for pecl to succeed
    pear config-set http_proxy ${http_proxy} \
;fi
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
## Prevent error in nginx error.log
RUN touch /var/log/xdebug_remote.log
RUN chmod 777 /var/log/xdebug_remote.log

#sf ejecuta bin/console.
RUN echo "alias sf=/appdata/www/bin/console" >> /home/appuser/.bashrc

#Instalacion de Symfony/CLI
RUN wget https://get.symfony.com/cli/installer -O - | bash && export PATH="$HOME/.symfony/bin:$PATH"
RUN mv /root/.symfony5/bin/symfony /usr/local/bin/symfony

RUN mkdir -p /appdata/www

#optmiza dependecias de composer
USER appuser
#COPY ./php_config/php.ini.stg_testing $PHP_INI_DIR/php.ini
#RUN composer global require "hirak/prestissimo:^0.3" --prefer-dist --no-progress --no-suggest --optimize-autoloader --classmap-authoritative \
	#&& composer clear-cache

WORKDIR /appdata/www

FROM claritecs/bunker-maestro:latest

RUN apt-get install -y zip git
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && php composer-setup.php && php -r "unlink('composer-setup.php');" && mv composer.phar /usr/bin/composer
RUN composer install
CMD bash -c "composer dumpautoload && composer pipeline:checks"

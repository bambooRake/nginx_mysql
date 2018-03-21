FROM centos:centos6

RUN yum -y install https://dl.iuscommunity.org/pub/ius/archive/CentOS/6/x86_64/ius-release-1.0-14.ius.centos6.noarch.rpm \
    && yum -y update \
    && yum -y install php71u-cli \
    && yum -y install php71u-pecl-xdebug \
    && yum -y install php71u-json \
    && yum -y install php71u-xml \
    && yum -y install php71u-mbstring \
    && yum -y install php71u-pdo \
    && yum -y install php71u-fpm-nginx \
    && yum -y install php71u-mysqlnd \
    && yum -y install mysql56u
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && echo 'xdebug.remote_port=9004' >> /etc/php.ini \
    && echo 'xdebug.remote_enable=1' >> /etc/php.ini \
    && echo 'xdebug.remote_autostart=1' >> /etc/php.ini \
    && echo 'xdebug.remote_log=/tmp/xdebug.log' >> /etc/php.ini \
    && echo 'xdebug.remote_host=docker.for.win.localhost' >> /etc/php.ini 
COPY ./autorun.sh /usr/etc
ENTRYPOINT /usr/etc/autorun.sh
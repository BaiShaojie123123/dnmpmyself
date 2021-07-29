ARG PHP_VERSION
FROM ${PHP_VERSION}

ARG TZ
ARG PHP_EXTENSIONS
ARG CONTAINER_PACKAGE_URL


RUN sed -i "s/dl-cdn.alpinelinux.org/${CONTAINER_PACKAGE_URL}/g" /etc/apk/repositories


COPY ./extensions /tmp/extensions
#将.ssh目录复制到临时目录为了后面隧道连接使用
COPY ./.ssh /tmp/.ssh
WORKDIR /tmp/extensions
RUN chmod +x install.sh \
    && sh install.sh \
#    && mv /tmp/.ssh/init.sh /tmp/init.sh \
    && ls -la /tmp \
    ##将隧道连接等配置文件放到可执行文件目录下
    && mv /tmp/.ssh /root \
    && apk  add openssh-client \
    #mysql redis 隧道连接服务安装
    && apk  add autossh \
    && ls -la /root/.ssh \
    ##修改 阿里云服务器隧道连接所需的服务器连接私钥 !! 这个文件和config不可同时上传到公开地方否则任何人都能登陆
    && chmod 600 /root/.ssh/ali.pem \
#    && chmod +x /tmp/init.sh \
    && rm -rf /tmp/extensions

ADD ./extensions/install-php-extensions  /usr/local/bin/

RUN chmod uga+x /usr/local/bin/install-php-extensions

RUN apk --no-cache add tzdata \
    && cp "/usr/share/zoneinfo/$TZ" /etc/localtime \
    && echo "$TZ" > /etc/timezone


# Fix: https://github.com/docker-library/php/issues/240
RUN apk add gnu-libiconv libstdc++ --no-cache --repository http://${CONTAINER_PACKAGE_URL}/alpine/edge/community/ --allow-untrusted
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php


# Install composer and change it's cache home
RUN curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer
ENV COMPOSER_HOME=/tmp/composer

# php image's www-data user uid & gid are 82, change them to 1000 (primary user)
RUN apk --no-cache add shadow && usermod -u 1000 www-data && groupmod -g 1000 www-data


WORKDIR /www

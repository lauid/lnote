---
title: docker.php7.redis
date: 2018-06-06 15:26:36
tags: docker
categories: 运维
---

docker build经常会报找不到redis包的问题
No releases available for package "pecl.php.net/redis"

```yaml
ENV PHPREDIS_VERSION 3.0.0

RUN mkdir -p /usr/src/php/ext/redis \
		&& curl -L https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 \
		&& echo 'redis' >> /usr/src/php-available-exts \
		&& docker-php-ext-install redis
```

- [ How to install php-redis extension using the official PHP Docker image approach? ]( https://stackoverflow.com/questions/31369867/how-to-install-php-redis-extension-using-the-official-php-docker-image-approach/38567344#38567344 )

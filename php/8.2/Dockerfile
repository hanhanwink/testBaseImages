# syntax=docker/dockerfile:1
ARG PHP_VERSION=8.2
FROM php:${PHP_VERSION}-fpm-bookworm

ENV COMPOSER_ALLOW_SUPERUSER=1
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /app

# ------------------------------------------------------------------
# 1. 系统依赖 + Node.js 20 LTS
# ------------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
        $PHPIZE_DEPS \
        apt-utils \
        ca-certificates \
        iputils-ping \
        net-tools \
        telnet \
        curl \
        gnupg \
        git \
        unzip \
        zip \
        less \
        wget \
        vim \
        cmake \
        nginx \
        supervisor \
        cron \
        libzip-dev \
        libbz2-dev \
        zlib1g-dev \
        libicu-dev \
        g++ \
        libpq-dev \
        libjpeg-dev \
        libpng-dev \
        libfreetype6-dev \
        libwebp-dev \
        libssl-dev \
        libonig-dev \
        libxml2-dev \
        libsodium-dev \
        libargon2-dev \
        libmagickwand-dev \
        imagemagick \
        mariadb-client \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


 


# ------------------------------------------------------------------
# 2. PHP 扩展：pdo_mysql / pdo_pgsql / mysqli / gd(WebP) / intl /
#    opcache / pcntl / sockets / exif / zip / bz2 / bcmath
# ------------------------------------------------------------------
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
    && docker-php-ext-configure intl \
    && docker-php-ext-install -j"$(nproc)" \
        pdo_mysql \
        pdo_pgsql \
        mysqli \
        bcmath \
        bz2 \
        intl \
        opcache \
        pcntl \
        sockets \
        exif \
        gd \
        zip

# ------------------------------------------------------------------
# 3. PECL 扩展：imagick + redis（PHP 8.2 不要锁旧版本号）
# ------------------------------------------------------------------
RUN pecl channel-update pecl.php.net \
    && pecl install imagick-3.7.0 \
    && pecl install redis \
    && docker-php-ext-enable imagick redis \
    && rm -rf /tmp/pear ~/.pearrc

# ------------------------------------------------------------------
# 4. Composer
# ------------------------------------------------------------------ 
COPY --from=composer:2.7 /usr/bin/composer /usr/local/bin/composer


# ------------------------------------------------------------------
# 9. 可选：AWS CLI v2（你历史镜像里带过 aws-cli，需要时取消注释）
# ------------------------------------------------------------------
# RUN curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip \
#     && unzip /tmp/awscliv2.zip -d /tmp \
#     && /tmp/aws/install \
#     && rm -rf /tmp/aws /tmp/awscliv2.zip

COPY --from=amazon/aws-cli:2.37.9 /usr/local/aws-cli /usr/local/aws-cli

RUN ln -s /usr/local/aws-cli/v2/current/bin/aws /usr/local/bin/aws \
    && ln -s /usr/local/aws-cli/v2/current/bin/aws_completer /usr/local/bin/aws_completer \
    && aws --version


CMD ["tail", "-f", "/dev/stdout"]
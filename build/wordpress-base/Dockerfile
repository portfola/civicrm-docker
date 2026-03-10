# Published image: civicrm/wordpress-base
#
# This is a base image that serves as a foundation for the CiviCRM on WordPress
# image. It is based on the civicrm-base and includes all dependencies required
# to run CiviCRM on WordPress but does not include CiviCRM or WordPress codebases.
# 
# You may find this image useful as a base image for custom CiviCRM on WordPress
# build processes.

ARG PHP_VERSION

ARG IMAGE_PREFIX=civicrm

FROM ${IMAGE_PREFIX}/common-base:php${PHP_VERSION}

# Install required packages (including those required for PHP extensions).
# 
# Please document the reason for each installed package below.
#
# less is required for WP-CLI
# sudo is convenient for running WP-CLI as the www-data user
# mariadb-client is required for WP-CLI (already present in common-base)
# unzip is required for uncompressing zip archives

RUN set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    less \
    sudo \
    unzip \
    && \
    rm -rf /var/lib/apt/lists/*

# Configure and install php extensions recommended by WordPress (in addition to those installed for CiviCRM).
#
# Based on https://make.wordpress.org/hosting/handbook/server-environment/#php-extensions

RUN set -eux && \
    docker-php-ext-install exif && \
    pecl install igbinary && \
    docker-php-ext-enable igbinary

# Install WP-CLI
#
# Based on https://make.wordpress.org/cli/handbook/guides/installing/
#
# The real binary is installed as wp-cli.phar. A wrapper script at /usr/local/bin/wp
# runs it as www-data with the correct WORDPRESS_CONFIG_FILE, ensuring it works in
# interactive shells, non-interactive exec, and scripts.

RUN curl https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -o /usr/local/bin/wp-cli.phar \
    && chmod +x /usr/local/bin/wp-cli.phar \
    && printf '#!/bin/bash\nexec sudo -u www-data WORDPRESS_CONFIG_FILE="${WORDPRESS_CONFIG_FILE:-/var/www/private/wp-config.php}" /usr/local/bin/wp-cli.phar "$@"\n' > /usr/local/bin/wp \
    && chmod +x /usr/local/bin/wp

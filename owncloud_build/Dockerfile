FROM ubuntu:22.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Set environment variables
ENV OWNCLOUD_DIR=/var/www/owncloud

# Update system and install prerequisites
RUN apt update && \
    apt upgrade -y && \
    apt install -y software-properties-common wget curl ca-certificates

# Add ondrej/php PPA for PHP 7.4
RUN add-apt-repository ppa:ondrej/php -y && \
    apt update && apt upgrade -y

# Install PHP 8.1 for build owncloud only
RUN apt install -y \
    composer \
    php8.1 \
    php8.1-cli \
    php8.1-common \
    php8.1-xml \
    php8.1-mbstring \
    php8.1-zip \
    php8.1-curl \
    php8.1-gd \
    php8.1-apcu \
    php8.1-imagick \
    php8.1-intl \
    php8.1-memcached

# Install nodejs for build owncloud only
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash - && \
    apt install -y nodejs build-essential

# Install Yarn for build owncloud only
RUN curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | tee /etc/apt/trusted.gpg.d/yarn.asc && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt update && \
    apt install -y yarn

# Install additional useful packages
RUN apt install -y \
    unzip \
    bzip2 \
    rsync \
    curl \
    jq

# Create entrypoint script
RUN echo '#!/bin/bash\n\
set -e\n\
\n\
# Prepare for installation\n\
update-alternatives --set php /usr/bin/php8.1\n\
cd /var/www/owncloud\n\
make' > /usr/local/bin/docker-entrypoint.sh && \
    chmod +x /usr/local/bin/docker-entrypoint.sh

# Set working directory
WORKDIR ${OWNCLOUD_DIR}

# Set entrypoint
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
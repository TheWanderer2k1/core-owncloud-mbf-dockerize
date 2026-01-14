FROM ubuntu:22.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Set args
ARG OWNCLOUD_DOMAIN
ARG DB_TYPE
ARG DB_NAME
ARG DB_USER
ARG DB_PASS
ARG DB_HOST
ARG DB_PORT
ARG ADMIN_USER
ARG ADMIN_PASS
ARG OWNCLOUD_DIR=/var/www/owncloud
ARG OWNCLOUD_IP

# Set env
ENV OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
ENV DB_TYPE=${DB_TYPE}
ENV DB_NAME=${DB_NAME}
ENV DB_USER=${DB_USER}
ENV DB_PASS=${DB_PASS}
ENV DB_HOST=${DB_HOST}
ENV DB_PORT=${DB_PORT}
ENV ADMIN_USER=${ADMIN_USER}
ENV ADMIN_PASS=${ADMIN_PASS}
ENV OWNCLOUD_DIR=/var/www/owncloud
ENV OWNCLOUD_IP=${OWNCLOUD_IP}

# Update system and install prerequisites
RUN apt update && \
    apt upgrade -y && \
    apt install -y software-properties-common wget curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Add ondrej/php PPA for PHP 7.4
RUN add-apt-repository ppa:ondrej/php -y && \
    apt update && apt upgrade -y

# Install Apache and PHP 7.4 with required extensions
RUN apt install -y \
    apache2 \
    libapache2-mod-php7.4 \
    openssl \
    wget \
    redis-tools \
    php7.4 \
    php7.4-imagick \
    php7.4-common \
    php7.4-curl \
    php7.4-gd \
    php7.4-imap \
    php7.4-intl \
    php7.4-json \
    php7.4-mbstring \
    php7.4-gmp \
    php7.4-bcmath \
    php7.4-pgsql \
    php7.4-ssh2 \
    php7.4-xml \
    php7.4-zip \
    php7.4-apcu \
    php7.4-redis \
    php7.4-ldap \
    php-phpseclib

# Install smbclient PHP module
RUN apt-get install -y php7.4-smbclient && \
    echo "extension=smbclient.so" > /etc/php/7.4/mods-available/smbclient.ini && \
    phpenmod smbclient

# Verify smbclient
RUN php -m | grep smbclient

# Install additional useful packages
RUN apt install -y \
    unzip \
    bzip2 \
    rsync \
    curl \
    jq \
    inetutils-ping \
    ldap-utils \
    smbclient \
    cron

# Enable Apache modules
RUN a2enmod rewrite headers env dir mime setenvif ssl

# Configure Apache Virtual Host
RUN echo "<VirtualHost *:80>\n\
    ServerName ${OWNCLOUD_DOMAIN}\n\
    DirectoryIndex index.php index.html\n\
    DocumentRoot /var/www/owncloud\n\
    <Directory /var/www/owncloud>\n\
        Options +FollowSymlinks -Indexes\n\
        AllowOverride All\n\
        Require all granted\n\
        <IfModule mod_dav.c>\n\
            Dav off\n\
        </IfModule>\n\
        SetEnv HOME /var/www/owncloud\n\
        SetEnv HTTP_HOME /var/www/owncloud\n\
    </Directory>\n\
</VirtualHost>" > /etc/apache2/sites-available/owncloud.conf

RUN apachectl -t

RUN a2dissite 000-default && \
    a2ensite owncloud.conf

RUN echo "ServerName ${OWNCLOUD_DOMAIN}" >> /etc/apache2/apache2.conf

RUN apache2ctl -k restart

# Create entrypoint script
RUN echo '#!/bin/bash\n\
set -e\n\
\n\
# Install ownCloud if not already installed\n\
if [ ! -f /var/www/owncloud/config/config.php ]; then\n\
 echo "Installing ownCloud..."\n\
 /var/www/owncloud/occ maintenance:install \\\n\
        --database "$DB_TYPE" \\\n\
        --database-name "$DB_NAME" \\\n\
        --database-user "$DB_USER" \\\n\
        --database-pass "$DB_PASS" \\\n\
        --database-host "$DB_HOST:$DB_PORT" \\\n\
        --data-dir "$OWNCLOUD_DIR/data" \\\n\
        --admin-user "$ADMIN_USER" \\\n\
        --admin-pass "$ADMIN_PASS" \n\
 \n\
 echo "Configure trusted domains..."\n\
 /var/www/owncloud/occ config:system:set trusted_domains 1 --value="$OWNCLOUD_IP" \n\
 /var/www/owncloud/occ config:system:set trusted_domains 2 --value="$OWNCLOUD_DOMAIN" \n\
 /var/www/owncloud/occ config:system:set trusted_domains 3 --value="localhost" \n\
 \n\
 echo "ownCloud installation completed! Please wait for Apache to start..."\n\
else\n\
 echo "ownCloud is already installed. Please wait for Apache to start..."\n\
fi\n\
\n\
# Make sure the permissions are correct\n\
chown -R www-data:www-data /var/www/owncloud\n\
# Start apache in background\n\
echo "Starting Apache..."\n\
exec apache2ctl -D FOREGROUND' > /usr/local/bin/docker-entrypoint.sh && \
    chmod +x /usr/local/bin/docker-entrypoint.sh

# Configure logrotate
RUN echo '/var/www/owncloud/data/owncloud.log {\n\
    size 10M\n\
    rotate 12\n\
    copytruncate\n\
    missingok\n\
    compress\n\
    compresscmd /bin/gzip\n\
}' > /etc/logrotate.d/owncloud

WORKDIR ${OWNCLOUD_DIR}

# add marketplace app
RUN wget https://github.com/owncloud/market/releases/download/v0.9.0/market-0.9.0.tar.gz && \
    tar -xvf market-0.9.0.tar.gz -C apps/ && \
    rm market-0.9.0.tar.gz

# add s3 primary storage app
RUN wget https://github.com/owncloud/files_primary_s3/releases/download/v1.6.0/files_primary_s3-1.6.0.tar.gz && \
    tar -xvf files_primary_s3-1.6.0.tar.gz -C apps/ && \
    rm files_primary_s3-1.6.0.tar.gz

# Expose port
EXPOSE 80

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
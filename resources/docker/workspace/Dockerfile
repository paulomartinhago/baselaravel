#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/LaraDock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#

FROM laradock/workspace:1.3

MAINTAINER Paulo Martinhago <pmartinhago@gmail.com>

#
#--------------------------------------------------------------------------
# Mandatory Software's Installation
#--------------------------------------------------------------------------
#
# Mandatory Software's such as ("php7.0-cli", "git", "vim", ....) are
# installed on the base image 'laradock/workspace' image. If you want
# to add more Software's or remove existing one, you need to edit the
# base image (https://github.com/LaraDock/workspace).
#

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

#--------------------------------------------------------------------------
# Non-Root User:
#--------------------------------------------------------------------------

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ARG PGID=1000
RUN groupadd -g $PGID goutnix && \
    useradd -u $PUID -g goutnix -m goutnix

#--------------------------------------------------------------------------
# Set Timezone
#--------------------------------------------------------------------------

ARG TZ=America/Sao_Paulo
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#--------------------------------------------------------------------------
# Composer:
#--------------------------------------------------------------------------

# Add the composer.json
COPY ./composer.json /home/goutnix/.composer/composer.json

# Make sure that ~/.composer belongs to goutnix
RUN chown -R goutnix:goutnix /home/goutnix/.composer
USER goutnix

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}
RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
;fi

#--------------------------------------------------------------------------
# Crontab
#--------------------------------------------------------------------------
USER root

COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

#--------------------------------------------------------------------------
# User Aliases
#--------------------------------------------------------------------------

USER goutnix
COPY ./aliases.sh /home/goutnix/aliases.sh
RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source /home/goutnix/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc
    
USER root
RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source /home/goutnix/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

#--------------------------------------------------------------------------
# xDebug:
#--------------------------------------------------------------------------

ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get update && \
    apt-get install -y --force-yes php7.0-xdebug && \
    sed -i 's/^/;/g' /etc/php/7.0/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc \
;fi
# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/7.0/cli/conf.d/xdebug.ini

#--------------------------------------------------------------------------
# ssh:
#--------------------------------------------------------------------------
ARG INSTALL_WORKSPACE_SSH=false
ENV INSTALL_WORKSPACE_SSH ${INSTALL_WORKSPACE_SSH}

ADD insecure_id_rsa /tmp/id_rsa
ADD insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
    rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
        && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
        && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
        && rm -f /tmp/id_rsa* \
        && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
;fi

#--------------------------------------------------------------------------
# Node / NVM:
#--------------------------------------------------------------------------

# Check if NVM needs to be installed
ARG NODE_VERSION=stable
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NODE=false
ENV INSTALL_NODE ${INSTALL_NODE}
ENV NVM_DIR /home/goutnix/.nvm
RUN if [ ${INSTALL_NODE} = true ]; then \
    # Install nvm (A Node Version Manager)
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.6/install.sh | bash && \
        . $NVM_DIR/nvm.sh && \
        nvm install ${NODE_VERSION} && \
        nvm use ${NODE_VERSION} && \
        nvm alias ${NODE_VERSION} && \
        npm install -g gulp bower vue-cli \
;fi

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

# Add NVM binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/goutnix/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

#--------------------------------------------------------------------------
# Non-root user : PHPUnit path
#--------------------------------------------------------------------------

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER goutnix

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

#--------------------------------------------------------------------------
# Laravel Artisan Alias
#--------------------------------------------------------------------------
USER root

RUN echo "" >> ~/.bashrc && \
    echo 'alias art="php artisan"' >> ~/.bashrc

#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------

# Clean up
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set default work directory
WORKDIR /var/www
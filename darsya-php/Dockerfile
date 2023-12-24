FROM fadlee/php7-apache

# Set UID and GID
ARG USER_ID=1000
ARG GROUP_ID=1000

# Create a user and group
RUN groupadd -g $GROUP_ID darsya \
    && useradd -u $USER_ID -g $GROUP_ID -m -s /bin/bash darsya \
    && sed -i 's/www-data/darsya/g' /etc/apache2/envvars


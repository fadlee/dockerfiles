FROM bitnami/mariadb:10.5

USER root

RUN apt-get update && apt-get install -y rsync && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives

RUN sed -i "/print_welcome_page/ i [ -d /slave-data ] && rsync -aPv --delete /slave-data/ /bitnami/mariadb/data/" \
        /opt/bitnami/scripts/mariadb/entrypoint.sh

USER 1001

FROM debian:13.4-slim

LABEL maintainer="Dennis Boeckmann <dev@dbck.de>"
LABEL description="Debian based simple dbckmail container for development purposes"

ENV DEBIAN_FRONTEND="noninteractive"

RUN apt-get update \
    && apt-get -q -y --no-install-recommends install \
    curl \
    dovecot-imapd \
    nginx \
    php \
    php-fpm \
    php-mbstring \
    php-pear \
    php-sqlite3 \
    php-zip \
    postfix \
    roundcube \
    roundcube-plugins \
    roundcube-plugins-extra \
    roundcube-sqlite3 \
    rsyslog \
    sqlite3 \
    ssl-cert \
    telnet \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

ENV DBCKMAIL_USER="dbckmail"
ENV DBCKMAIL_PASSWORD="dbckmail"
ENV DBCKMAIL_MAILBOX_LIMIT="51200000"
ENV DBCKMAIL_MESSAGE_LIMIT="10240000"
ENV DBCKMAIL_MAX_RECIPIENT_LIMIT="1000"
ENV DBCKMAIL_ROUNDCUBE_CONFIG_REQUEST_PATH=""
ENV DBCKMAIL_ROUNDCUBE_NAME="DBCKMail Roundcube"

# Avoid kernel logging
RUN sed -i '/imklog/s/^/#/' /etc/rsyslog.conf

# Postfix
COPY root/etc/postfix/* /etc/postfix/

# Dovecot
COPY root/usr/local/sbin/dovecot-start-wating-for-postfix.sh /usr/local/sbin/dovecot-start-wating-for-postfix.sh
RUN chmod +x /usr/local/sbin/dovecot-start-wating-for-postfix.sh
COPY root/etc/dovecot/conf.d/* /etc/dovecot/conf.d/
RUN groupadd -g 5000 vmail && \
    useradd -g vmail -u 5000 vmail -d /var/mail/vmail -m && \
    usermod -a -G dovecot postfix

# NGINX
COPY root/etc/nginx/sites-available/roundcube /etc/nginx/sites-available/
RUN rm /etc/nginx/sites-enabled/default && \
    ln -s /etc/nginx/sites-available/roundcube /etc/nginx/sites-enabled/roundcube

# Roundcube
COPY root/etc/roundcube/config.inc.php etc/roundcube/config.inc.php
RUN mkdir -p /var/lib/roundcube/db && \
    sqlite3 -init /usr/share/roundcube/SQL/sqlite.initial.sql /var/lib/roundcube/db/sqlite.db && \
    chmod 775 -R /var/lib/roundcube/db && \
    chown -R www-data:www-data /var/lib/roundcube/db

# Supervisord
COPY root/etc/supervisor/conf.d/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

COPY root/docker-entrypoint.sh /
RUN chmod 777 /docker-entrypoint.sh

EXPOSE 25 80 465 587 143 993

ENTRYPOINT ["/docker-entrypoint.sh"]

HEALTHCHECK --interval=5s --timeout=5s CMD curl -f http://127.0.0.1 || exit 1
FROM govpf/debian:trixie

ENV APACHE_CONFDIR=/etc/apache2
ENV APACHE_ENVVARS=$APACHE_CONFDIR/envvars

ENV APACHE_HTTP_PORT=80

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends apache2; \
	rm -rf /var/lib/apt/lists/*; \
	\
	sed -ri 's/^export ([^=]+)=(.*)$/: ${\1:=\2}\nexport \1/' "$APACHE_ENVVARS"; \
	\
	. "$APACHE_ENVVARS"; \
	for dir in \
		"$APACHE_LOCK_DIR" \
		"$APACHE_RUN_DIR" \
		"$APACHE_LOG_DIR" \
	; do \
		rm -rvf "$dir"; \
		mkdir -p "$dir"; \
		chown "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$dir"; \
	done; \
	\
	rm -rvf /var/www/html/*; \
	\
	ln -sfT /dev/stderr "$APACHE_LOG_DIR/error.log"; \
	ln -sfT /dev/stdout "$APACHE_LOG_DIR/access.log"; \
	ln -sfT /dev/stdout "$APACHE_LOG_DIR/other_vhosts_access.log"; \
	chown -R --no-dereference "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$APACHE_LOG_DIR"; \
	\
	sed -i 's/^ServerTokens.*/ServerTokens Prod/' /etc/apache2/conf-available/security.conf; \
	sed -i 's/^ServerSignature.*/ServerSignature Off/' /etc/apache2/conf-available/security.conf


RUN a2dismod mpm_event \
	&& a2enmod mpm_prefork


RUN { \
		echo 'DirectoryIndex disabled'; \
		echo 'DirectoryIndex index.html'; \
		echo; \
		echo '<Directory /var/www/>'; \
		echo '\tOptions -Indexes'; \
		echo '\tAllowOverride All'; \
		echo '</Directory>'; \
	} | tee "$APACHE_CONFDIR/conf-available/docker-default.conf" \
	&& a2enconf docker-default \
	&& mkdir /docker-entrypoint-init.d


COPY 2.4/docker-entrypoint 2.4/apache2-foreground /usr/local/bin/

ENTRYPOINT ["docker-entrypoint"]

WORKDIR /var/www/html

EXPOSE 80 443

CMD ["apache2-foreground"]
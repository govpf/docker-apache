FROM govpf/debian:bullseye

ENV APACHE_CONFDIR=/etc/apache2
ENV APACHE_ENVVARS=$APACHE_CONFDIR/envvars

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends apache2; \
	apt-get remove --purge --auto-remove -y; \
	rm -rf /var/lib/apt/lists/*; \
	\
# generically convert lines like
#   export APACHE_RUN_USER=www-data
# into
#   : ${APACHE_RUN_USER:=www-data}
#   export APACHE_RUN_USER
# so that they can be overridden at runtime ("-e APACHE_RUN_USER=...")
	sed -ri 's/^export ([^=]+)=(.*)$/: ${\1:=\2}\nexport \1/' "$APACHE_ENVVARS"; \
	\
# setup directories and permissions
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
# delete the "index.html" that installing Apache drops in here
	rm -rvf /var/www/html/*; \
	mkdir -p /var/www/html; \
	chown www-data:www-data /var/www/html; \
	\
# logs should go to stdout / stderr
	ln -sfT /dev/stderr "$APACHE_LOG_DIR/error.log"; \
	ln -sfT /dev/stdout "$APACHE_LOG_DIR/access.log"; \
	ln -sfT /dev/stdout "$APACHE_LOG_DIR/other_vhosts_access.log"; \
	chown -R --no-dereference "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$APACHE_LOG_DIR"; \
# Allow to run as a non-root user
	adduser --system --no-create-home --group apache; \
	usermod -a -G www-data apache; \
	chown -R apache:apache "$APACHE_CONFDIR"; \
	chmod g+w $APACHE_RUN_DIR /var/www/html; \
	sed -i "s/<VirtualHost.*:80>/<VirtualHost *>/g" /etc/apache2/sites-available/000-default.conf; \
# Security
# https://owasp.org/www-project-secure-headers
# https://owasp.org/www-project-secure-headers/#div-bestpractices
	a2enmod headers; \
	sed -i 's/#Header set X-Content-Type-Options.*/Header set X-Content-Type-Options: "nosniff"/' /etc/apache2/conf-available/security.conf; \
	sed -i 's/#Header set X-Frame-Options.*/Header set X-Frame-Options: "deny"/' /etc/apache2/conf-available/security.conf; \
	sed -i 's/^ServerTokens.*/ServerTokens Prod/' /etc/apache2/conf-available/security.conf; \
	sed -i 's/^ServerSignature.*/ServerSignature Off/' /etc/apache2/conf-available/security.conf; \
	echo "Header set Content-Security-Policy: \"default-src 'self'; object-src 'none'; child-src 'self'; frame-ancestors 'none'; upgrade-insecure-requests; block-all-mixed-content\"\n" >> /etc/apache2/conf-available/security-extras.conf; \
	echo "Header set Cache-Control: 'no-store'" >> /etc/apache2/conf-available/security-extras.conf; \
	echo "Header set Permissions-Policy: 'accelerometer=(),autoplay=(),camera=(),display-capture=(),document-domain=(),encrypted-media=(),fullscreen=(),geolocation=(),gyroscope=(),magnetometer=(),microphone=(),midi=(),payment=(),picture-in-picture=(),publickey-credentials-get=(),screen-wake-lock=(),sync-xhr=(self),usb=(),web-share=(),xr-spatial-tracking=()'" >> /etc/apache2/conf-available/security-extras.conf; \
	echo "Header set Strict-Transport-Security: 'max-age=31536000 ; includeSubDomains; preload'" >> /etc/apache2/conf-available/security-extras.conf; \
	echo "Header set Referrer-Policy: 'no-referrer'" >> /etc/apache2/conf-available/security-extras.conf; \
	a2enconf security-extras;

RUN a2dismod mpm_event && a2enmod mpm_prefork

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
STOPSIGNAL SIGWINCH

USER apache
CMD ["apache2-foreground"]

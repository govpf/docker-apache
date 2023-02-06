FROM govpf/alpine:3
RUN set -eux; \
	apk update; \
	apk upgrade; \
	apk add apache2 apache2-ctl apache2-proxy apache2-ssl; \
    apk del --purge;

RUN rm -rvf /var/www/localhost; \
        rm -rvf /var/www/logs; \
        rm -rvf /var/www/run; \
        mkdir -p /var/www/html; 

RUN touch /var/log/apache2/access.log; \
	touch /var/log/apache2/access.log; \
	ln -sfT /dev/stderr "/var/log/apache2/error.log"; \
        ln -sfT /dev/stdout "/var/log/apache2/access.log"; 
        
RUN sed -i '\%ScriptAlias% s/^/#/' /etc/apache2/httpd.conf ;\
	sed -i 's#/var/www/localhost/htdocs#/var/www#' /etc/apache2/httpd.conf; \
	sed -i 's#ErrorLog logs/error.log#ErrorLog /var/log/apache2/error.log#' /etc/apache2/httpd.conf; \
	sed -i 's#CustomLog logs/access.log combined#CustomLog /var/log/apache2/access.log combined#' /etc/apache2/httpd.conf; \
        sed -i 's/#LoadModule rewrite_module modules\/mod_rewrite.so/LoadModule rewrite_module modules\/mod_rewrite.so/' /etc/apache2/httpd.conf; \
	sed -i '/<IfModule headers_module>/a  \ \ \ \ Header set X-Content-Type-Options: "nosniff"' /etc/apache2/httpd.conf; \
	sed -i 's/ServerTokens OS/ServerTokens Prod/' /etc/apache2/httpd.conf; \
	sed -i 's/ServerSignature On/ServerSignature Off/' /etc/apache2/httpd.conf; \
	sed -i '/<IfModule headers_module>/a  \ \ \ \ Header set Content-Security-Policy: "default-src '\''self'\''; connect-src '\''self'\'' *.gov.pf *.gouv.pf ; script-src '\''self'\'' '\''unsafe-inline'\'' ; object-src '\''none'\''; child-src '\''self'\''; style-src '\''self'\'' '\''unsafe-inline'\''; frame-ancestors '\''none'\''; upgrade-insecure-requests; block-all-mixed-content"' /etc/apache2/httpd.conf; \
	sed -i '/<IfModule headers_module>/a  \ \ \ \ Header set Cache-Control: '\''no-store'\''' /etc/apache2/httpd.conf; \
	sed -i '/<IfModule headers_module>/a  \ \ \ \ Header set Permissions-Policy: '\''accelerometer=(),autoplay=(),camera=(),display-capture=(),document-domain=(),encrypted-media=(),fullscreen=(),geolocation=(),gyroscope=(),magnetometer=(),microphone=(),midi=(),payment=(),picture-in-picture=(),publickey-credentials-get=(),screen-wake-lock=(),sync-xhr=(self),usb=(),web-share=(),xr-spatial-tracking=()'\''' /etc/apache2/httpd.conf; \
	sed -i '/<IfModule headers_module>/a  \ \ \ \ Header set Strict-Transport-Security: '\''max-age=31536000 ; includeSubDomains; preload'\''' /etc/apache2/httpd.conf; \
 	sed -i '/<IfModule headers_module>/a  \ \ \ \ Header set Referrer-Policy: '\''no-referrer'\''' /etc/apache2/httpd.conf; \
 	sed -i '/<IfModule dir_module>/a  \ \ \ \ DirectoryIndex disabled' /etc/apache2/httpd.conf; \
	sed -i 's#Options Indexes FollowSymLinks#Options -Indexes#' /etc/apache2/httpd.conf; \
	sed -i 's#AllowOverride None#AllowOverride All#' /etc/apache2/httpd.conf; \
	sed -i 's#Require all granted##' /etc/apache2/httpd.conf; \
	sed -i 's/logs/\/var\/log\/apache2/g' /etc/apache2/conf.d/ssl.conf; \
	sed -i 's/443/8443/g' /etc/apache2/conf.d/ssl.conf;
 	#sed -i 's/Listen 80/Listen 8080/' /etc/apache2/httpd.conf; 

RUN echo $'<Directory /var/www/html/> \n\
        Options Indexes FollowSymLinks \n\
        AllowOverride None \n\
        Require all granted \n\
</Directory>' >> /etc/apache2/httpd.conf

RUN echo $'<VirtualHost *> \n\
     DocumentRoot /var/www/html \n\
     AddDefaultCharset utf-8 \n\
     ProxyRequests Off \n\
     ProxyPreserveHost On \n\
     <Directory /var/www/html/> \n\
        RewriteEngine On \n\
        RewriteRule ^index\.html$ - [L] \n\
     </Directory> \n\
</VirtualHost>' > /etc/apache2/conf.d/000-default.conf

COPY 2.4/alpine/docker-entrypoint 2.4/alpine/apache2-foreground /usr/local/bin/

RUN curl https://bin.gov.pf/artifactory/ca/casit-prod2.crt > /usr/local/share/ca-certificates/casit-prod2.crt; \
    update-ca-certificates

RUN chmod 755 /usr/local/bin/docker-entrypoint; \
    chmod 755 /usr/local/bin/apache2-foreground;\
    chown -R apache:apache /etc/apache2; \ 
    chown -R apache:apache /var/log/apache2; \
    chown -R apache:apache /var/run/apache2; \
    chown -R apache:apache /run/apache2;  \
    chown -R apache:apache /var/www; \
    chown -R apache:apache /usr/lib/apache2; \
    chown -R apache:apache /etc/ssl/apache2

ENTRYPOINT ["docker-entrypoint"]

WORKDIR /var/www/html
STOPSIGNAL SIGWINCH

USER apache
CMD ["apache2-foreground"]





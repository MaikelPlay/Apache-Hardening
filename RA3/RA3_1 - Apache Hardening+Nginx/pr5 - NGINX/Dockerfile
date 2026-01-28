FROM debian:bookworm

# Evitar preguntas interactivas
ENV DEBIAN_FRONTEND=noninteractive

# 1. INSTALACIÓN (Añadimos git como pide el manual)
RUN apt-get update && apt-get install -y \
    nginx \
    libnginx-mod-http-modsecurity \
    php-fpm \
    openssl \
    apache2-utils \
    git \
    && apt-get clean

# 2. INSTALAR REGLAS OWASP (Pasos exactos del manual)
# Clonamos el repositorio
RUN git clone https://github.com/coreruleset/coreruleset.git /etc/nginx/modsec/owasp-crs

# Movemos el archivo de configuración de ejemplo
RUN cp /etc/nginx/modsec/owasp-crs/crs-setup.conf.example /etc/nginx/modsec/owasp-crs/crs-setup.conf

# 3. CREAR ARCHIVO MODSECURITY.CONF (Incluyendo las reglas)
# Usamos una configuración base y añadimos los Include de las reglas OWASP
RUN echo "SecRuleEngine On\n\
SecRequestBodyAccess On\n\
SecResponseBodyAccess On\n\
SecResponseBodyMimeType text/plain text/html text/xml\n\
SecDataDir /tmp\n\
# Cargamos la configuración de OWASP y sus reglas\n\
Include /etc/nginx/modsec/owasp-crs/crs-setup.conf\n\
Include /etc/nginx/modsec/owasp-crs/rules/*.conf" > /etc/nginx/modsec/modsecurity.conf

# 4. CERTIFICADOS SSL (RA3.2)
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/nginx.key \
    -out /etc/ssl/certs/nginx.crt \
    -subj "/C=ES/ST=Valencia/L=Valencia/O=Ciberseguridad/CN=localhost"

# 5. AUTH BÁSICA Y ARCHIVOS (Práctica 5)
RUN htpasswd -bc /etc/nginx/.htpasswd maikel maikel123 && \
    mkdir -p /var/www/html/privado && \
    echo "<?php phpinfo(); ?>" > /var/www/html/index.php && \
    echo "<h1>Zona Privada - WAF OWASP Activo</h1>" > /var/www/html/privado/index.html

COPY default /etc/nginx/sites-available/default

EXPOSE 80 443

CMD ["/bin/sh", "-c", "service php8.2-fpm start && nginx -g 'daemon off;'"]
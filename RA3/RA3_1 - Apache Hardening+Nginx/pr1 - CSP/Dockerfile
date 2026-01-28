# 1. HERENCIA: Usamos la imagen oficial actualizada
FROM php:8.2-apache

# 2. LIMPIEZA: Desactivamos el listado de directorios (Autoindex)
# Esto evita que un atacante vea la estructura de carpetas si falta el index.php
RUN a2dismod autoindex -f

# 3. MÓDULOS: Habilitamos el módulo de cabeceras necesario para CSP
RUN a2enmod headers

# 4. OCULTACIÓN: Hardening de información del servidor
# Modificamos la configuración global para no revelar que es Apache 2.4.x
RUN echo "ServerTokens Prod" >> /etc/apache2/apache2.conf && \
    echo "ServerSignature Off" >> /etc/apache2/apache2.conf

# 5. POLÍTICA DE SEGURIDAD (CSP): Previene inyecciones de código (XSS)
# La aplicamos en un archivo de configuración limpio
RUN echo 'Header set Content-Security-Policy "default-src self;"' > /etc/apache2/conf-enabled/security.conf

# 6. CONTENIDO: Copiamos tu web
COPY index.html /var/www/html/

EXPOSE 80
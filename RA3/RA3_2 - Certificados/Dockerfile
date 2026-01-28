# 1. HERENCIA ACUMULATIVA (Base PR4)
FROM maikelplay/pps:pr4

# 2. INSTALACIÓN OPENSSL
RUN apt-get update && apt-get install -y openssl && \
    a2enmod ssl && \
    rm -rf /var/lib/apt/lists/*

# 3. GENERAR CERTIFICADOS
RUN mkdir -p /etc/apache2/ssl && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/apache2/ssl/apache.key \
    -out /etc/apache2/ssl/apache.crt \
    -subj "/C=ES/ST=Castellon/L=Castellon/O=Caminas/OU=ServerDev/CN=www.midominioseguro.com"

# 4. CONFIGURACIÓN APACHE
COPY conf/midominio.conf /etc/apache2/sites-available/midominio.conf
RUN a2dissite 000-default.conf && a2ensite midominio.conf

# 5. CONTENIDO Y PERMISOS 
COPY content/ /var/www/html/

# Forzamos a que el usuario www-data sea el dueño de los archivos
# Esto soluciona el problema de que no se vean los estilos o imágenes
RUN chown -R www-data:www-data /var/www/html && \
    chmod -R 755 /var/www/html

EXPOSE 80 443
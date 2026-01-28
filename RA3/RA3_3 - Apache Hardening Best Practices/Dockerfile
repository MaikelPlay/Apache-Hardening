# 1. HERENCIA TOTAL (Base: PR6 ya tiene SSL y AntiDDOS)
FROM maikelplay/pps:pr6

# 2. INSTALACIÓN DE MODSECURITY (WAF)
RUN apt-get update && apt-get install -y \
    libapache2-mod-security2 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. COPIAR CONFIGURACIONES (Desde carpeta conf)
COPY conf/hardening.conf /etc/apache2/conf-available/hardening.conf
COPY conf/security-headers.conf /etc/apache2/conf-available/security-headers.conf
COPY conf/modsecurity.conf /etc/modsecurity/modsecurity.conf

# 4. ACTIVAR MÓDULOS Y CONFIGURACIONES
RUN a2enmod headers rewrite unique_id security2 && \
    a2enconf hardening security-headers

# 5. PREPARAR LOGS DE AUDITORÍA
RUN touch /var/log/apache2/modsec_audit.log && \
    chown www-data:www-data /var/log/apache2/modsec_audit.log

# 6. COPIAR WEB (Desde carpeta www)
COPY www/ /var/www/html/

# Aseguramos permisos
RUN chown -R www-data:www-data /var/www/html && \
    chmod -R 755 /var/www/html

EXPOSE 80 443
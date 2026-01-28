# 1. HERENCIA
FROM maikelplay/pps:pr3

# 2. INSTALACIÓN
RUN apt-get update && \
    apt-get install -y libapache2-mod-evasive apache2-utils && \
    rm -rf /var/lib/apt/lists/*

# 3. LOGS Y PERMISOS (CLAVE DEL ÉXITO)
# Copiado exactamente de tu compañero: root:www-data
RUN mkdir -p /var/log/mod_evasive && \
    chown -R root:www-data /var/log/mod_evasive && \
    chmod 770 /var/log/mod_evasive

# 4. CONFIGURACIÓN (evasive.conf)
# Usamos exactamente sus valores: PageCount 5, SiteCount 100
RUN echo "<IfModule mod_evasive20.c>" > /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSHashTableSize    3097" >> /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSPageCount        5" >> /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSSiteCount        100" >> /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSPageInterval     1" >> /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSSiteInterval     1" >> /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSBlockingPeriod   10" >> /etc/apache2/mods-available/evasive.conf && \
    echo "    DOSLogDir           \"/var/log/mod_evasive\"" >> /etc/apache2/mods-available/evasive.conf && \
    echo "</IfModule>" >> /etc/apache2/mods-available/evasive.conf

# 5. ACTIVACIÓN
RUN a2enmod evasive

EXPOSE 80
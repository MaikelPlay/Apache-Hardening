# 1. HERENCIA
FROM maikelplay/pps:pr2

# 2. INSTALACIÓN
RUN apt-get update && \
    apt-get install -y git && \
    rm -rf /var/lib/apt/lists/*

# 3. DESCARGA (CORREGIDO): Usamos una ruta temporal segura (/usr/src/...)
# Esto evita el conflicto de "folder already exists"
RUN git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git /usr/src/owasp-crs

# 4. MOVIMIENTO DE REGLAS
# Movemos las reglas a la carpeta de configuración de ModSecurity
RUN mkdir -p /etc/modsecurity/crs && \
    cp -r /usr/src/owasp-crs/rules /etc/modsecurity/crs/ && \
    cp /usr/src/owasp-crs/crs-setup.conf.example /etc/modsecurity/crs/crs-setup.conf

# 5. INTEGRACIÓN (Security2.conf)
# Apuntamos a las nuevas rutas que acabamos de crear
RUN echo "<IfModule security2_module>" > /etc/apache2/mods-enabled/security2.conf && \
    echo "  SecDataDir /var/cache/modsecurity" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  SecRuleEngine On" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  # 1. Config Base" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  IncludeOptional /etc/modsecurity/*.conf" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  # 2. CRS Config" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  Include /etc/modsecurity/crs/crs-setup.conf" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  # 3. CRS Reglas" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  Include /etc/modsecurity/crs/rules/*.conf" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "</IfModule>" >> /etc/apache2/mods-enabled/security2.conf

# 6. LIMPIEZA
RUN rm -rf /usr/src/owasp-crs && \
    apt-get purge -y git && \
    apt-get autoremove -y

EXPOSE 80
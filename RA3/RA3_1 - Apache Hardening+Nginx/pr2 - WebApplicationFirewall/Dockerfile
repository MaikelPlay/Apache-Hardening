# 1. HERENCIA: imagen validada de la Práctica 1
FROM maikelplay/pps:pr1

# 2. INSTALACIÓN: Instación de WAF (ModSecurity)
# Limpio caché (rm -rf) para que la imagen pese menos
RUN apt-get update && \
    apt-get install -y libapache2-mod-security2 && \
    rm -rf /var/lib/apt/lists/*

# 3. CONFIGURACIÓN: Preparo el archivo de configuración
RUN cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

# 4. HARDENING (CRÍTICO): Cambio a modo "Bloqueo"
# Si no hacemos esto, solo detecta pero NO para los ataques.
RUN sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf

# 5. ACTIVACIÓN: Hay que asegurarse de que Apache cargue el módulo
RUN a2enmod security2

EXPOSE 80
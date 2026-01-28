<div align="center">

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Apache](https://img.shields.io/badge/apache-%23D42029.svg?style=for-the-badge&logo=apache&logoColor=white)
![OWASP](https://img.shields.io/badge/OWASP-CRS-brightgreen?style=for-the-badge&logo=owasp&logoColor=white)
![WAF](https://img.shields.io/badge/WAF-ModSecurity-blue?style=for-the-badge&logo=security&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Security](https://img.shields.io/badge/Security-Hardening-green?style=for-the-badge&logo=security&logoColor=white)

</div>

<div align="center">

![Docker Pulls](https://img.shields.io/docker/pulls/maikelplay/pps?style=flat-square&logo=docker&logoColor=white&label=Docker%20Pulls&color=blue)
![Repo Size](https://img.shields.io/github/repo-size/MaikelPlay/Apache-Hardening?style=flat-square&logo=github&logoColor=white&label=Repo%20Size&color=orange)
![Last Commit](https://img.shields.io/github/last-commit/MaikelPlay/Apache-Hardening?style=flat-square&logo=git&logoColor=white&label=Last%20Update&color=success)

</div>

<div align="center">

[![](https://img.shields.io/badge/Ver_en_GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/MaikelPlay/Apache-Hardening)
[![](https://img.shields.io/badge/Descargar_de_DockerHub-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://hub.docker.com/r/maikelplay/pps)

</div>

# Práctica 3: OWASP Core Rule Set (CRS)

Esta práctica representa el nivel más alto de endurecimiento del servidor. Hemos integrado el **OWASP Core Rule Set (CRS)**, un conjunto de reglas de detección de ataques de código abierto que dota al WAF de "inteligencia" para proteger al servidor contra las vulnerabilidades más críticas (OWASP Top 10), como Inyección SQL y Java Deserialization.

---

## 📂 Estructura del directorio

A diferencia de implementaciones que requieren múltiples archivos de configuración externos, hemos logrado una solución **totalmente autocontenida** en el Dockerfile:

```text
Practica3_OWASP/
└── Dockerfile      # Descarga, configuración e integración de reglas CRS
```

##  Configuración Realizada

Para llegar al estado final del servidor, se realizaron las siguientes acciones técnicas automatizadas:

### A. Implementación del Core Rule Set (CRS)
Se automatizó la descarga desde el repositorio oficial de SpiderLabs/OWASP para garantizar firmas actualizadas:
* **Clonación**: `git clone ... /usr/src/owasp-crs`
* **Despliegue**: Migración de reglas y archivo `crs-setup.conf` al directorio `/etc/modsecurity/crs/`.

### B. Inyección Dinámica de Configuración
En lugar de copiar archivos externos, se modificó el archivo security2.conf en tiempo de construcción (RUN echo ...) para cargar las reglas en el orden correcto:
* **Configuración Base ModSecurity.**
* **Configuración CRS (Setup).**
* **Reglas CRS (*.conf)**.

### C. Optimización de la Imagen
Se implementó una política de limpieza estricta: tras instalar y configurar las reglas, se eliminan git y los archivos temporales, reduciendo drásticamente el tamaño final de la imagen.

---

##  Dockerfile

El siguiente código muestra cómo se integra OWASP CRS sobre la imagen previa (WAF activado) sin dependencias externas.

![Configuración del Dockerfile](assets/pr3-2.png)

```dockerfile
# 1. HERENCIA: Partimos de la imagen con WAF activado (Práctica 2)
FROM maikelplay/pps:pr2

# 2. INSTALACIÓN: Necesitamos git para descargar las reglas
RUN apt-get update && \
    apt-get install -y git && \
    rm -rf /var/lib/apt/lists/*

# 3. DESCARGA: Clonamos el Core Rule Set (CRS) oficial
RUN git clone [https://github.com/SpiderLabs/owasp-modsecurity-crs.git](https://github.com/SpiderLabs/owasp-modsecurity-crs.git) /usr/src/owasp-crs

# 4. MOVIMIENTO DE REGLAS: Organizo las carpetas en /etc/modsecurity
RUN mkdir -p /etc/modsecurity/crs && \
    cp -r /usr/src/owasp-crs/rules /etc/modsecurity/crs/ && \
    cp /usr/src/owasp-crs/crs-setup.conf.example /etc/modsecurity/crs/crs-setup.conf

# 5. INTEGRACIÓN: Configuro Apache dinámicamente para leer las reglas
RUN echo "<IfModule security2_module>" > /etc/apache2/mods-enabled/security2.conf && \
    echo "  SecDataDir /var/cache/modsecurity" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  SecRuleEngine On" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  # Carga de Reglas OWASP" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  Include /etc/modsecurity/crs/crs-setup.conf" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "  Include /etc/modsecurity/crs/rules/*.conf" >> /etc/apache2/mods-enabled/security2.conf && \
    echo "</IfModule>" >> /etc/apache2/mods-enabled/security2.conf

# 6. LIMPIEZA
RUN rm -rf /usr/src/owasp-crs && apt-get purge -y git && apt-get autoremove -y

EXPOSE 80
```

## Despliegue
Proceso de construcción (pr3) y arranque en el puerto 8082.

![Proceso de Build y Run](assets/pr3.png)

### Comandos de Construcción y Ejecución
```bash
# 1. Construir la imagen con reglas OWASP
docker build -t maikelplay/pps:pr3 .

# 2. Arrancar en puerto 8082
docker run -d -p 8082:80 --name owasp_test_v2 maikelplay/pps:pr3
```

##  Validación y Auditoría
Se han ejecutado pruebas de penetración reales para confirmar que el conjunto de reglas OWASP bloquea patrones de ataque complejos.

### 1. Tabla de Pruebas Realizadas

| Tipo de ataque | Payload (Carga) | Respuesta | Regla Activada (Log) |
| :--- | :--- | :--- | :--- |
| **SQL Injection (Básico)** | `?id=1' UNION SELECT 1--` | **403 Forbidden** | SQL Injection Attack Detected |
| **SQL Injection (Complejo)** | `?id=1 UNION SELECT database()` | **403 Forbidden** | OWASP_CRS/WEB_ATTACK/SQL_INJECTION |
| **Path Traversal** | `?page=../../etc/passwd` | **403 Forbidden** | Inbound Anomaly Score Exceeded |

### 2. Evidencia de Logs (Auditoría)
En la siguiente captura se observa cómo ModSecurity detecta la inyección SQL, la clasifica como CRITICAL y aplica el bloqueo.

![Logs de ModSecurity](assets/pr3-3.png)

### 3. Evidencia Visual (Navegador)
El navegador muestra el bloqueo efectivo al intentar inyectar comandos SQL en la URL.

![Bloqueo OWASP en Navegador](assets/pr3-4.png)


## 🌐 Docker Hub
Imagen disponible para su descarga

| Campo | Valor |
| :--- | :--- |
| **Repositorio** | `maikelplay/pps` |
| **Etiqueta (Tag)** | `pr3` |
| **Comando Pull** | `docker pull maikelplay/pps:pr3` |

---
<div align="center">
    <p>Desarrollado con ❤️ por <b>MaikelPlay</b></p>
    <a href="https://github.com/MaikelPlay">
        <img src="https://img.shields.io/badge/GitHub-MaikelPlay-181717?style=flat&logo=github&logoColor=white" alt="GitHub">
    </a>
    <a href="https://hub.docker.com/u/maikelplay">
        <img src="https://img.shields.io/badge/Docker-MaikelPlay-2496ED?style=flat&logo=docker&logoColor=white" alt="DockerHub">
    </a>
    <a href="https://www.linkedin.com/in/mikel-jordan-moral/">
    <img src="https://img.shields.io/badge/LinkedIn-Mikel_Jordan-0077B5?style=flat&logo=linkedin&logoColor=white" alt="LinkedIn">
</a>

<a href="https://maikelplay.github.io/portfolio-web/">
    <img src="https://img.shields.io/badge/Portfolio-Visit_Web-8A2BE2?style=flat&logo=google-chrome&logoColor=white" alt="Portfolio">
</a>
</div>

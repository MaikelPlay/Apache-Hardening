<div align="center">

# RA3_1: Server Hardening & Defense in Depth
### Portafolio de Puesta en Producción Segura

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Apache](https://img.shields.io/badge/apache-%23D42029.svg?style=for-the-badge&logo=apache&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![ModSecurity](https://img.shields.io/badge/WAF-ModSecurity-blue?style=for-the-badge&logo=security&logoColor=white)
![OWASP](https://img.shields.io/badge/OWASP-CRS-black?style=for-the-badge&logo=owasp&logoColor=white)
![SSL](https://img.shields.io/badge/SSL-OpenSSL-red?style=for-the-badge&logo=openssl&logoColor=white)

<br>

<img src="https://img.shields.io/docker/pulls/maikelplay/pps?style=flat-square&logo=docker&logoColor=white&label=Docker%20Pulls&color=blue" alt="Docker Pulls">
<img src="https://img.shields.io/github/last-commit/MaikelPlay/Apache-Hardening?style=flat-square&logo=git&logoColor=white&label=Last%20Update&color=success" alt="Last Commit">

[![](https://img.shields.io/badge/Ver_en_GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/MaikelPlay)
[![](https://img.shields.io/badge/Descargar_de_DockerHub-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://hub.docker.com/r/maikelplay/pps)

</div>

---

## Visión General del Proyecto

Este repositorio documenta la implementación de una estrategia de **Defensa en Profundidad (Defense in Depth)** para servidores web. El objetivo ha sido evolucionar desde un contenedor básico hasta una infraestructura fortificada, aplicando capas de seguridad progresivas que mitigan las vulnerabilidades del **OWASP Top 10**, ataques de fuerza bruta y denegación de servicio (DoS).

### Resultado Final: Arquitectura Ultraprotegida
El siguiente dashboard muestra el estado final del servidor (Práctica 7), donde todas las capas de defensa (Red, Transporte, Aplicación y Sistema) operan simultáneamente.

![Server Hardening Final](assets/pr7-5.png)

---

## Despliegue y Auditoría (Docker Hub)

Para facilitar la auditoría y el despliegue rápido, todas las fases del proyecto han sido empaquetadas en imágenes Docker independientes y subidas al registro público.

**Estrategia de Etiquetado:**
* **`pps`**: Repositorio principal (Puesta en Producción Segura).
* **`pr1` - `pr7`**: Versiones incrementales. Cada etiqueta hereda y mejora la seguridad de la anterior.

![Docker Hub Repository](assets/pr-final-Docker.png)

### Tabla de Versiones Disponibles

| Etiqueta (Tag) | Arquitectura | Descripción Técnica del Hardening |
| :--- | :--- | :--- |
| `pps:pr1` | Apache | Configuración base, eliminación de `autoindex` y CSP inicial. |
| `pps:pr2` | Apache | Implementación de WAF (ModSecurity) en modo detección. |
| `pps:pr3` | Apache | Integración de OWASP Core Rule Set (CRS) para bloqueo activo. |
| `pps:pr4` | Apache | Capa Anti-DDoS con `mod_evasive` y Rate Limiting. |
| `pps:pr5` | **Nginx** | Stack alternativo seguro: Nginx + PHP-FPM + WAF + Auth Básica. |
| `pps:pr6` | Apache | Infraestructura de Clave Pública (PKI), SSL/TLS y HSTS. |
| `pps:pr7` | Apache | **Hardening Final**: Ofuscación, Cabeceras estrictas y Auditoría. |

---

## Documentación Técnica de las Prácticas

A continuación se detalla la evolución técnica del proyecto, explicando las medidas de seguridad implementadas en cada fase.

### 🔹 1. CSP & Hardening Base
Fase inicial de reducción de la superficie de ataque. Se configuró Apache para minimizar la información expuesta.
* **Acciones:** Desactivación de la firma del servidor (`ServerSignature Off`), eliminación del listado de directorios (`Options -Indexes`) y configuración de una **Content Security Policy (CSP)** restrictiva para prevenir la ejecución de scripts no autorizados (mitigación XSS).

### 🔹 2. Web Application Firewall (WAF)
Introducción de seguridad en la capa de aplicación mediante **ModSecurity v3**.
* **Acciones:** Instalación del módulo `libapache2-mod-security2`. Configuración del motor de reglas (`SecRuleEngine On`) para pasar de un modo pasivo a un modo de bloqueo activo, permitiendo interceptar peticiones HTTP malformadas antes de que sean procesadas por el backend.

### 🔹 3. OWASP Core Rule Set (CRS)
Potenciación del WAF mediante inteligencia colectiva.
* **Acciones:** Implementación del conjunto de reglas **CRS de SpiderLabs**. Este set protege contra las vulnerabilidades más críticas (SQL Injection, Remote Code Execution, LFI) utilizando un sistema de puntuación de anomalías (*Anomaly Scoring*) para reducir falsos positivos.

### 🔹 4. Protección Anti-DDoS
Protección de la disponibilidad del servicio contra ataques de inundación.
* **Acciones:** Configuración de **mod_evasive**. Se establecieron umbrales de peticiones por segundo (Page Count / Site Count) y tiempos de bloqueo (*Blocking Period*). El servidor responde automáticamente con un código **403 Forbidden** a IPs que muestren comportamiento abusivo o de fuerza bruta.

### 🔹 5. Arquitectura Segura Nginx + PHP 8.4
Cambio de stack tecnológico para comparar arquitecturas de seguridad en entornos de alto rendimiento.
* **Acciones:** Construcción de una imagen desde cero basada en `Debian Bookworm`.
* **Seguridad:** Implementación de WAF en Nginx, comunicación por sockets Unix para PHP-FPM, y protección de áreas administrativas (`/privado`) mediante autenticación básica y control de acceso.

### 🔹 6. Cifrado y Certificados Digitales (PKI)
Implementación de la capa de confidencialidad e integridad de los datos.
* **Acciones:** Generación de claves RSA de 2048 bits y certificados X.509 autofirmados con OpenSSL.
* **Configuración:** Creación de VirtualHosts para redirigir forzosamente todo el tráfico HTTP (Puerto 80) a HTTPS (Puerto 443) mediante códigos de estado 301, asegurando que ninguna comunicación viaje en texto plano.

### 🔹 7. Defensa en Profundidad (Estado Final)
Consolidación de todas las capas anteriores y aplicación de "Fine-Tuning" avanzado.
* **Ofuscación:** Modificación de la identidad del servidor a *"Servidor Privado"* para dificultar el reconocimiento (*Fingerprinting*) por parte de atacantes.
* **Cabeceras de Seguridad:** Inyección estricta de `X-Frame-Options`, `X-Content-Type-Options` y `Strict-Transport-Security` (HSTS).
* **Control de Métodos:** Restricción exclusiva a verbos `GET` y `POST`, bloqueando `PUT`, `DELETE` o `TRACE`.
* **Permisos:** Endurecimiento de permisos a nivel de sistema de archivos (`chmod 750` y `chown www-data`) para prevenir escalada de privilegios.

---

## Despliegue Rápido

Para probar la versión final y auditada del servidor en tu entorno local:

```bash
# Descargar la imagen final
docker pull maikelplay/pps:pr7

# Ejecutar el contenedor (Mapeando puertos 8085->80 y 8443->443)
docker run -d -p 8085:80 -p 8443:443 --name servidor_seguro maikelplay/pps:pr7

Acceso web: https://localhost:8443
```

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

# Proyectos Bochan - Entrega Final

Bienvenidos a la documentación oficial de los proyectos. A continuación se presentan las demostraciones en vídeo del trabajo realizado.

---


## Explicación breve Boochan 1 y su proceso

### Objetivo del Proyecto
Diseñar, desplegar y securizar un servidor web y de archivos en Linux, evolucionando de una arquitectura tradicional de puertos abiertos a una infraestructura profesional "Zero Trust" de alta disponibilidad, totalmente blindada contra ataques externos.

### Resumen Ejecutivo (Para la Portada)
El proyecto Bochan 1 documenta la creación y el bastionado (hardening) de un servidor Ubuntu Server a través de cuatro ejes fundamentales:

Infraestructura Base: Despliegue inicial de un servidor web de alto rendimiento (Nginx) y un entorno enjaulado y cifrado para la transferencia de archivos (ProFTPD / SFTP).

Evolución Zero Trust: Supresión de la exposición de puertos públicos (cierre de Port Forwarding) migrando la conectividad a túneles cifrados de Cloudflare. Esto oculta la IP real del servidor e implementa validación de identidad (OTP y Browser Rendering) para la administración remota.

Criptografía y Hardening: Implementación de defensa reactiva (Fail2Ban), autenticación asimétrica sin contraseñas (Llaves RSA para SSH) y automatización de certificados SSL/TLS (Let's Encrypt) con políticas HSTS, logrando la máxima calificación de seguridad (A+) en auditorías externas.

Resiliencia (Disaster Recovery): Particionado de hardware virtual secundario e implementación de una política de copias de seguridad incrementales, automatizadas y cifradas mediante BorgBackup y Crontab.

###  Demostración Proyecto Integrador (Bochan 1)
*Proyecto realizado por Héctor, Iker y Jorge.*

https://www.youtube.com/playlist?list=PLU7Qnmepg_8_AgyWdUj0gUuw8uaapg_bu

---

## Explicacion breve Boochan 2 y su proceso

  #### Objetivo del Proyecto
Diseñar e implementar una infraestructura corporativa híbrida y segura en la nube, centralizando la gestión de identidades, el control de acceso y el almacenamiento en red para simular un entorno empresarial real.

  #### Resumen Ejecutivo (Para la Portada)
El proyecto Boochan 2 detalla el despliegue técnico de un servidor en Microsoft Azure (Ubuntu IaaS) configurado sobre cuatro pilares fundamentales:

Identidad y Dominio: Creación de un Controlador de Dominio (Samba AD DC) para la gestión centralizada de usuarios, grupos y permisos (NSS & RFC2307).

Conectividad Segura: Establecimiento de un túnel VPN cifrado (WireGuard) para conectar clientes locales de forma segura a la nube privada virtual.

Almacenamiento Avanzado: Implementación de discos virtuales con cuotas y aplicación de seguridad granular a nivel de kernel mediante listas de control de acceso (ACLs) y ABE (Access Based Enumeration), garantizando la invisibilidad de recursos no autorizados.

Monitorización: Automatización del procesamiento de logs del servidor para la extracción y registro de eventos críticos.

  ###  Demostración Boochan 2
*Proyecto realizado por Iker y Jorge.*

https://www.youtube.com/playlist?list=PLU7Qnmepg_8_94ZFcMpZDa0VXZ-DgVjHv


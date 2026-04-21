<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(255, 0, 0), rgb(0, 0, 255), transparent); margin-bottom: 20px;">

- 1. **Servidor Web Nginx:**
    
    - Instalamos Nginx como motor de alto rendimiento: `sudo apt install nginx -y`.
        
    - Verificamos que el servicio esté activo: `sudo systemctl status nginx`.
        
    - **Validación:** Al introducir `http://192.168.0.200` en el navegador del anfitrión,         <div style="float: right; margin-right: 180px; margin-left: 20px; margin-top: 30px; pointer-events: none;"> <img src="mario2.png" style="width: 50px;"> </div>visualizamos la página de bienvenida de Nginx.

- 2. **Servidor de Archivos ProFTPD con Enjaulado:** 
    
    - Instalamos el servicio: `sudo apt install proftpd -y`.
        
    - Para evitar que los usuarios escalen directorios, aplicamos el **Hardening** en `sudo nano /etc/proftpd/proftpd.conf`:
        
        - Descomentamos `DefaultRoot ~` para que cada usuario quede atrapado en su `/home`.
            
- 3. **Implementación de Cifrado SSL/TLS (FTPS):**
    
    - No permitimos conexiones inseguras. Generamos un certificado RSA de 2048 bits:
        
        - `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/proftpd.key -*out* /etc/ssl/certs/proftpd.crt`
            
    - Configuramos el módulo de seguridad en `/etc/proftpd/tls.conf` forzando el cifrado con `TLSRequired on`.

- 4. **Software para uso de FTP y comprobaciones:**
    - **Validación Final:** Descargando el programa de FileZilla Client nos conectamos desde  usando **FTP explícito sobre TLS**. El sistema nos pide aceptar nuestro certificado y el icono del candado confirma que la comunicación es privada, la configuración de FileZilla será la siguiente.<div style="margin-left: 0px; margin-bottom: 50px; pointer-events: none;"> <img src="mario3.webp" style="width: 100px; filter: drop-shadow(-5px 5px 10px rgba(255,0,0,0.3));"> </div>
      
      ![[filezilla.png]]

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(255, 0, 0), rgb(0, 0, 255), transparent); margin-bottom: 20px;">

FASE 4 Redireccion de puertos (A) →→ [[Fase 4 PORTAL Versión A (Publicación WAN y Configuración Perimetral (SR))]]
FASE 4 Cloudflare tunnel (B) →→ [[Fase 4 PORTAL Versión B (Infraestructura Cloudlare Tunnel (Zero Trust) (SR))]]
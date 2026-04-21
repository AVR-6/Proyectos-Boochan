<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(168, 0, 255), rgb(0, 166, 255), transparent); margin-bottom: 20px;">

**🎯 S5.1: Justificación del Cambio (Evolución de 4-A a 5)**

En la **Fase 4-A**, se configuró la conectividad mediante el modelo tradicional de **Port Forwarding**. Aunque funcional, este modelo expone la IP pública del servidor y depende de protocolos obsoletos como FTPS, que son complejos de gestionar tras firewalls.

**La Fase 5 representa la profesionalización de la infraestructura:**

- **Seguridad Zero Trust:** Se sustituye la apertura de puertos por un túnel saliente cifrado.
    
- **Simplificación de Protocolos:** Se descarta FTPS en favor de **SFTP (SSH File Transfer Protocol)**, centralizando la gestión de archivos y comandos en el puerto 22.
    
- **Enmascaramiento Total:** La IP real del servidor queda oculta tras la red de Cloudflare.
    



**S5.2: Implementación Profesional del Túnel**

### 1. Instalación y Autenticación

Instalamos el agente `cloudflared` y vinculamos el servidor con el dominio gestionado en Cloudflare:

 <div style="float: right; margin-right: 100px; margin-left: 520px; pointer-events: none;"> <img src="ditto2.gif" style="width: 90px;"> </div>

```
# Instalación del agente
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

# Vinculación (Login vía navegador)
cloudflared tunnel login

# Creación del túnel de identidad única
cloudflared tunnel create portal-mask
```

### 2. Estándar de Organización y Directorios

Para cumplir con el estándar **FHS (Filesystem Hierarchy Standard)** y asegurar la persistencia en los backups (Fase 7), migramos la configuración de la carpeta de root a la ruta global del sistema:



```
sudo mkdir -p /etc/cloudflared
sudo cp /root/.cloudflared/* /etc/cloudflared/
sudo chown -R iker:iker /etc/cloudflared
```

### 3. Configuración del Ingress (Mapeo de Servicios)

Editamos el archivo de configuración definiendo el dominio raíz para la web y un subdominio para la administración segura. `sudo nano /etc/cloudflared/config.yml`

YAML

```
tunnel: TU_ID_TUNNEL
credentials-file: /etc/cloudflared/TU_ID_TUNNEL.json
protocol: http2

ingress:
  - hostname: boochan.space
    service: http://localhost:80
  - hostname: sftp.boochan.space
    service: ssh://localhost:22
  - service: http_status:404
```

---

## 🔒 S5.3: El "Apagón" de Puertos y Hardening Perimetral

Una vez levantado el túnel, el servidor es accesible de forma saliente. Esto permite blindar el router y el sistema:

1. **En el Router:** Eliminar todas las reglas de _Port Forwarding_ y _DMZ_. Específicamente, cerrar los puertos **21, 22 y 80**.
    
2. **En Ubuntu (UFW):**
    

    
    ```
    # Bloquear accesos directos externos
    sudo ufw deny 21/tcp
    sudo ufw deny 80/tcp
    # Solo permitir SSH para gestión local o vía túnel
    sudo ufw allow from 192.168.1.0/24 to any port 22
    ```
    

---

## 🤖 S5.4: Configuración de Accesos y Capa Zero Trust

### 1. Registro DNS

Se vinculan los nombres de host al ID del túnel para que Cloudflare sepa dónde inyectar el tráfico:

 <div style="float: right; margin-right: 620px; pointer-events: none;"> <img src="ditto.webp" style="width: 90px;"> </div>

```
# Vincular el dominio raíz (Apex Domain)
cloudflared tunnel route dns portal-mask boochan.space

# Vincular el acceso administrativo
cloudflared tunnel route dns portal-mask sftp.boochan.space
```

### 2. Cloudflare Access (Identity Provider)

Para el subdominio `sftp.boochan.space`, se ha configurado una **Access Application** de tipo _Self-hosted_:

- **Política:** Solo se permite el acceso a correos autorizados mediante **OTP (One-Time Pin)**.
    
- **Browser Rendering:** Habilitado para SSH, permitiendo administrar el servidor mediante una consola web segura desde cualquier navegador.

La configuración se vera algo tal que así:
 <div style="pointer-events: none;"> <img src="ssh.png" style="width: 800px;"> </div>

Mientras que la policitica de entrada que añadiremos sera algo así:
 <div style="pointer-events: none;"> <img src="politica.png" style="width: 800px;"> </div>

 <div style="float: right; margin-left: 520px; pointer-events: none;"> <img src="zorua.png" style="width: 200px;"> </div>

## 🔍 S5.5: Validación y Resolución de Conflictos

Durante el despliegue se identificaron y resolvieron los siguientes puntos críticos:

- **Conflicto de SSH:** Se desactivó `ssh.socket` para permitir que `ssh.service` gestionara exclusivamente el puerto 22 sin errores de "Address already in use".
    
- **Sincronización de IDs:** Se detectaron colisiones con infraestructuras previas (otras VMs). Se solucionó mediante el forzado de rutas (`-f`) y la verificación unívoca del UUID del túnel en los nodos de Madrid (`mad01`, `mad06`).

**1. Resolución del Conflicto de SSH (Address already in use)**

El error ocurría porque en versiones modernas de Ubuntu, el sistema utiliza "sockets" para escuchar el puerto 22, lo que impedía que el servicio SSH tradicional tomara el control total.

- **Síntoma:** Al intentar reiniciar SSH, el log mostraba: `Error: Bind to port 22 failed - Address already in use`.
    
- **Solución Técnica:** Ejecutamos la desactivación del socket para liberar el puerto definitivamente:
    
    Bash
    
    ```
    # Desactivar y enmascarar el socket para que no vuelva a arrancar
    sudo systemctl stop ssh.socket
    sudo systemctl disable ssh.socket
    sudo systemctl mask ssh.socket
    
    # Reiniciar el servicio SSH estándar
    sudo systemctl restart ssh
    ```
    
- **Validación:** Al ejecutar `sudo ss -tulpn | grep :22`, ahora solo aparece el proceso `sshd`.
    

---

**2. Sincronización de IDs y Forzado de Rutas (Túnel Zero Trust)**

Al reutilizar configuraciones en diferentes VMs o tras reinstalaciones, Cloudflare puede detectar el ID del túnel como "Zombie" o bloqueado por una conexión previa en los nodos regionales de Madrid (`mad`).

- **Síntoma:** El comando `cloudflared tunnel run` fallaba indicando que el túnel ya estaba activo o que el ID no era unívoco.
    
- **Solución Técnica:** Aplicamos un forzado de la conexión y limpieza de caché de rutas:
    
    Bash
    
    ```
    # Forzar la conexión del túnel ignorando sesiones previas
    cloudflared tunnel run --force <NOMBRE_O_ID_DEL_TUNEL>
    
    # En caso de error de UUID, limpiar credenciales y re-autenticar:
    rm ~/.cloudflared/*.json
    cloudflared tunnel login
    ```
    

> **Estado Final:** Conexión establecida mediante protocolo **http2**. La web pública es accesible en `boochan.space` y la gestión administrativa está blindada tras el firewall de identidad de Cloudflare.

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(168, 0, 255), rgb(0, 166, 255), transparent); margin-bottom: 20px;">

Para ir a la siguiente Fase Click aqui →→ [[Fase 6 BATMAN (Hardening y Blindaje de Seguridad (SI))]]

<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(0, 112, 243), rgb(243, 128, 32), transparent); margin-bottom: 20px;">

- 1. **Instalación y Vinculación Inicial:**
        
    
    - Descargamos el conector oficial e instalamos el paquete: `curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb` 
      `sudo dpkg -i cloudflared.deb`
        
    - **Vínculo con Cloudflare:** Ejecutamos `cloudflared tunnel login`. Se generará un enlace que debemos abrir en el navegador para autorizar el dominio `makoko.space`. Esto descarga el certificado `cert.pem`.
        
    - **Creación del Túnel:** `cloudflared tunnel create makoko-tunnel`. Anotamos el ID generado para el fichero de configuración.
        
- 2. **Configuración de Rutas DNS:** 	<div style="float: right; margin-right: 100px; margin-left: 20px; margin-top: 30px; pointer-events: none;"> <img src="portalrickymorty.gif" style="width: 100px;"> </div>
        
    
    - Registramos los subdominios en la red de Cloudflare para que apunten al túnel:
        
        - `cloudflared tunnel route dns makoko-tunnel makoko.space`
            
        - `cloudflared tunnel route dns makoko-tunnel ssh.makoko.space`
            
        - `cloudflared tunnel route dns makoko-tunnel ftps.makoko.space`
            
- 3. **Configuración de Servicios (Ingress Rules):**
        
    
    - Editamos el fichero maestro: `sudo nano /etc/cloudflared/config.yml`.
        
    - Definimos el ID del túnel y mapeamos los servicios a los puertos locales: 
        
	- Necesitaremos cambiar el tunnel por nuestro ID tunnel y en credentials-file modificaremos tambien el ID, si no no nos funcionara nada
        
		```
		tunnel: afae8856-64f7-4fd3-803c-458c3830ec34
		credentials-file: /etc/cloudflared/afae8856-64f7-4fd3-803c-458c3830ec34.json
		ingress:
		  - hostname: makoko.space
		    service: http://localhost:80
		  - hostname: ssh.makoko.space
		    service: ssh://localhost:22
		  - hostname: ftps.makoko.space
		    service: ssh://localhost:22
		  - service: http_status:404
		``` 
    - **Punto clave: La regla 404 siempre debe ir al final para evitar errores de sintaxis.

	<div style="float: right; margin-right: 500px; margin-left: 20px; margin-top: 30px; pointer-events: none;"> <img src="portaltft.gif" style="width: 150px;"> </div>

- 4. **Gestión de Archivos Segura (SFTP) y Automatización:**
        
    
    - Para conectar clientes como **Bitvise** o **FileZilla**, creamos un "puente local" en Windows que redirige el tráfico hacia el túnel: `cloudflared access tcp --hostname ssh.makoko.space --listener 127.0.0.1:2222`
        
    - **Automatización del puente:** Para automatizar el puente y no tener que poner el comando de arriba cada vez que querramos meternos, necesitaremos crear un script como el de abajo, para que te funcione bien necesitaras modificar el usuario por tu usuario con el que funciones en tu maquina virtual y en keyphrase introduciremos la contraseña, despues de modificar todo el documento en un txt, guardaremos el archivo como .vbs (Script).
        
        
        ```
        Set WshShell = CreateObject("WScript.Shell")
        WshShell.Run "cloudflared access tcp --hostname ftps.makoko.space --listener 127.0.0.1:2222", 0, False
        WScript.Sleep 2000
        WshShell.Run """C:\Program Files (x86)\Bitvise SSH Client\BvSsh.exe"" -host=127.0.0.1 -port=2222 -user=hector -keyphrase=CONTRASEÑA ", 1, False
        ```
        
- 5. **Validación y Seguridad Zero Trust:**
        
    
    - **Validación SSH:** Mediante `ssh.makoko.space` accedemos a la terminal vía navegador con autenticación PIN por email.
        
-  5.1. **Configuración del Panel Zero Trust (Application Launcher)**

Para que el subdominio `ssh.makoko.space` sea accesible, no basta con el túnel; debemos crear una **Aplicación** en el panel de Cloudflare para gestionar quién tiene permiso de entrar.

**Pasos en el Panel de Cloudflare:**

1. **Crear la Aplicación:**
    
    - Ve a **Zero Trust -> Access -> Applications** y pulsa en **Add an application**.
        
    - Selecciona **Self-hosted**.
        
    - **Application Name:** `Acceso SSH Seguro`.
        
    - **Domain:** Pon `ssh` en el subdominio y selecciona `makoko.space`.
        
2. **Configurar la Política de Acceso (Policy):**
    
    - **Policy Name:** `Acceso por correo`.
        
    - **Action:** `Allow`.
        
    - **Assign group (Include):** Aquí es donde pones tu regla. Lo más común es poner **Emails** y añadir tu correo personal.
        
    - _Esto es lo que activa que Cloudflare te envíe el código PIN al correo cuando intentas entrar._
     <div style="pointer-events: none;"> <img src="politica.png" style="width: 800px;"> Foto de como queda la politica</p></div>
1. **Configuración de la Consola (Browser Rendering):**
    
    - En la misma configuración de la App, ve a la pestaña **Settings**.
        
    - Busca **Browser rendering** y selecciona **SSH**.
        
    - _Esto es lo que permite que en la fase de validación puedas ver la terminal directamente en el navegador de Google Chrome sin usar programas externos._

**El resultado de DNS y Aplicaciónes debería de ser similar a las siguientes fotos:**
<div style="pointer-events: none;"> <img src="dns.png" style="width: 800px;"> Foto de como queda los DNS</p>
<div style="pointer-events: none;"> <img src="aplicacion.png" style="width: 800px;"> Foto de como queda la aplicación</p>
<div style="clear: both;"></div>

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(0, 112, 243), rgb(243, 128, 32), transparent); margin-bottom: 20px;">

Para ir a la siguiente Fase Click aqui →→ [[Fase 6 BATMAN (Hardening y Blindaje de Seguridad (SI))]]
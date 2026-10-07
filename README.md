# 🎬 Andy Nginx Stream Pro

Reproductor HLS para Windows usando **Nginx + HLS.js**, con una ruta local `.m3u8` y controles personalizados.

El proyecto permite tener una página HTML local y utilizar Nginx como servidor HTTP/proxy para entregar un playlist HLS al reproductor.

---

## 📁 Estructura del proyecto

La instalación utilizada en este proyecto es:

```text
C:\nginx-1.31.6\
└── nginx-1.31.6\
    ├── conf\
    │   ├── nginx.conf
    │   └── spiderman2026.conf
    │
    ├── html\
    │   └── aw_mejorado.html
    │
    ├── logs\
    │   ├── access.log
    │   └── error.log
    │
    └── nginx.exe
```

---

# 1. 📥 Instalación de Nginx

Descarga una versión de Nginx para Windows y extrae el contenido, por ejemplo:

```text
C:\nginx-1.31.6\
```

La carpeta final debe contener:

```text
C:\nginx-1.31.6\nginx-1.31.6\nginx.exe
```

Comprobar:

```cmd
cd /d C:\nginx-1.31.6\nginx-1.31.6
```

Después:

```cmd
nginx.exe -v
```

Debe mostrar la versión instalada.

---

# 2. 🌐 Página HTML

La página principal está ubicada en:

```text
C:\nginx-1.31.6\nginx-1.31.6\html\aw_mejorado.html
```

El archivo contiene:

* reproductor HTML5
* HLS.js
* botón Play/Pause
* volumen
* barra de progreso
* pantalla completa
* indicador de conexión
* interfaz personalizada
* reproducción HLS

La URL local será:

```text
http://localhost/
```

o:

```text
http://localhost/aw_mejorado.html
```

---

# 3. ⚙️ Configuración de nginx.conf

Archivo:

```text
C:\nginx-1.31.6\nginx-1.31.6\conf\nginx.conf
```

Dentro de `http { }` se puede cargar la configuración adicional:

```nginx
include spiderman2026.conf;
```

Ejemplo:

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    include spiderman2026.conf;
}
```

---

# 4. 🎥 Configuración de spiderman2026.conf

Archivo:

```text
C:\nginx-1.31.6\nginx-1.31.6\conf\spiderman2026.conf
```

Configuración básica:

```nginx
server {
    listen 80;
    server_name localhost;

    root C:/nginx-1.31.6/nginx-1.31.6/html;
    index aw_mejorado.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location = /spiderman2026.m3u8 {
        proxy_pass https://SERVIDOR-HLS/ruta/master.m3u8;

        proxy_set_header Host SERVIDOR-HLS;
        proxy_set_header User-Agent "Mozilla/5.0";
        proxy_ssl_server_name on;

        proxy_buffering off;
        proxy_cache off;

        add_header Access-Control-Allow-Origin * always;
    }
}
```

> Sustituye `SERVIDOR-HLS` y la ruta por una fuente HLS que tengas autorización para utilizar.

---

# 5. 🔗 URL local del HLS

El reproductor utiliza:

```javascript
const STREAM_URL = "/spiderman2026.m3u8";
```

Por tanto, el navegador solicita:

```text
http://localhost/spiderman2026.m3u8
```

Nginx recibe esa petición y la procesa mediante:

```nginx
location = /spiderman2026.m3u8
```

El navegador no necesita conocer directamente la URL de origen.

---

# 6. ▶️ Iniciar Nginx

Abrir **CMD como administrador**.

Entrar a la carpeta:

```cmd
cd /d C:\nginx-1.31.6\nginx-1.31.6
```

Comprobar configuración:

```cmd
nginx.exe -t
```

Debe aparecer:

```text
syntax is ok
test is successful
```

Después iniciar:

```cmd
nginx.exe
```

Abrir:

```text
http://localhost/
```

---

# 7. 🔄 Recargar Nginx

Cuando se modifica una configuración:

```cmd
nginx.exe -t
```

Si todo está correcto:

```cmd
nginx.exe -s reload
```

Si Windows devuelve:

```text
OpenEvent(...) failed (5: Acceso denegado)
```

abrir CMD como administrador y repetir.

También se puede detener completamente:

```cmd
nginx.exe -s stop
```

y volver a iniciar:

```cmd
nginx.exe
```

---

# 8. 🔍 Comprobar que Nginx está funcionando

Ejecutar:

```cmd
tasklist | findstr nginx
```

Si está funcionando aparecerá algo parecido a:

```text
nginx.exe
nginx.exe
```

Comprobar la página:

```text
http://localhost/
```

---

# 9. 🧪 Comprobar el playlist HLS

Primero:

```text
http://localhost/spiderman2026.m3u8
```

Una respuesta correcta debe comenzar con:

```text
#EXTM3U
```

Por ejemplo:

```text
#EXTM3U
#EXT-X-STREAM-INF:...
playlist.m3u8
```

Un `200 OK` solamente significa que el primer playlist fue entregado correctamente.

Para reproducir HLS correctamente también deben funcionar los playlists secundarios y los segmentos.

---

# 10. 📺 HLS con múltiples calidades

Un `master.m3u8` puede contener diferentes resoluciones:

```text
#EXT-X-STREAM-INF:BANDWIDTH=849536,RESOLUTION=1152x480
index-f1-v1-a1.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=1738824,RESOLUTION=1728x720
index-f2-v1-a1.m3u8
```

El reproductor HLS.js seleccionará automáticamente una calidad dependiendo de las condiciones de reproducción.

Flujo:

```text
aw_mejorado.html
       │
       ▼
/spiderman2026.m3u8
       │
       ▼
master.m3u8
       │
       ├── index-f1-v1-a1.m3u8
       │
       └── index-f2-v1-a1.m3u8
              │
              ▼
          segmentos
              │
              ▼
            VIDEO
```

---

# 11. ⚠️ Error 404 en playlists secundarios

Si la consola muestra:

```text
index-f1-v1-a1.m3u8
Failed to load resource:
404 Not Found
```

significa que el navegador consiguió el `master.m3u8`, pero el siguiente playlist no está disponible en la ruta solicitada.

Esto es diferente de un problema del HTML.

Revisar:

```text
http://localhost/index-f1-v1-a1.m3u8
```

y:

```text
http://localhost/index-f2-v1-a1.m3u8
```

---

# 12. ⚠️ Error 403 Forbidden

Si aparece:

```text
403 Forbidden
nginx
```

revisar:

1. El `server` que realmente está cargando Nginx.
2. El `root`.
3. Las reglas `location`.
4. Los permisos.
5. El archivo `nginx.conf` utilizado por la instancia.
6. El archivo de errores.

Ver el registro:

```cmd
type logs\error.log
```

Para revisar la configuración completa cargada:

```cmd
nginx.exe -T
```

Este comando es especialmente útil cuando existen varios archivos `.conf`.

---

# 13. 🔎 Comprobar qué configuración está cargando Nginx

Ejecutar:

```cmd
nginx.exe -T
```

Buscar:

```text
spiderman2026.conf
```

También comprobar:

```text
location = /spiderman2026.m3u8
```

Si no aparece, Nginx no está incluyendo el archivo.

En ese caso revisar:

```nginx
include spiderman2026.conf;
```

dentro de:

```nginx
http {
    ...
}
```

---

# 14. 📜 Logs

Los logs están normalmente en:

```text
C:\nginx-1.31.6\nginx-1.31.6\logs\
```

Archivos importantes:

```text
access.log
error.log
```

Ver errores:

```cmd
type logs\error.log
```

Para seguir un problema de reproducción, revisar especialmente:

```text
404
403
502
504
connection refused
SSL
upstream
```

---

# 15. 🌐 CORS

Si el reproductor y el servidor utilizan diferentes orígenes, puede ser necesario permitir CORS:

```nginx
add_header Access-Control-Allow-Origin * always;
```

También se puede restringir a un dominio específico:

```nginx
add_header Access-Control-Allow-Origin "https://tudominio.com" always;
```

Para producción es preferible utilizar un origen concreto cuando sea posible.

---

# 16. 🔐 HTTPS

Para desarrollo local:

```text
http://localhost/
```

es suficiente.

Para publicar el reproductor en Internet se recomienda utilizar:

```text
https://tudominio.com/
```

y configurar TLS mediante el método correspondiente a tu servidor.

---

# 17. 🔊 Autoplay

Chrome y otros navegadores pueden bloquear:

```javascript
video.play();
```

si el usuario todavía no ha interactuado con la página.

El error:

```text
NotAllowedError:
play() failed because the user didn't interact with the document first
```

no significa necesariamente que HLS esté roto.

La solución más sencilla es permitir que el usuario pulse:

```text
▶ Play
```

También se puede utilizar reproducción muted cuando sea apropiado.

---

# 18. 🖥️ Pantalla completa

El reproductor utiliza:

```javascript
videoContainer.requestFullscreen();
```

para activar pantalla completa.

El botón correspondiente es:

```text
⛶
```

---

# 19. 🔊 Control de volumen

El reproductor cambia:

```javascript
video.muted
```

cuando se pulsa el botón de volumen.

Los estados son:

```text
🔊 sonido
🔇 silenciado
```

---

# 20. ⏱️ Barra de progreso

El reproductor calcula:

```javascript
video.currentTime
video.duration
```

y actualiza:

```text
00:00 / 00:00
```

La barra permite seleccionar una posición del vídeo cuando el contenido HLS proporciona una duración utilizable.

---

# 21. 🧹 Detener Nginx

Para detener Nginx:

```cmd
cd /d C:\nginx-1.31.6\nginx-1.31.6
nginx.exe -s stop
```

Comprobar:

```cmd
tasklist | findstr nginx
```

Si no aparece ningún proceso, Nginx está detenido.

---

# 22. 🔁 Reinicio completo

Cuando existan problemas y quieras empezar limpio:

```cmd
cd /d C:\nginx-1.31.6\nginx-1.31.6
```

Detener:

```cmd
nginx.exe -s stop
```

Comprobar:

```cmd
tasklist | findstr nginx
```

Comprobar configuración:

```cmd
nginx.exe -t
```

Iniciar:

```cmd
nginx.exe
```

Después abrir:

```text
http://localhost/
```

---

# 23. 🧰 Diagnóstico rápido

### Página no aparece

Comprobar:

```text
http://localhost/
```

y:

```cmd
tasklist | findstr nginx
```

---

### Página aparece pero vídeo no

Abrir:

```text
http://localhost/spiderman2026.m3u8
```

Debe devolver:

```text
#EXTM3U
```

---

### Master funciona pero vídeo no

Revisar los playlists secundarios.

Ejemplo:

```text
http://localhost/index-f1-v1-a1.m3u8
```

---

### Playlist secundario funciona pero vídeo no

Revisar los segmentos que aparecen dentro del playlist secundario.

---

### 404

Revisar:

```cmd
nginx.exe -T
```

y:

```text
logs/error.log
```

---

### 403

Revisar:

```text
logs/error.log
```

y verificar qué `server` está atendiendo la petición.

---

### 502 Bad Gateway

Normalmente indica un problema comunicándose con el servidor upstream.

Revisar:

```text
logs/error.log
```

---

### Autoplay bloqueado

No es necesariamente un fallo de Nginx.

Pulsar:

```text
▶ Play
```

---

# 24. 🚀 Flujo recomendado de pruebas

Siempre probar en este orden:

```text
1. Nginx
   ↓
2. localhost
   ↓
3. master.m3u8
   ↓
4. playlist secundario
   ↓
5. segmentos
   ↓
6. HLS.js
   ↓
7. vídeo
```

No conviene cambiar cinco cosas simultáneamente.

---

# 25. 📌 Comandos principales

### Entrar a Nginx

```cmd
cd /d C:\nginx-1.31.6\nginx-1.31.6
```

### Ver versión

```cmd
nginx.exe -v
```

### Comprobar configuración

```cmd
nginx.exe -t
```

### Ver configuración completa

```cmd
nginx.exe -T
```

### Iniciar

```cmd
nginx.exe
```

### Recargar

```cmd
nginx.exe -s reload
```

### Detener

```cmd
nginx.exe -s stop
```

### Ver proceso

```cmd
tasklist | findstr nginx
```

### Ver errores

```cmd
type logs\error.log
```

---

# 26. 🎯 URLs locales del proyecto

Página:

```text
http://localhost/
```

HTML:

```text
http://localhost/aw_mejorado.html
```

Master HLS:

```text
http://localhost/spiderman2026.m3u8
```

Playlist secundario:

```text
http://localhost/index-f1-v1-a1.m3u8
```

Segundo playlist:

```text
http://localhost/index-f2-v1-a1.m3u8
```

---

# 27. 🔒 Uso responsable

Utiliza únicamente streams, vídeos y servidores sobre los que tengas autorización o derechos de uso.

Nginx funciona aquí como servidor/proxy HTTP para una fuente HLS autorizada. No debe utilizarse para evadir controles de acceso, autenticación, DRM o restricciones de contenido de terceros.

---

# 28. ✅ Estado final esperado

Cuando todo está correctamente configurado:

```text
http://localhost/
        │
        ▼
aw_mejorado.html
        │
        ▼
HLS.js
        │
        ▼
/spiderman2026.m3u8
        │
        ▼
Nginx
        │
        ▼
Master HLS
        │
        ├── calidad 1
        │
        └── calidad 2
             │
             ▼
          segmentos
             │
             ▼
          Reproductor
```

El resultado final es un reproductor HLS local servido mediante Nginx, con una interfaz personalizada y controles de reproducción.

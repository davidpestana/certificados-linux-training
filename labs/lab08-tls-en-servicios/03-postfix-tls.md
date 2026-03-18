# Laboratorio 08 — TLS en servicios

## Probar acceso entre contenedores usando TLS

En los ejercicios anteriores accedimos al servidor HTTPS desde el propio entorno del host.
Ahora realizaremos la conexión desde **otro contenedor Docker**, simulando el comportamiento de un cliente independiente.

Esto permite reproducir escenarios reales de comunicación entre servicios.

Aquí a veces estarás en la terminal del Codespace y otras dentro del contenedor cliente; en cada paso te decimos dónde, para que no haya duda.

---

### Paso 1 — Identificar el contenedor del servidor NGINX

En la terminal del Codespace (host), localiza el contenedor que ejecuta el servidor HTTPS:

```bash
docker ps
```

En la salida aparecerá un contenedor basado en la imagen `nginx`.

Observa especialmente la columna:

```
PORTS
```

Debería aparecer algo similar a:

```
0.0.0.0:8443->443/tcp
```

Esto indica que el puerto HTTPS del contenedor está expuesto en el puerto `8443` del entorno.

---

### Paso 2 — Crear un contenedor cliente temporal

En el host, lanza un contenedor que actuará como cliente; se abrirá una shell y quedarás dentro del contenedor:

```bash
docker run -it --rm --network lab08-net curlimages/curl sh
```

Se abrirá una shell dentro del contenedor.

Desde este contenedor intentaremos conectarnos al servidor HTTPS.

---

### Paso 3 — Probar la conexión HTTPS desde el contenedor cliente

Como el servidor NGINX se arrancó en una red Docker *user-defined* con el alias `web.test.local`, este contenedor cliente puede resolverlo por DNS interno de Docker.

Dentro del contenedor ejecuta:

```bash
curl https://web.test.local
```

Es posible que aparezca un error relacionado con el certificado.

Esto ocurre porque el contenedor cliente no confía en la autoridad certificadora que creamos en los laboratorios.

Esto reproduce el comportamiento habitual cuando un cliente intenta conectarse a un servicio que utiliza una CA desconocida.

---

### Paso 4 — Probar la conexión ignorando la verificación del certificado

Para observar la respuesta del servidor, repite la conexión ignorando la verificación del certificado.

```bash
curl -k https://web.test.local
```

La salida debería mostrar el contenido del servidor:

```
Servidor HTTPS de prueba
```

Esto confirma que la conexión TLS funciona correctamente aunque el cliente no confíe en la CA.

---

### Paso 5 — Probar la conexión confiando en la CA del laboratorio

Abre otra terminal del Codespace y muestra el certificado raíz para copiar su contenido:

```bash
cat ~/pki-ca/ca.crt
```

Vuelve a la shell del contenedor cliente (la del paso 2) y crea ahí un archivo con ese contenido. Si tiene `nano`, úsalo; si no, `cat > ca.crt`, pega y cierra con Ctrl+D. Luego, ya dentro del contenedor, prueba la conexión indicando la CA:

Ahora realiza la conexión indicando explícitamente la CA.

```bash
curl --cacert ca.crt https://web.test.local
```

Esta vez la conexión debería realizarse sin errores de certificado.

Esto demuestra cómo los clientes pueden confiar en certificados cuando disponen de la autoridad certificadora correspondiente.

---

En este ejercicio hemos simulado una comunicación entre servicios utilizando contenedores Docker y hemos observado cómo influye la confianza en las autoridades certificadoras durante las conexiones TLS.

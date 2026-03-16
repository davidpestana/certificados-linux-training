# Laboratorio 08 — TLS en servicios

Hasta ahora hemos trabajado con certificados, claves y autoridades certificadoras desde la línea de comandos.
En este laboratorio comenzaremos a utilizar esos certificados en **servicios reales**.

Muchos servicios de infraestructura utilizan TLS para proteger sus comunicaciones.
Entre los más habituales se encuentran:

* servidores web
* servidores de correo
* APIs internas
* servicios de autenticación
* proxies y balanceadores.

En este laboratorio configuraremos servicios reales para utilizar certificados emitidos por nuestra PKI.

---

## Qué aprenderás en este laboratorio

Durante este laboratorio aprenderás a:

* configurar **HTTPS en servidores web**
* utilizar certificados en servicios reales
* verificar el funcionamiento de TLS desde el cliente
* identificar problemas comunes en configuraciones TLS.

Este laboratorio conecta el trabajo realizado en la PKI con el uso real de certificados en servicios.

---

## Conceptos clave

Cuando un servicio utiliza TLS necesita varios elementos:

**Certificado del servidor**

Identifica al servicio frente a los clientes.

**Clave privada**

Permite al servidor demostrar que es el propietario del certificado.

**Cadena de certificados**

Incluye:

* certificado del servidor
* certificado intermedio
* autoridad certificadora raíz.

Esto permite a los clientes verificar la identidad del servicio.

---

## Servicios que configuraremos

En este laboratorio trabajaremos con distintos tipos de servicios.

**NGINX**

Servidor web moderno ampliamente utilizado para servir contenido HTTPS.

Si el servidor de aplicaciones es Java, suele utilizar keystores específicos (JKS/PKCS12) para gestionar sus certificados. Puedes revisar el ejercicio `04-uso-certs-en-apps-java.md` del **Lab07** para ver cómo se configuran keystores y truststores en aplicaciones Java.

---

**Apache HTTP Server**

Servidor web muy extendido que también permite configurar TLS mediante módulos específicos.

---

**Postfix**

Servidor de correo que puede utilizar TLS para proteger comunicaciones SMTP.

---

**SSH**

Servicio de acceso remoto que puede utilizar certificados y autoridades de firma para autenticar clientes y servidores.

---

**OpenVPN**

Servicio de VPN que utiliza certificados X.509 para autenticar servidores y clientes y establecer túneles seguros.

---

Para simplificar el entorno utilizaremos **contenedores Docker**, lo que permite desplegar servicios rápidamente dentro del Codespace.

Antes de liarte con NGINX y la inspección TLS, asegúrate de haber terminado el **Lab05** (así tendrás los certificados en `~/pki-labs/web-server`). Para el ejercicio de inspección hace falta además tener el servidor NGINX del 01 en marcha. En cada ficha te indicamos si lo que toca va en la terminal del Codespace o dentro de un contenedor, para que no haya duda de dónde ejecutar cada cosa.

---

## Herramientas utilizadas

Durante los ejercicios utilizaremos herramientas como:

* **docker**
* **openssl**
* **curl**
* **ssh / ssh-keygen**
* **openvpn**

Estas herramientas permiten desplegar servicios, probar conexiones TLS y analizar certificados.

---

## Laboratorios incluidos

Este laboratorio contiene los siguientes ejercicios:

```id="z8c2v1"
01-nginx-https.md
02-apache-https.md
03-postfix-tls.md
04-ssh-certificados.md
05-openvpn-tls.md
```

Cada ejercicio aplica certificados TLS a un tipo de servicio diferente.

---

## Qué deberías comprender al terminar

Al finalizar este laboratorio deberías ser capaz de:

* configurar HTTPS en un servidor web
* utilizar certificados emitidos por una PKI
* verificar el funcionamiento de TLS en servicios reales
* identificar problemas comunes en configuraciones TLS.

Este conocimiento permitirá en los siguientes laboratorios comprender cómo se validan certificados y cómo se automatiza su gestión en sistemas reales.

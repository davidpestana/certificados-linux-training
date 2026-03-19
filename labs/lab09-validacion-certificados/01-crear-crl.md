# Laboratorio 09 — Validación de certificados

## Crear y publicar una lista de revocación (CRL)

En una infraestructura PKI no basta con emitir certificados.
También es necesario poder **revocar certificados antes de su fecha de expiración**.

Esto puede ocurrir por varios motivos:

* compromiso de una clave privada
* cambio de identidad de un servicio
* errores en la emisión de certificados.

Una forma de comunicar estas revocaciones es mediante una **CRL (Certificate Revocation List)**.

En este ejercicio generaremos una CRL utilizando la autoridad certificadora creada en los laboratorios anteriores.

---

### Paso 1 — Ir al directorio de la autoridad certificadora

Sitúate en el directorio donde creamos la PKI.

```bash id="u7b4uv"
cd ~/pki-ca
```

Comprueba que los archivos principales de la CA están disponibles.

```bash id="y0m0dn"
ls
```

Deberías ver elementos como:

```id="bmb2ow"
ca.crt
private
certs
crl
index.txt
serial
```

Estos archivos permiten a la CA gestionar certificados y revocaciones.

---

### Paso 2 — Generar una CRL inicial

Crea una lista de revocación vacía.

> Si al ejecutar el siguiente comando obtienes el error
> `cannot lookup how long until the next CRL is due`,
> significa que falta la directiva `default_crl_days` en tu `openssl-ca.cnf`.
> Abre el archivo y añade esta línea dentro de la sección `[ CA_default ]`:
>
> ```text
> default_crl_days  = 30
> ```

```bash id="kqu5hw"
openssl ca -config openssl-ca.cnf -gencrl -out crl/ca.crl
```

Comprueba que el archivo se ha generado.

```bash id="ovm08n"
ls crl
```

Deberías ver:

```id="5rc8ds"
ca.crl
```

Esta CRL aún no contiene certificados revocados.

---

### Paso 3 — Examinar la CRL generada

Visualiza el contenido de la lista de revocación.

```bash id="6p4knk"
openssl crl -in crl/ca.crl -text -noout
```

Busca una sección similar a:

```id="n2pn68"
No Revoked Certificates
```

Esto indica que actualmente ningún certificado ha sido revocado.

---

### Paso 4 — Verificar la firma de la CRL

Comprueba que la CRL ha sido firmada por la autoridad certificadora.

```bash id="scopst"
openssl crl -in crl/ca.crl -noout -issuer
```

La salida debería mostrar algo similar a:

```id="2d4kxb"
issuer=CN = Training Root CA
```

Esto confirma que la CRL pertenece a la autoridad certificadora que creamos en el laboratorio.

---

### Paso 5 — Comprobar la validez temporal de la CRL

Muestra las fechas de validez de la lista de revocación.

```bash id="j3m6u3"
openssl crl -in crl/ca.crl -noout -nextupdate -lastupdate
```

Estas fechas indican:

* cuándo se publicó la CRL
* cuándo deberá generarse una nueva versión.

Las CRL deben actualizarse periódicamente para que los clientes puedan comprobar revocaciones recientes.

---

En este ejercicio hemos generado una lista de revocación utilizando nuestra autoridad certificadora y hemos examinado su contenido y firma.

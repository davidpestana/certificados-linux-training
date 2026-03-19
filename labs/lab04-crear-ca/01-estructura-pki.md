# Laboratorio 04 — Crear una autoridad certificadora

## Preparar la estructura de una PKI

En los laboratorios anteriores hemos generado claves y certificados autofirmados.
En este laboratorio comenzaremos a construir una **infraestructura de certificación (PKI)** que nos permitirá emitir certificados para distintos servicios.

Las autoridades certificadoras utilizan una estructura de directorios específica para gestionar certificados emitidos, solicitudes y revocaciones.

---

### Paso 1 — Crear el directorio de trabajo de la CA

Crea un nuevo directorio donde construiremos la autoridad certificadora.

```bash
mkdir -p ~/pki-ca
cd ~/pki-ca
```

Comprueba que estás dentro del directorio correcto.

```bash
pwd
```

Este directorio contendrá todos los archivos relacionados con la autoridad certificadora.

---

### Paso 2 — Crear la estructura de directorios de la CA

Las implementaciones de PKI suelen utilizar una estructura de directorios para separar distintos tipos de archivos.

Crea los siguientes directorios:

```bash
mkdir certs
mkdir crl
mkdir newcerts
mkdir private
mkdir csr
```

Comprueba que los directorios se han creado correctamente.

```bash
ls -l
```

Deberías ver algo similar a:

```
certs
crl
csr
newcerts
private
```

Cada uno de estos directorios tendrá un propósito específico dentro de la PKI.

---

### Paso 3 — Crear archivos de control de la CA

Las autoridades certificadoras utilizan algunos archivos para mantener el control de los certificados emitidos.

Crea el archivo de índice de certificados.

```bash
touch index.txt
```

Crea también el archivo que almacenará el número de serie de los certificados.

```bash
echo 1000 > serial
```

Comprueba que ambos archivos existen.

```bash
ls
```

Estos archivos permiten a la autoridad certificadora llevar un registro de los certificados emitidos.

---

### Paso 4 — Crear un archivo de configuración para OpenSSL

OpenSSL utiliza archivos de configuración para definir cómo debe comportarse la autoridad certificadora.

Crea un archivo llamado `openssl-ca.cnf`.

```bash
nano openssl-ca.cnf
```

Añade el siguiente contenido básico:

```
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = .
certs             = $dir/certs
crl_dir           = $dir/crl
database          = $dir/index.txt
new_certs_dir     = $dir/newcerts
certificate       = $dir/ca.crt
serial            = $dir/serial
private_key       = $dir/private/ca.key
default_days      = 365
default_crl_days  = 30
default_md        = sha256
policy            = policy_any

[ policy_any ]
countryName             = optional
stateOrProvinceName     = optional
organizationName        = optional
commonName              = supplied
```

Guarda el archivo.

Este archivo indica a OpenSSL:

* dónde almacenar certificados
* dónde está la clave privada de la CA
* cómo deben generarse los certificados.

---

### Paso 5 — Verificar la estructura creada

Lista todo el contenido del directorio.

```bash
tree
```

Si `tree` no está instalado puedes usar:

```bash
ls -R
```

Deberías ver una estructura similar a:

```
certs/
crl/
csr/
newcerts/
private/
index.txt
serial
openssl-ca.cnf
```

Esta estructura será utilizada en los siguientes ejercicios para crear la clave de la autoridad certificadora y emitir certificados firmados.

## Lab 05 — Rotación de certificados con Ansible

En este ejercicio usarás **Ansible** para desplegar y **rotar certificados** en un servicio (nginx) que corre en un contenedor. El foco es la **rotación de certificados** automatizada con Ansible, no Ansible en abstracto.

El entorno se levanta con **Docker Compose** y volúmenes mapeados: editas en el host y, cuando necesites terminal en el contenedor, usas *attach shell*.

---

### 0. Crear y levantar el escenario con Docker Compose

En el curso se usa la convención de la carpeta **`docker/`**: tú creas una subcarpeta por escenario con su `docker-compose.yml` y, si aplica, carpetas para configs y certificados (ver [Lab 00 — Entorno con Docker Compose](../lab00-entorno/03-entorno-docker-compose.md)).

1. Crea la subcarpeta para este lab y la carpeta de certificados:

```bash
mkdir -p docker/lab10-ansible/certs
cd docker/lab10-ansible
```

2. Crea un **Dockerfile** que tenga nginx y SSH (para que Ansible se conecte por SSH). Por ejemplo:

```dockerfile
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    nginx \
    openssh-server \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /run/sshd \
    && echo "PermitRootLogin yes" >> /etc/ssh/sshd_config \
    && echo "root:lab" | chpasswd

RUN mkdir -p /etc/ssl/local && chmod 755 /etc/ssl/local

EXPOSE 22 80
CMD service ssh start && nginx -g "daemon off;"
```

3. Crea **docker-compose.yml** con un servicio `web1`, puertos 2222 (SSH) y 8080 (nginx), y un volumen para certificados:

```yaml
services:
  web1:
    build: .
    container_name: web1
    ports:
      - "2222:22"
      - "8080:80"
    volumes:
      - ./certs:/etc/ssl/local
    restart: unless-stopped
```

4. Levanta el servicio en segundo plano:

```bash
docker compose up -d
```

5. Cuando necesites **ejecutar comandos dentro del contenedor** (revisar nginx, comprobar archivos, etc.):

```bash
docker compose exec web1 bash
```

Sal con `exit`. Los configs y certificados se editan en el host en `certs/` (y otras carpetas que montes); no hace falta entrar al contenedor para eso.

6. El contenedor `web1` expone SSH en el puerto **2222** y nginx en **8080**. Ansible se conectará por SSH al puerto 2222 (usuario `root`, contraseña `lab` en este entorno de prácticas; en producción se usarían claves).

---

### 1. Preparar el entorno de Ansible

1. Crea un directorio de trabajo para el playbook (puedes usar uno al lado de `docker/` o dentro del repo):

```bash
mkdir -p ~/labs/ansible-certs
cd ~/labs/ansible-certs
```

2. Crea el inventario `hosts.ini` apuntando al contenedor levantado por Compose (SSH en localhost:2222):

```ini
[webservers]
web1 ansible_host=127.0.0.1 ansible_port=2222 ansible_user=root ansible_connection=ssh
```

Para que Ansible use contraseña en este lab de prácticas (solo entorno aislado):

```bash
pip install ansible
# Opcional: instalar sshpass para conexión por contraseña
# sudo apt install -y sshpass
# y ejecutar: ansible-playbook -i hosts.ini deploy_certs.yml --ask-pass
```

O configura autenticación por clave si lo prefieres.

---

### 2. Estructura básica del playbook

1. Crea `deploy_certs.yml` con una estructura que **despliegue certificados en el contenedor** (por ejemplo, en `/etc/ssl/local`, que en este escenario está mapeado a `docker/lab10-ansible/certs/` en el host):

```yaml
- name: Desplegar certificados TLS (rotación)
  hosts: webservers
  become: true

  vars:
    cert_source_dir: "/path/a/tus/certificados"   # ruta en el host donde están server.crt y server.key
    cert_dest_dir: "/etc/ssl/local"

  tasks:
    - name: Crear directorio de certificados
      file:
        path: "{{ cert_dest_dir }}"
        state: directory
        owner: root
        group: root
        mode: '0755'

    - name: Copiar certificado y clave
      copy:
        src: "{{ cert_source_dir }}/{{ item }}"
        dest: "{{ cert_dest_dir }}/{{ item }}"
        owner: root
        group: root
        mode: '0640'
      loop:
        - server.crt
        - server.key

    - name: Recargar nginx si cambia el certificado
      service:
        name: nginx
        state: reloaded
  handlers: []
```

2. Ajusta `cert_source_dir` a la ruta donde tengas `server.crt` y `server.key` (p. ej. salida de la PKI del curso o una copia en `docker/lab10-ansible/certs/`).

---

### 3. Comportamiento idempotente

El módulo `copy` de Ansible solo marca *changed* cuando el contenido del archivo cambia.

1. Ejecuta el playbook la primera vez:

```bash
ansible-playbook -i hosts.ini deploy_certs.yml
```

2. Vuelve a ejecutarlo sin cambiar nada. Comprueba que las tareas de copia salen en *ok* y que el servicio no se recarga si no hay cambios.

La idea es que el **recarga de nginx** solo ocurra cuando realmente se actualicen certificado o clave (rotación).

---

### 4. Simular una rotación de certificado

1. Genera un nuevo certificado para el mismo servicio con la PKI del curso.
2. Sustituye `server.crt` y `server.key` en `cert_source_dir` por las nuevas versiones.
3. Vuelve a ejecutar el playbook:

```bash
ansible-playbook -i hosts.ini deploy_certs.yml
```

Comprueba que las tareas de copia marcan *changed* y que nginx se recarga con los nuevos certificados. Si tienes el volumen mapeado, verás los archivos actualizados en `docker/lab10-ansible/certs/` en el host.

---

### 5. Relación con cron y certmonger

Piensa en un flujo en producción:

* Un script o **certmonger** renueva el certificado y lo deja en una ruta (p. ej. en el host).
* Un job de **Ansible** (manual o desde un scheduler) despliega ese certificado en los servidores o contenedores afectados.
* Los servicios se recargan de forma controlada.

Anota este flujo en un archivo de notas (`flujo-rotacion-ansible.md`).

---

### 6. Conclusiones

Al finalizar este ejercicio deberías ser capaz de:

* Levantar el escenario con **Docker Compose** (carpeta `docker/lab10-ansible`, volúmenes, `up -d`, `exec` para shell).
* Usar Ansible para **desplegar y rotar certificados** en un servicio en contenedor.
* Entender el flujo de rotación: renovación (script/certmonger) + despliegue (Ansible) + recarga del servicio.

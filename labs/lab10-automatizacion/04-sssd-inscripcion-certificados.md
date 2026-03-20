## Lab 04 — Automatización de certificados con sssd (conceptos)

En este ejercicio revisarás el papel de **sssd** en la automatización de inscripción y renovación de certificados en entornos integrados con servicios de identidad (como FreeIPA o Active Directory).

El objetivo es entender el flujo de alto nivel, aunque en el entorno del curso no se disponga necesariamente de una infraestructura IPA/AD real.

---

### 1. Rol de sssd en entornos de identidad

Investiga y anota:

* qué es `sssd` (System Security Services Daemon)
* qué tipos de backends puede utilizar (LDAP, Kerberos, IPA, AD)
* cómo se integra con otros componentes del sistema (PAM, NSS, sudo, etc.).

Puedes consultar las páginas de manual:

```bash
man sssd
man sssd.conf
```

> **¿`man sssd` no funciona?**
>
> En Codespaces y entornos mínimos `sssd` no está instalado.
> Puedes instalar el paquete para disponer de la herramienta y sus manuales:
>
> ```bash
> sudo apt update && sudo apt install -y sssd sssd-tools
> ```
>
> Si no necesitas instalar el servicio completo, puedes instalar solo la
> documentación (en algunas distribuciones):
>
> ```bash
> sudo apt install -y sssd-common   # incluye man pages
> ```
>
> Como alternativa, puedes consultar la documentación online:
>
> * [man sssd (man7.org)](https://man7.org/linux/man-pages/man8/sssd.8.html)
> * [man sssd.conf (man7.org)](https://man7.org/linux/man-pages/man5/sssd.conf.5.html)
> * [Documentación oficial de SSSD](https://sssd.io/docs/introduction.html)

---

### 2. Certificados y sssd en FreeIPA / AD

En infraestructuras como FreeIPA o Active Directory:

* los equipos y usuarios pueden tener **certificados asociados** a sus identidades
* la inscripción y renovación de certificados puede gestionarse de forma centralizada.

Analiza (a nivel conceptual):

1. Cómo se almacena la relación entre identidad y certificado.
2. Qué ventajas tiene que la renovación se gestione desde el servicio de identidad.

Anota tus conclusiones en un archivo de notas (por ejemplo `notas-sssd.md`).

---

### 3. Ejemplo de configuración relevante en sssd.conf

Revisa ejemplos de configuración de `sssd.conf` relacionados con certificados (puedes usar documentación oficial o ejemplos de tu entorno si los tienes).

Identifica parámetros típicos, por ejemplo:

* mapeo de certificados a identidades
* opciones de caché
* parámetros específicos de FreeIPA o AD.

No es necesario que apliques esta configuración en tu sistema; el objetivo es comprender qué partes de `sssd` intervienen cuando se usan certificados.

---

### 4. Flujo conceptual de inscripción y renovación

Dibuja (en papel o en un archivo de texto) un flujo simplificado donde intervengan:

1. Un host Linux unido a un dominio (IPA/AD).
2. `sssd` como componente local.
3. El servicio de identidad (servidores IPA/AD).
4. La CA corporativa.

Incluye pasos como:

* generación de clave local
* solicitud de certificado
* emisión por parte de la CA
* renovación automática antes de la expiración.

---

### 5. Integración con certmonger

En muchos entornos, `sssd` se utiliza junto con **certmonger** para gestionar:

* solicitudes de certificado hacia la CA
* renovación automática
* actualización de servicios cuando cambia un certificado.

Reflexiona sobre cómo se repartirían las responsabilidades:

* qué hace `sssd`
* qué hace `certmonger`
* qué hace la CA / servicio de identidad.

---

### 6. Conclusiones

Al finalizar este ejercicio deberías ser capaz de:

* describir el rol de `sssd` en un entorno de identidad corporativo
* explicar, a nivel conceptual, cómo se podría automatizar la inscripción y renovación de certificados
* relacionar `sssd` con otras herramientas vistas en el curso, como `certmonger`.


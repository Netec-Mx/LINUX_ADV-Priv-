# Laboratorio Servicios de Red 

## Objetivo de la práctica:
Al finalizar la práctica, serás capaz de:
- Configurar servicios esenciales 
- Gestionar puertos y sockets 
- Diagnosticar problemas de red 
- Garantizar la seguridad de la red por medio de un firewall 
- Mantener un funcionamiento robusto y seguro del sistema

## Duración aproximada:
- 110 minutos.

# Laboratorio 4

## Configuración del Entorno para los Laboratorios:

Para estos laboratorios, necesitarás al menos dos máquinas virtuales Ubuntu Server (una VM Ubuntu Cliente y una VM Ubuntu Server) [cite: 1].

* **Máquina 1: `servidor-ubuntu`** (donde configurarás los servicios y harás la mayor parte del diagnóstico)[cite: 2].
* **Máquina 2: `cliente-ubuntu`** (la máquina cliente, para probar las conexiones SSH y web)[cite: 3].

Asegúrate de que ambas máquinas puedan comunicarse entre sí (idealmente, en la misma red NAT o red interna en tu software de virtualización)[cite: 4].

---

### Laboratorio 4.1: Configuración de Servicios (SSH y NTP) y Gestión de Puertos con UFW

Este laboratorio se enfoca en configurar servicios esenciales como SSH para acceso remoto y NTP para sincronización horaria, y luego asegurar estos servicios con UFW[cite: 5].

**Objetivos:**

* Instalar y configurar el servicio SSH en `servidor-ubuntu`[cite: 6].
* Configurar la autenticación por clave SSH[cite: 6].
* Instalar y configurar el servicio NTP (`systemd-timesyncd`)[cite: 7].
* Configurar reglas de firewall UFW para SSH y verificar puertos[cite: 7].

**Máquinas:** `servidor-ubuntu` (VM o máquina real) y `cliente-ubuntu` (VM o máquina local para probar SSH)[cite: 8].

---

#### Ejercicio 4.1: Configuración Segura de SSH

**Duración estimada:** 15-20 minutos

**En `servidor-ubuntu` (el servidor que quieres acceder remotamente):**

1.  **Actualizar el sistema e instalar OpenSSH Server:**
    ```bash
    sudo apt update
    sudo apt upgrade -y
    sudo apt install openssh-server -y
    ```

2.  **Verificar el estado del servicio SSH:**
    ```bash
    systemctl status ssh
    ```
    * **Solución esperada:** Deberías ver `Active: active (running)` y `Loaded: ...; enabled; ...`. Esto confirma que SSH está corriendo y se iniciará automáticamente[cite: 10].

3.  **Configurar UFW para permitir SSH:**
    ```bash
    sudo ufw allow OpenSSH
    sudo ufw enable
    sudo ufw status verbose
    ```
    * **Solución esperada:** `Status: active` y una regla `OpenSSH ALLOW Anywhere` (o `22/tcp ALLOW Anywhere`)[cite: 12].

4.  **Hacer una copia de seguridad del archivo de configuración SSH:**
    ```bash
    sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
    ```

5.  **Editar el archivo de configuración SSH para mayor seguridad:**
    ```bash
    sudo nano /etc/ssh/sshd_config
    ```
    * Busca la línea `#Port 22` y cámbiala a un puerto diferente (ej. `Port 2222`). Descoméntala[cite: 13].
    * Busca la línea `#PermitRootLogin prohibit-password` (o `PermitRootLogin yes`) y cámbiala a `PermitRootLogin no`. Descoméntala[cite: 14].
    * Deja `PasswordAuthentication yes` por ahora. Lo cambiaremos a `no` DESPUÉS de configurar la autenticación por clave[cite: 15].

6.  **Reiniciar el servicio SSH para aplicar los cambios:**
    ```bash
    sudo systemctl restart ssh
    ```

7.  **Actualizar la regla UFW para el nuevo puerto SSH:**
    ```bash
    sudo ufw delete allow OpenSSH # Elimina la regla predeterminada
    sudo ufw allow 2222/tcp # Permite el nuevo puerto SSH (reemplaza 2222 con tu puerto)
    sudo ufw reload # Recarga las reglas
    sudo ufw status verbose # Verifica la nueva regla
    ```
    * **Solución esperada:** La regla para el puerto 22 debería haber desaparecido y la nueva regla para el puerto `2222/tcp` (o el que elegiste) debería aparecer[cite: 16].

**En `cliente-ubuntu` (o tu máquina local):**

1.  **Generar un par de claves SSH (si no tienes uno):**
    ```bash
    ssh-keygen -t rsa -b 4096 -C mi_clave_lab@ejemplo.com
    ```
    * Presiona Enter para la ubicación por defecto (`~/.ssh/id_rsa`)[cite: 17].
    * Introduce una frase de contraseña (`passphrase`) fuerte cuando se te pida. Esto protege tu clave privada[cite: 18].

2.  **Copiar la clave pública al `servidor-ubuntu`:** Reemplaza `usuario_remoto` con el nombre de usuario que usas en `servidor-ubuntu` (no root), `IP_servidor` con la IP de `servidor-ubuntu` y `2222` con tu puerto SSH[cite: 19].
    ```bash
    ssh-copy-id -p 2222 usuario_remoto@IP_servidor
    ```
    * Te pedirá la contraseña del `usuario_remoto` en `servidor-ubuntu`[cite: 20].

3.  **Probar la conexión SSH usando la clave:**
    ```bash
    ssh -p 2222 usuario_remoto@IP_servidor
    ```
    * **Solución esperada:** Deberías poder conectarte sin pedir la contraseña del usuario, solo tu frase de contraseña de la clave SSH (si la configuraste)[cite: 21].
    * Si esto falla, **NO CONTINÚES** con el siguiente paso hasta que puedas conectarte con la clave[cite: 22].
    * Revisa el archivo `/var/log/auth.log` en el servidor si tienes problemas[cite: 23].

**En `servidor-ubuntu` (una vez confirmada la conexión con clave):**

1.  **Deshabilitar la autenticación por contraseña:**
    ```bash
    sudo nano /etc/ssh/sshd_config
    ```
    * Busca la línea `PasswordAuthentication yes` y cámbiala a `PasswordAuthentication no`[cite: 24].

2.  **Reiniciar el servicio SSH:**
    ```bash
    sudo systemctl restart ssh
    ```

3.  **Probar nuevamente la conexión desde `cliente-ubuntu`:**
    ```bash
    ssh -p 2222 usuario_remoto@IP_servidor
    ```
    * **Solución esperada:** Solo deberías poder conectarte usando tu clave SSH[cite: 25].
    * Si intentaras conectar sin la clave (ej. desde otra máquina sin clave), fallaría la autenticación por contraseña[cite: 26].

---

#### Ejercicio 4.2: Configuración de NTP (`systemd-timesyncd`)

**Duración estimada:** 5-10 minutos

**En `servidor-ubuntu`:**

1.  **Verificar el estado actual del servicio NTP:**
    ```bash
    timedatectl status
    ```
    * **Solución esperada:** Deberías ver `NTP service: active` y `System clock synchronized: yes` (si tiene conexión a internet)[cite: 27].

2.  **Configurar servidores NTP personalizados (opcional):** Si deseas usar otros servidores NTP (ej. de tu región o red interna)[cite: 28].
    ```bash
    sudo nano /etc/systemd/timesyncd.conf
    ```
    * Busca la línea `NTP=` y descoméntala (elimina el `#`)[cite: 29].
    * Cambia los servidores por defecto a algunos de tu elección (ej., servidores de `pool.ntp.org` o tus propios servidores):
        ```ini
        [Time]
        NTP=0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org
        ```
    * Guarda y sal[cite: 30].

3.  **Reiniciar el servicio `systemd-timesyncd`:**
    ```bash
    sudo systemctl restart systemd-timesyncd
    ```

4.  **Verificar la sincronización de la hora y los servidores usados:**
    ```bash
    timedatectl status
    ```
    * **Solución esperada:** Deberías ver que el `NTP service` sigue `active` y, después de un momento, el reloj debería estar sincronizado[cite: 31].
    * Aunque `timedatectl` no muestra los servidores específicos que `systemd-timesyncd` está utilizando en tiempo real, puedes verificarlo en el archivo de configuración[cite: 32].

---

### Laboratorio 4.3: Gestión Avanzada de Puertos y Sockets (`ss`, `lsof`)

Este laboratorio se enfoca en usar las herramientas `ss` y `lsof` para inspeccionar el estado de los puertos y las conexiones[cite: 33].

**Objetivos:**

* Identificar puertos en estado de escucha y sus procesos asociados usando `ss`[cite: 34].
* Identificar conexiones activas usando `ss`[cite: 34].
* Usar `lsof` para encontrar qué proceso está usando un puerto específico[cite: 35].
* Instalar y configurar Apache HTTP Server para un servicio adicional[cite: 36].

**Máquinas:** `servidor-ubuntu`[cite: 36].

---

#### Ejercicio 4.3: Análisis de Puertos y Sockets con `ss`

**Duración estimada:** 10-15 minutos

1.  **Instalar Apache HTTP Server (si no lo tienes):**
    ```bash
    sudo apt install apache2 -y
    sudo ufw allow 'Apache Full'
    sudo ufw reload
    ```
    * Esto asegurará que tenemos otro servicio (además de SSH) escuchando en la red[cite: 37].

2.  **Verificar todos los puertos TCP en estado de "escucha" (LISTEN) y sus procesos:**
    ```bash
    sudo ss -ltnp
    ```
    * **Solución esperada:** Deberías ver el puerto 2222 (o tu puerto SSH personalizado) asociado a `sshd` y el puerto 80 y 443 asociado a `apache2`[cite: 38].
    * Puede haber otros puertos como 53 para DNS o 68 para DHCP si el sistema los usa[cite: 39].
        ```
        Netid State  Recv-Q Send-Q  Local Address:Port Peer Address:Port Process
        tcp   LISTEN 0      128     0.0.0.0:2222      0.0.0.0:* users:(("sshd",pid=PID_SSH,fd=FD_SSH))
        tcp   LISTEN 0      128     0.0.0.0:80        0.0.0.0:* users:(("apache2",pid=PID_APACHE,fd=FD_APACHE))
        tcp   LISTEN 0      128     0.0.0.0:443       0.0.0.0:* users:(("apache2",pid=PID_APACHE_SSL,fd=FD_APACHE_SSL))
        ```

3.  **Verificar todos los puertos UDP en estado de "escucha" (LISTEN) y sus procesos:**
    ```bash
    sudo ss -lunp
    ```
    * **Solución esperada:** Podrías ver `systemd-resolved` en el puerto 53 (para DNS local) o el puerto 68 para DHCP[cite: 40].
        ```
        Netid State  Recv-Q Send-Q  Local Address:Port Peer Address:Port Process
        udp   UNCONN 0      0       127.0.0.53%lo:53   0.0.0.0:* users:(("systemd-resolve",pid=PID_RESOLVED,fd=FD_RESOLVED))
        ```

4.  **Generar una conexión activa (en una nueva terminal o desde `cliente-ubuntu`):** Abre una nueva terminal en `servidor-ubuntu` (o conéctate por SSH desde `cliente-ubuntu` al `servidor-ubuntu` usando tu usuario y el puerto 2222)[cite: 41].
    ```bash
    ssh -p 2222 usuario_remoto@127.0.0.1 # Si lo haces desde el mismo servidor
    # O desde cliente-ubuntu:
    # ssh -p 2222 usuario_remoto@IP_servidor
    ```
    * Mantén esta conexión abierta[cite: 42].

5.  **Verificar todas las conexiones TCP activas (LISTEN y ESTABLISHED) con procesos:** En la terminal principal de `servidor-ubuntu`, ejecuta:
    ```bash
    sudo ss -atnp
    ```
    * **Solución esperada:** Además de los puertos en `LISTEN`, deberías ver una línea `ESTAB` (Established) para tu conexión SSH activa, mostrando tu IP de origen y el puerto 2222[cite: 43].
        ```
        Netid State  Recv-Q Send-Q  Local Address:Port    Peer Address:Port     Process
        ...
        tcp   ESTAB  0      0       127.0.0.1:2222        127.0.0.1:51234       users:(("sshd",pid=PID_SSH_CHILD,fd=FD_SSH_CHILD))
        # O si desde cliente-ubuntu:
        # tcp ESTAB 0 0 IP_servidor:2222 IP_cliente:PUERTO_CLIENTE users:(("sshd",pid=PID_SSH_CHILD,fd=FD_SSH_CHILD))
        ```
    * Cierra la sesión SSH de prueba.

---

#### Ejercicio 4.4: Diagnóstico con `lsof`

**Duración estimada:** 5-10 minutos

1.  **Identificar el proceso que usa el puerto 80 (HTTP):**
    ```bash
    sudo lsof -i :80
    ```
    * **Solución esperada:** Deberías ver `apache2` como el comando y su PID asociado al puerto `*:80`[cite: 44].
        ```
        COMMAND   PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
        apache2   PID_APACHE root  4u  IPv4 xxxxx      0t0  TCP *:http (LISTEN)
        ```

2.  **Identificar todos los sockets de red abiertos por el proceso `apache2`:** Primero, encuentra el PID del proceso principal de `apache2` (lo viste con `ss -ltnp` o `systemctl status apache2`)[cite: 45].
    ```bash
    sudo lsof -i -a -p <PID_del_proceso_principal_de_apache2>
    ```
    * **Solución esperada:** Verás varias líneas que corresponden a los sockets de Apache, incluyendo los de escucha en 80 y 443[cite: 45].


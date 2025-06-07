# Práctica 6.1. Gestión de permisos de archivos

Esta práctica te guiará a través de la creación y modificación de **permisos de archivos** y el uso del comando **`sudo`**.

### Objetivos de la práctica:

- Comprender cómo los permisos afectan el acceso a archivos.
- Practicar el uso seguro de `sudo`.

### Instrucciones

### Tarea 1. Crea un archivo de prueba

Paso 1. Abre tu terminal y crea un archivo simple:

    ```bash
    echo "Este es un archivo de prueba." > mi_documento.txt
    ls -l mi_documento.txt
    ```

Paso 2. Observa la salida de `ls -l`. Verás algo como `-rw-r--r--`, indicando permisos de lectura/escritura para el propietario, y solo lectura para el grupo y otros.

### Tarea 2. Intenta ejecutar el archivo

    ```bash
    ./mi_documento.txt
    ```

Verás un error de "**Permiso denegado**", porque es un archivo de texto, no un ejecutable.

### Tarea 3. Convierte el archivo en un script ejecutable y añade permisos

    ```bash
    echo "#!/bin/bash" > mi_script.sh
    echo "echo '¡Hola desde mi script!'" >> mi_script.sh
    chmod +x mi_script.sh # Añade permiso de ejecución para el propietario, grupo y otros
    ls -l mi_script.sh
    ```

Ahora deberías ver `rwxr-xr-x` en la salida. La `x` al final de cada bloque indica permiso de ejecución.

### Tarea 4. Ejecuta el script

    ```bash
    ./mi_script.sh
    ```

Deberías ver el mensaje "¡Hola desde mi script!".

### Tarea 5. Restringe permisos de escritura para "otros"

Paso 1. Vamos a quitar el permiso de escritura para la categoría "otros" (world) del script.

    ```bash
    chmod o-w mi_script.sh # Quita el permiso de escritura para "otros"
    ls -l mi_script.sh
    ```

La salida debería ser ahora `-rwxr-xr-x`. ¡Ah, espera! ¿Por qué no cambió? Porque la "w" para "otros" no estaba activada. Probemos con un archivo donde sí tenga sentido.

### Tarea 6. Restringe permisos de escritura en un documento

    ```bash
    echo "Contenido secreto" > documento_secreto.txt
    chmod o-w documento_secreto.txt # Quita el permiso de escritura para "otros"
    ls -l documento_secreto.txt
    ```

Verás que los permisos para "otros" son ahora `-r--` o similares, sin la `w`.

### Tarea 7. Cambia permisos usando notación numérica

Ahora cambiaremos el script para que solo el propietario tenga todos los permisos, y nadie más pueda hacer nada con él.

    ```bash
    chmod 700 mi_script.sh # Propietario: rwx (4+2+1=7), Grupo: --- (0), Otros: --- (0)
    ls -l mi_script.sh
    ```

La salida será `-rwx------`. Solo tú (el propietario) puedes leer, escribir y ejecutar.

---

## Uso de `sudo`

### Tarea 1. Intenta un comando de administrador sin `sudo`

    ```bash
    apt update
    ```

Verás un error que indica que necesitas ser root o usar `sudo`.

### Tarea 2. Ejecuta el mismo comando con `sudo`

    ```bash
    sudo apt update
    ```

Se te pedirá tu contraseña de usuario. Ingresa y observa cómo el comando se ejecuta con éxito, ya que `apt update` requiere privilegios elevados.

### Tarea 3. Abre un archivo de sistema con `sudo` y un editor de texto

Paso 1. El archivo `/etc/hosts` es un archivo de sistema. Intenta abrirlo sin `sudo`:

    ```bash
    nano /etc/hosts
    ```

Paso 2. Veras que `nano` abre el archivo, pero al intentar guardar (`Ctrl+O`), te dirá que no tienes permiso.

Paso 3. Ahora, ábrelo con `sudo`:

    ```bash
    sudo nano /etc/hosts
    ```

Paso 4. Se te pedirá tu contraseña. Una vez dentro, puedes hacer cambios (no guardes nada crítico para esta prueba) y verás que puedes guardar (`Ctrl+O`) sin problemas.

### Tarea 4. Entiende el archivo `/etc/sudoers`

> *¡Advertencia!* Siempre usa `sudo visudo` para editar este archivo. Un error de sintaxis puede bloquearte.

    ```bash
    sudo visudo
    ```

Paso 1. Busca la línea `%sudo ALL=(ALL:ALL) ALL`. 

Esta línea indica que cualquier usuario en el grupo `sudo` puede ejecutar cualquier comando en cualquier host como cualquier usuario. 

Paso 2. Sal del editor sin guardar (`Ctrl+X` en `nano`).

---

# Práctica 6.2. Auditoría básica

Este laboratorio te familiarizará con comandos para revisar la actividad de los usuarios y procesos en tu sistema.

## Objetivo de la práctica:

- Aprender a usar `last`, `who`, `w`, `ps aux` y a revisar logs para una auditoría básica.

---

### Tarea 1. Monitoriza la Actividad de Usuarios. Ve quién se ha conectado recientemente

    ```bash
    last
    ```

Observa la lista de sesiones. Busca tu propio usuario, cualquier "reboot" (reinicios) y la dirección IP desde donde se conectaron.

### Tarea 2. Ve quién está conectado ahora mismo

    ```bash
    who
    ```

Deberías verte a ti mismo y quizás otros usuarios si están conectados simultáneamente. Fíjate en la terminal (`tty` o `pts`) y la IP de origen.

### Tarea 3. Ve quién está conectado y qué está haciendo

    ```bash
    w
    ```

Esta es una versión más detallada de `who`. Presta atención a la columna `WHAT` para ver qué comandos están ejecutando los usuarios activos.

---

# Práctica 6.3. Revisar procesos en ejecución

### Tarea 1. Lista todos los procesos del sistema

    ```bash
    ps aux
    ```

Esta lista es muy larga. Presta atención a la columna `USER` para ver qué usuario posee cada proceso y a la columna `COMMAND` para el nombre del programa.

### Tarea 2. Busca un proceso específico (ej. SSH)

    ```bash
    ps aux | grep sshd
    ```

Esto filtrará los procesos relacionados con el demonio SSH. Observa que el `USER` es `root` o `sshd`.

---

# Práctica 6.3. Explorar los logs del sistema

### Tarea 1. Ve el log de autenticación

Este log es crucial para la seguridad, ya que registra inicios de sesión y usos de `sudo`.

    ```bash
    less /var/log/auth.log
    ```

Paso 1. Usa las flechas para desplazarte. 

Paso 2. Busca líneas que contengan "Accepted password" (inicio de sesión exitoso) o "Failed password" (intento fallido). 

Paso 3. Presiona `q` para salir de `less`.

### Tarea 2. Monitoriza el log de autenticación en tiempo real

    ```bash
    sudo tail -f /var/log/auth.log
    ```

Paso 1. Deja este comando ejecutándose. 

Paso 2. En otra terminal, intenta iniciar sesión con una contraseña incorrecta (o incluso con tu usuario y contraseña correctos) y observa cómo aparecen nuevas líneas en el log en tiempo real. 

Paso 3. Presiona `Ctrl+C` para detener `tail`.

### Tarea 3. Busca un comando `sudo` específico en los logs

    ```bash
    grep "sudo" /var/log/auth.log
    ```

Verás todas las instancias donde se ha utilizado el comando `sudo` y por qué usuario.

---

# Práctica 6.4. Hardening básico del sistema

Este laboratorio se enfoca en asegurar aspectos clave como SSH y la configuración del kernel.

## Objetivo de la práctica:

- Aplicar medidas básicas de **endurecimiento** a SSH y al kernel.

### Instrucciones

### Hardening de SSH

### Tarea 1. Haz una copia de seguridad del archivo de configuración SSH

Siempre haz una copia de seguridad antes de modificar archivos de configuración críticos.

    ```bash
    sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
    ```

### Tarea 2. Edita el archivo de configuración SSH

Vamos a cambiar el puerto, deshabilitar la autenticación con contraseña y el acceso directo de root.

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

Paso 1. Dentro del editor, busca y modifica/añade las siguientes líneas:

    ```ini
    # Cambia el puerto 22 por uno diferente, por ejemplo 2222
    Port 2222

    # Deshabilita el inicio de sesión de root directo
    PermitRootLogin no

    # Deshabilita la autenticación por contraseña (una vez que configures claves SSH)
    # PasswordAuthentication yes  # <-- Comenta esta línea
    PasswordAuthentication no     # <-- Añade esta línea

    # Si solo quieres permitir ciertos usuarios (opcional)
    # AllowUsers tu_usuario_principal
    ```

Paso 2. Guarda los cambios (`Ctrl+O`, `Enter`) y sal (`Ctrl+X`).

### Tarea 3. Reinicia el servicio SSH

    ```bash
    sudo systemctl restart sshd
    ```

Si tienes una sesión SSH activa, no la cierres todavía por si algo salió mal. Abre una nueva terminal e intenta conectarte usando el nuevo puerto.

### Tarea 4. Prueba la nueva configuración SSH

Abre una nueva terminal (o desde otra máquina) e intenta conectarte:

    ```bash
    ssh tu_usuario@localhost -p 2222 # Si estás en la misma máquina
    # O si estás en otra máquina
    ssh tu_usuario@IP_del_servidor -p 2222
    ```

Si la conexión es exitosa, ¡felicidades! Has endurecido SSH. 
Si falla, revisa `/var/log/auth.log` para ver el error y los cambios en `sshd_config.bak` para revertir si es necesario.

### Tarea 5. Instala y habilita Fail2Ban (opcional, pero muy recomendado)

    ```bash
    sudo apt install fail2ban
    sudo systemctl enable fail2ban
    sudo systemctl start fail2ban
    sudo systemctl status fail2ban # Verifica que esté corriendo
    ```

Fail2Ban monitoreará automáticamente los logs de SSH y bloqueará IPs con intentos fallidos.

---

### Configuración del Firewall (UFW)

### Tarea 1. Deniega todo el tráfico entrante por defecto y permite el saliente

    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    ```

### Tarea 2. Permite los servicios necesarios (SSH y otros)

    ```bash
    sudo ufw allow 2222/tcp # Asegúrate de usar el puerto SSH que configuraste antes
    # Si es un servidor web, también:
    # sudo ufw allow http
    # sudo ufw allow https
    ```

### Tarea 3. Habilita UFW

    ```bash
    sudo ufw enable
    ```

Se te advertirá que esto puede interrumpir conexiones SSH. Confirma con `y`.

### Tarea 4. Verifica el estado del firewall

    ```bash
    sudo ufw status verbose
    ```

Deberías ver que UFW está activo y las reglas que definiste.

---

### Parámetros del Kernel (sysctl)

### Tarea 1. Haz una copia de seguridad del archivo `sysctl.conf`

    ```bash
    sudo cp /etc/sysctl.conf /etc/sysctl.conf.bak
    ```

### Tarea 2. Edita el archivo de configuración del kernel

    ```bash
    sudo nano /etc/sysctl.conf
    ```

Añade las siguientes líneas al final del archivo. Estas son algunas recomendaciones comunes de seguridad:

    ```ini
    # Protección contra ataques SYN-Flood
    net.ipv4.tcp_syncookies = 1

    # Ignorar paquetes ICMP de broadcast (para evitar ataques Smurf)
    net.ipv4.icmp_echo_ignore_broadcasts = 1

    # Habilitar protección contra IP spoofing (Reverse Path Filtering)
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.conf.default.rp_filter = 1

    # Ignorar paquetes ICMP de redirigir (para evitar ataques de hombre en el medio)
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.default.send_redirects = 0

    # Deshabilitar aceptación de origen de ruta (para evitar ataques de ruteo)
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv4.conf.default.accept_source_route = 0

    # Habilitar protección contra escalada de privilegios por ASLR (Address Space Layout Randomization)
    kernel.randomize_va_space = 2

    # Deshabilitar reenvío de IP si no es un router
    net.ipv4.ip_forward = 0
    net.ipv6.conf.all.forwarding = 0
    ```

Guarda los cambios (`Ctrl+O`, `Enter`) y sal (`Ctrl+X`).

### Tarea 3. Aplica los cambios del kernel

    ```bash
    sudo sysctl -p
    ```
---



## Laboratorio 8: Preparación del Entorno

Este laboratorio preparará tu servidor Ubuntu para los ejercicios siguientes.

**Objetivo:** Configurar una VM Ubuntu limpia para los laboratorios.

**Requisitos:** Una máquina virtual (VirtualBox, VMware, KVM) o un servidor en la nube con Ubuntu Server 22.04 LTS (o similar) instalado.

### Pasos:

1.  **Actualizar el sistema** (si no se ha actualizado antes):
    * Inicia sesión en tu servidor Ubuntu.
    * Actualiza los paquetes y el sistema:

        ```bash
        sudo apt update
        sudo apt upgrade -y
        sudo apt dist-upgrade -y # Para asegurar la actualización de dependencias
        sudo apt autoremove -y # Para limpiar paquetes obsoletos
        ```

2.  **Instalar herramientas básicas necesarias**:
    * Instala `git` (necesario para la sección 8.2):

        ```bash
        sudo apt install git -y
        ```

    * Instala `net-tools` y `iputils-ping` (para `netstat` y `ping` en validaciones):

        ```bash
        sudo apt install net-tools iputils-ping -y
        ```

    * Instala `htop` y `glances` (herramientas de monitoreo visual):

        ```bash
        sudo apt install htop glances -y
        ```

3.  **Crear algunos archivos de configuración y scripts de ejemplo**:
    * Creamos un directorio para configuraciones simuladas:

        ```bash
        sudo mkdir -p /etc/lab_configs
        echo "DATABASE_HOST=localhost" | sudo tee /etc/lab_configs/db_config.conf
        echo "APP_VERSION=1.0.0" | sudo tee /etc/lab_configs/app_settings.conf
        ```

    * Creamos un directorio para scripts simulados:

        ```bash
        sudo mkdir -p /usr/local/lab_scripts
        echo '#!/bin/bash' | sudo tee /usr/local/lab_scripts/daily_report.sh
        echo 'echo "Reporte diario generado: $(date)"' | sudo tee -a /usr/local/lab_scripts/daily_report.sh
        sudo chmod +x /usr/local/lab_scripts/daily_report.sh

        echo '#!/bin/bash' | sudo tee /usr/local/lab_scripts/cleanup.sh
        echo 'echo "Limpieza iniciada: $(date)"' | sudo tee -a /usr/local/lab_scripts/cleanup.sh
        echo 'find /tmp -type f -name "*.tmp" -delete' | sudo tee -a /usr/local/lab_scripts/cleanup.sh
        sudo chmod +x /usr/local/lab_scripts/cleanup.sh
        ```

    * Verifica que los archivos existan:

        ```bash
        ls -l /etc/lab_configs/
        ls -l /usr/local/lab_scripts/
        ```

4.  **Crear un directorio de destino para los backups (local)**:
    * `sudo mkdir -p /mnt/lab_backups`
    * `sudo chown -R $USER:$USER /mnt/lab_backups` # Asegura que tu usuario pueda escribir aquí
    * **¡Importante!** En un escenario real, este directorio estaría en un almacenamiento externo, NAS o servidor de backups dedicado.

5.  **Revisión del entorno:** Asegúrate de que no haya errores durante la instalación y creación de archivos.

---

# Práctica 8.1. Backups de configuración y scripts (rsync, tar, scp)

## Objetivo de la práctica: 
- Practicar la creación de backups de configuraciones y scripts usando las utilerías `rsync`, `tar` y `scp`.

**Requisitos:** Servidor Ubuntu configurado en Laboratorio 8. Además, para la sección de `scp` y `rsync` remoto, necesitarás un segundo servidor Ubuntu (VM o en la nube) con acceso SSH (usuario y contraseña o clave SSH). Llamaremos a este `servidor_remoto`.

### Backup Local con rsync

### Instrucciones:

### Tarea 1. Ejecutar rsync para copiar configuraciones

Paso 1. Asegúrate de estar en el directorio de tu usuario (por ejemplo, `cd ~`).

    * `rsync -avh --exclude='*.bak' /etc/lab_configs/ /mnt/lab_backups/lab_configs_rsync_local/`

Paso 2. Observa la salida: `rsync` mostrará los archivos que está copiando.

### Tarea 2. Ejecutar rsync para copiar scripts
    
    * `rsync -avh /usr/local/lab_scripts/ /mnt/lab_backups/lab_scripts_rsync_local/`

### Tarea 3. Verificar el backup

    * `ls -l /mnt/lab_backups/lab_configs_rsync_local/`
    * `ls -l /mnt/lab_backups/lab_scripts_rsync_local/`

Comprueba que los archivos de ejemplo estén presentes.

### Tarea 4. Modificar un archivo y ver el efecto de rsync

Paso 1. Modifica uno de los archivos de configuración de origen:

        ```bash
        echo "NEW_SETTING=true" | sudo tee -a /etc/lab_configs/app_settings.conf
        ```

Paso 2. Vuelve a ejecutar el comando `rsync` para las configuraciones:

        ```bash
        rsync -avh --exclude='*.bak' /etc/lab_configs/ /mnt/lab_backups/lab_configs_rsync_local/
        ```

Paso 3. Observa la salida: Ahora `rsync` solo debería mostrar el archivo `app_settings.conf` como modificado.

Paso 4. Verifica el contenido en el backup:

        ```bash
        cat /mnt/lab_backups/lab_configs_rsync_local/app_settings.conf
        ```

### Backup Local con tar

### Instrucciones

### Tarea 1. Crear un backup tar.gz de las configuraciones y scripts
    
    * `DATE=$(date +%Y%m%d-%H%M%S)`
    * `tar -czPv -f /mnt/lab_backups/configs_and_scripts_"$DATE".tar.gz /etc/lab_configs/ /usr/local/lab_scripts/`
    * **Explicación:**
        * `DATE=$(date +%Y%m%d-%H%M%S)`: Crea una estampa de tiempo para el nombre del archivo.
        * `-c`: Crea un nuevo archivo.
        * `-z`: Comprime el archivo con `gzip`.
        * `-P`: Preserva las rutas absolutas (útil para restaurar).
        * `-v`: Muestra el progreso (verbose).
        * `-f <nombre_archivo>`: Especifica el nombre del archivo de salida.
        * `/etc/lab_configs/ /usr/local/lab_scripts/`: Son los directorios a incluir en el backup.

### Tarea 2. Verificar el archivo de backup
    
    * `ls -lh /mnt/lab_backups/`
    * Deberías ver el archivo `.tar.gz` con la estampa de tiempo.

### Tarea 3. Verificar el contenido del tar.gz (sin extraer)
    
    * `tar -tf /mnt/lab_backups/configs_and_scripts_"$DATE".tar.gz`
    * Esto listará los archivos dentro del tarball.

### Tarea 4. Simular una restauración (opcional, en un directorio temporal)
    
    * `mkdir -p /tmp/restore_test`
    * `tar -xzPv -f /mnt/lab_backups/configs_and_scripts_"$DATE".tar.gz -C /tmp/restore_test`
    * `ls -l /tmp/restore_test/etc/lab_configs/`
    * `ls -l /tmp/restore_test/usr/local/lab_scripts/`
    * `rm -rf /tmp/restore_test`

### Backup Remoto con scp

**Objetivo:** Copiar un archivo o directorio de configuración/scripts a un servidor remoto de forma segura.

**Requisitos:**
* Asegúrate de tener el `servidor_remoto` configurado.
* En tu servidor local de Ubuntu, genera una clave SSH y copia la clave pública al `servidor_remoto` para permitir el acceso sin contraseña (¡MUY recomendado para automatización!).
    * En tu servidor local:

        ```bash
        ssh-keygen -t rsa -b 4096 -C "lab_backup_user@local_server"
        ssh-copy-id -i ~/.ssh/id_rsa.pub usuario_remoto@IP_SERVIDOR_REMOTO
        ```

    * Prueba el acceso SSH: `ssh usuario_remoto@IP_SERVIDOR_REMOTO`

### Instrucciones

### Tarea 1. Crear un directorio de backup en el servidor remoto
    * Conéctate al `servidor_remoto` y crea el directorio:

        ```bash
        ssh usuario_remoto@IP_SERVIDOR_REMOTO "mkdir -p /mnt/remote_lab_backups/scp_backups"
        ```

### Tarea 2. Copiar un archivo de configuración específico con scp

Paso 1. Desde tu servidor local:

        ```bash
        scp /etc/lab_configs/db_config.conf usuario_remoto@IP_SERVIDOR_REMOTO:/mnt/remote_lab_backups/scp_backups/
        ```

Paso 2. Verifica en el `servidor_remoto`:

        ```bash
        ssh usuario_remoto@IP_SERVIDOR_REMOTO "ls -l /mnt/remote_lab_backups/scp_backups/"
        ```

### Tarea 3. Copiar un directorio completo de scripts con scp
    * Desde tu servidor local:

        ```bash
        scp -r /usr/local/lab_scripts/ usuario_remoto@IP_SERVIDOR_REMOTO:/mnt/remote_lab_backups/scp_backups/
        ```

    * Verifica en el `servidor_remoto`:

        ```bash
        ssh usuario_remoto@IP_SERVIDOR_REMOTO "ls -l /mnt/remote_lab_backups/scp_backups/lab_scripts/"
        ```

### Backup Remoto con rsync

**Requisitos:** Misma configuración SSH que en el Laboratorio 8.1.3.

### Instrucciones:

### Tarea 1. Crear directorios de backup para rsync en el servidor remoto
    * `ssh usuario_remoto@IP_SERVIDOR_REMOTO "mkdir -p /mnt/remote_lab_backups/rsync_backups/lab_configs_rsync_remote"`
    * `ssh usuario_remoto@IP_SERVIDOR_REMOTO "mkdir -p /mnt/remote_lab_backups/rsync_backups/lab_scripts_rsync_remote"`

### Tarea 2. Ejecutar rsync para copiar configuraciones al servidor remoto
    * Desde tu servidor local:

        ```bash
        rsync -avzh --exclude='*.bak' /etc/lab_configs/ usuario_remoto@IP_SERVIDOR_REMOTO:/mnt/remote_lab_backups/rsync_backups/lab_configs_rsync_remote/
        ```

    * Verifica en el `servidor_remoto`:

        ```bash
        ssh usuario_remoto@IP_SERVIDOR_REMOTO "ls -l /mnt/remote_lab_backups/rsync_backups/lab_configs_rsync_remote/"
        ```

### Tarea 3. Ejecutar rsync para copiar scripts al servidor remoto
    * Desde tu servidor local:

        ```bash
        rsync -avzh /usr/local/lab_scripts/ usuario_remoto@IP_SERVIDOR_REMOTO:/mnt/remote_lab_backups/rsync_backups/lab_scripts_rsync_remote/
        ```

    * Verifica en el `servidor_remoto`:

        ```bash
        ssh usuario_remoto@IP_SERVIDOR_REMOTO "ls -l /mnt/remote_lab_backups/rsync_backups/lab_scripts_rsync_remote/"
        ```

### Tarea 4. Modificar un archivo en el servidor local y resincronizar
Paso 1. Modifica un script en el servidor local:

        ```bash
        echo 'echo "FIN del script: $(date)"' | sudo tee -a /usr/local/lab_scripts/daily_report.sh
        ```

Paso 2. Vuelve a ejecutar el `rsync` para los scripts al servidor remoto:

        ```bash
        rsync -avzh /usr/local/lab_scripts/ usuario_remoto@IP_SERVIDOR_REMOTO:/mnt/remote_lab_backups/rsync_backups/lab_scripts_rsync_remote/
        ```

Paso 3. Observa la salida: Solo el archivo `daily_report.sh` debería aparecer como transferido.
Paso 4. Verifica el contenido en el `servidor_remoto`:

        ```bash
        ssh usuario_remoto@IP_SERVIDOR_REMOTO "cat /mnt/remote_lab_backups/rsync_backups/lab_scripts_rsync_remote/daily_report.sh"
        ```

---

# Práctica 8.2. Versionado con Git para scripts y configuraciones

## Objetivo de la práctica: 

- Establecer un flujo de trabajo básico para versionar configuraciones y scripts usando Git, con un repositorio remoto.

**Requisitos:**
* Servidor Ubuntu configurado en Laboratorio 8.
* Una cuenta en GitHub, GitLab, Bitbucket o un servidor Git privado.
* Un token de acceso personal (PAT) si usas GitHub/GitLab y no quieres usar SSH para todos los pasos (aunque SSH es preferible para automatización).

### Instrucciones

### Tarea 1. Crear un nuevo repositorio remoto (en GitHub/GitLab)

Paso 1. Ve a tu proveedor de Git y crea un nuevo repositorio llamado, por ejemplo, `server-configs-and-scripts`.
Paso 2. No inicialices con `README` ni `.gitignore`. Lo haremos localmente.
Paso 3. Obtén la URL de clonación (HTTPS o SSH). Para este laboratorio, usaremos HTTPS inicialmente para simplificar, pero en automatización SSH es lo ideal.

### Tarea 2. Configurar Git en el servidor Ubuntu

Configura tu nombre de usuario y correo electrónico de Git:

        ```bash
        git config --global user.name "Tu Nombre"
        git config --global user.email tu_email@example.com
        ```

### Tarea 3. Crear un directorio para el repositorio local y copiar configuraciones/scripts

Paso 1. Crea un directorio donde trabajaremos con Git, que no es directamente donde están los archivos en uso.
    * `sudo mkdir -p /opt/git_configs`
    * `sudo chown -R $USER:$USER /opt/git_configs`
    * `cd /opt/git_configs`

Paso 2. Copia los archivos de ejemplo a este directorio:

        ```bash
        cp -r /etc/lab_configs/ . # Copia el contenido de /etc/lab_configs a /opt/git_configs/lab_configs
        cp -r /usr/local/lab_scripts/ . # Copia el contenido de /usr/local/lab_scripts a /opt/git_configs/lab_scripts
        ```

Paso 3. Verifica la copia: `ls -l`

### Tarea 4. Inicializar el repositorio Git local y hacer el primer commit
    
    * `cd /opt/git_configs`
    * `git init`
    * `git add .`
    * `git commit -m "Initial commit of lab configs and scripts"`

### Tarea 5. Conectar el repositorio local con el remoto y hacer el primer push
    
 **Opción A: Usando HTTPS** (si tu repositorio lo permite).
        * `git branch -M main` # Cambiar la rama principal a 'main'
        * `git remote add origin https://github.com/tu_usuario/server-configs-and-scripts.git` # Reemplaza con tu URL
        * `git push -u origin main`
        * Se te pedirá tu nombre de usuario y token de acceso personal (PAT) de GitHub/GitLab.

 **Opción B: Usando SSH** (recomendado para producción y automatización).

Si aún no lo hiciste, genera una clave SSH en tu servidor Ubuntu (`ssh-keygen`).

Añade la clave pública a tu cuenta o repositorio en GitHub/GitLab como "Deploy Key" (con permisos de escritura).

        * `git branch -M main`
        * `git remote add origin git@github.com:tu_usuario/server-configs-and-scripts.git` # Reemplaza con tu URL SSH
        * `git push -u origin main`

Si es la primera vez, se te pedirá confirmar el fingerprint del servidor.

### Tarea 6. Verificar el repositorio remoto

Abre tu navegador y ve a la URL de tu repositorio en GitHub/GitLab. Deberías ver los archivos `lab_configs/` y `lab_scripts/` subidos.

### Tarea 7. Realizar un cambio y subirlo

Paso 1. Modifica un archivo en tu repositorio local:

        ```bash
        echo "LOG_LEVEL=DEBUG" >> /opt/git_configs/lab_configs/app_settings.conf
        ```

Paso 2. Verifica el cambio (`git status`, `git diff`).
Paso 3. Añade, commitea y sube el cambio:

        ```bash
        git add lab_configs/app_settings.conf
        git commit -m "Add LOG_LEVEL to app_settings"
        git push origin main
        ```

Paso 4. Verifica el cambio en el repositorio remoto.

### Tarea 8. Simular un cambio en el remoto (por otro usuario) y jalarlo

Paso 1. Desde la interfaz web de GitHub/GitLab, edita el archivo `lab_scripts/daily_report.sh` y añade un comentario, luego commitea.

Paso 2. Vuelve a tu servidor Ubuntu.

Paso 3. Jala los cambios:

        ```bash
        cd /opt/git_configs
        git pull origin main
        ```

Paso 4. Verifica que el cambio se haya reflejado localmente:

        ```bash
        cat /opt/git_configs/lab_scripts/daily_report.sh
        ```

### Tarea 9. Crear un script de despliegue básico

Este script simulará cómo aplicarías los cambios del repositorio Git a las ubicaciones reales.

    * `sudo nano /usr/local/bin/deploy_from_git.sh`

Paso 1. Pega el siguiente contenido:

        ```bash
        #!/bin/bash

        REPO_PATH="/opt/git_configs"
        LOG_FILE="/var/log/deploy_from_git.log"

        log() {
            echo "$(date +'%Y-%m-%d %H:%M:%S') - $1" | sudo tee -a "$LOG_FILE"
        }

        log "Iniciando despliegue de configuraciones y scripts desde Git..."

        cd "$REPO_PATH" || { log "Error: No se pudo cambiar al directorio del repositorio Git. Abortando."; exit 1; }

        log "Jalando los últimos cambios del repositorio..."
        git pull origin main >> "$LOG_FILE" 2>&1
        if [ $? -ne 0 ]; then
            log "Error: Fallo al jalar los cambios de Git. Consulta $LOG_FILE para más detalles."
            exit 1
        fi
        log "Cambios jalados exitosamente."

        log "Copiando configuraciones a /etc/lab_configs/..."
        sudo rsync -av --delete "$REPO_PATH/lab_configs/" "/etc/lab_configs/" >> "$LOG_FILE" 2>&1
        if [ $? -ne 0 ]; then log "Advertencia: Fallo al copiar /etc/lab_configs/."; fi

        log "Copiando scripts a /usr/local/lab_scripts/..."
        sudo rsync -av --delete "$REPO_PATH/lab_scripts/" "/usr/local/lab_scripts/" >> "$LOG_FILE" 2>&1
        if [ $? -ne 0 ]; then log "Advertencia: Fallo al copiar /usr/local/lab_scripts/."; fi

        # Aquí iría la lógica para reiniciar servicios si los archivos relevantes cambiaron
        # Por ejemplo:
        # if grep -q "LOG_LEVEL" /etc/lab_configs/app_settings.conf; then
        #     log "app_settings.conf modificado, simulando reinicio de servicio de aplicación..."
        #     # sudo systemctl restart mi_aplicacion_servicio
        # fi

        log "Despliegue de Git completado."
        ```

Paso 2. Guarda y hazlo ejecutable.

        ```bash
        sudo chmod +x /usr/local/bin/deploy_from_git.sh
        ```

Paso 3. Ejecuta el script de despliegue:

        ```bash
        sudo /usr/local/bin/deploy_from_git.sh
        ```

Paso 4. Verifica que los cambios se hayan aplicado a los directorios originales.

        ```bash
        cat /etc/lab_configs/app_settings.conf
        ```

Paso 5. Revisa el log: 

`cat /var/log/deploy_from_git.log`

---

# Práctica 8.3. Control de cambios en el sistema

## Objetivo de la práctica: 

- Aprender a rastrear y monitorear los cambios realizados en el sistema, más allá de Git.

**Requisitos:** Servidor Ubuntu configurado en Laboratorio 8.

### Instrucciones

### Tarea 1. Revisar logs del sistema (journalctl)

Paso 1. Busca eventos recientes y errores:
        `journalctl -xe`
        
Paso 2. Filtra por tiempo (última hora):
        `journalctl --since "1 hour ago"`

Paso 3. Filtra por servicio (ej. ssh):
        `journalctl -u sshd.service`

Paso 4: Simula un cambio: Intenta iniciar sesión SSH un par de veces con una contraseña incorrecta y luego revisa los logs de autenticación:

        ```bash
        tail -f /var/log/auth.log # En otra terminal
        # En la primera terminal, intenta ssh tu_usuario@localhost con contraseña incorrecta
        # Luego, revisa el tail de /var/log/auth.log
        ```

### Tarea 2. Monitorear la instalación y desinstalación de paquetes (dpkg.log, apt/history.log)
    
Paso 1. Revisa el historial de paquetes instalados:
        `grep " install " /var/log/dpkg.log`
    
Paso 2. Revisa las operaciones de apt:
        `cat /var/log/apt/history.log`

Paso 3. Simula un cambio: Instala un paquete nuevo:

        ```bash
        sudo apt install cowsay -y
        ```

Paso 4. Revisa de nuevo los logs para ver el cambio.

### Paso 3. Monitorear modificaciones de archivos (find)

Paso 1. Encuentra archivos modificados en un período de tiempo:

        ```bash
        # Encuentra archivos modificados en los últimos 30 minutos en /etc/
        sudo find /etc/ -type f -mmin -30 -print0 | xargs -0 ls -l
        ```

Paso 2. Simula un cambio: Edita un archivo en `/etc/` que no esté en tu repositorio Git de prueba, por ejemplo, `/etc/sudoers.d/README`:

        ```bash
        sudo nano /etc/sudoers.d/README
        # Añade una línea al final como "Laboratorio de cambios."
        # Guarda y sal
        ```

Paso 3. Vuelve a ejecutar el comando `find` y observa que aparece el archivo modificado.

### Tarea 4. Auditoría de comandos (history)

Paso 1. Revisa tu historial de comandos (solo para tu usuario):
        `history`

Paso 2. Busca comandos específicos:
        `history | grep "sudo apt"`

> **Nota:** En producción, esto es útil para auditorías rápidas, pero las herramientas de auditoría de sistema (como `auditd`) son más robustas para rastrear la actividad de todos los usuarios y procesos.

### Tarea 5. Comparar archivos (diff)

Paso 1. Simula un cambio: Modifica el archivo `/etc/lab_configs/db_config.conf` manualmente (no vía Git):

        ```bash
        sudo nano /etc/lab_configs/db_config.conf
        # Cambia DATABASE_HOST=localhost a DATABASE_HOST=127.0.0.1
        # Guarda y sal
        ```

Paso 2. Compara la versión actual con la del backup más reciente de rsync que hiciste.

        ```bash
        diff /etc/lab_configs/db_config.conf /mnt/lab_backups/lab_configs_rsync_local/db_config.conf
        ```

La salida mostrará las diferencias.

---

# Práctica 8.4. Validaciones antes de reinicios o actualizaciones

## Objetivo de la práctica:
- Realizar una serie de verificaciones críticas antes de un reinicio o una actualización importante para asegurar la estabilidad del sistema.

**Requisitos:** Servidor Ubuntu configurado en Laboratorio 8.

### Instrucciones

### Tarea 1. Chequeo de recursos del sistema

Carga del sistema y procesos:

        * `uptime`
        * `htop` # Observa la carga de CPU, RAM y los procesos principales. Sal con F10 o 'q'.
        * `glances` # Otra buena opción visual. Sal con 'q'.
    * Uso de memoria:
        `free -h`
    * Uso de disco:
        * `df -h`
        * `du -sh /var/log/` # Revisa logs grandes
    * Conectividad de red:
        * `ping -c 4 google.com` # Prueba conectividad externa
        * `ip a` # Verifica interfaces y direcciones IP
        * `netstat -tulnp` # Puertos abiertos y escuchando

### Tarea 2. Estado de los servicios críticos

Paso 1. Lista servicios activos:
        `sudo systemctl list-units --type=service --state=running`
    
Paso 2. Verifica servicios específicos (simulados o reales si los tienes):
        * `sudo systemctl status ssh` # El servicio SSH debe estar activo
        * # Si instalaste Apache2 o Nginx en algún momento:
        * # `sudo systemctl status apache2`
        * # `sudo systemctl status nginx`

Paso 3. Busca servicios fallidos:
        `sudo systemctl list-units --type=service --state=failed`

### Tarea 3. Revisión de logs del sistema y aplicaciones
    
Logs recientes (errores, advertencias):
        `journalctl -xe --since "2 hours ago" | grep -i "error\\|fail\\|warn"`

Logs de aplicaciones simuladas: (Si tienes un servidor web o DB real, revisa sus logs)
        * # Por ejemplo, si tuvieras un servicio web, revisaría `/var/log/apache2/error.log`
        * # O un servicio de aplicación personalizado: `tail -n 50 /var/log/mi_aplicacion.log`

### Tarea 4. Validación de configuraciones (ej. Nginx/Apache - si los tienes)

Si tienes Nginx instalado:
        `sudo nginx -t`

Si tienes Apache2 instalado:
        `sudo apachectl configtest`

Simulación de configuración de ejemplo:

        ```bash
        echo "Validando configuración de app_settings.conf..."
        if grep -q "LOG_LEVEL" /etc/lab_configs/app_settings.conf; then
            echo "LOG_LEVEL encontrado. OK."
        else
            echo "Advertencia: LOG_LEVEL no encontrado en app_settings.conf. ¡Revisar!"
        fi
        ```

### Tarea 5. Chequeo de actualizaciones pendientes (antes de actualizar)
    * `sudo apt update`
    * `sudo apt list --upgradable`
    * Analiza la lista: ¿Hay kernels, librerías críticas (libc, systemd) o servicios de aplicaciones que se vayan a actualizar? Esto te dará una idea del impacto potencial.

### Tarea 6. Simular la actualización

Paso 1. **¡No ejecutes `sudo apt upgrade -y` aún!** Este paso es solo para ver qué pasaría.
    
Paso 2. Simula la actualización para ver los paquetes que se instalarán, actualizarán o eliminarán:

        ```bash
        sudo apt upgrade --assume-no-install --assume-no-remove --dry-run
        # O más simplemente
        sudo apt upgrade -s
        ```

Paso 3. Analiza la salida. Presta atención a cualquier mensaje sobre dependencias rotas o paquetes que serán eliminados.

### Tarea 7. Verificación de versiones de software (importante para compatibilidad)

Si tus aplicaciones dependen de versiones específicas de PHP, Python, Node.js, etc., verifica las versiones actuales.

    * `php -v` # Si tienes PHP instalado
    * `python3 --version`
    * `node -v` # Si tienes Node.js instalado

### Tarea 8. Puntos de restauración / Backups

Paso 1. Confirma la existencia de backups recientes:

  * `ls -lh /mnt/lab_backups/`

> **Recuerda:** Los backups deben ser externos al servidor de producción.

**Decisión:** Basado en estas validaciones, decide si es seguro proceder con el reinicio o la actualización, o si se necesita más investigación/solución de problemas.

---

## Práctica 8.5. Planificación de mantenimiento sin impacto

**Objetivo:** Simular y entender cómo se logra el mantenimiento sin impacto en un entorno de producción, centrándose en el concepto de redundancia y despliegue gradual.

**Requisitos:**
* Dos servidores Ubuntu: Tu `servidor_local` (el de los laboratorios anteriores) y el `servidor_remoto` (que usaste para backups remotos). Ambos deben estar configurados de manera similar.
* Conocimientos básicos de Apache2 o Nginx: Instalaremos y configuraremos un servidor web simple para simular el servicio.
* Concepto de balanceador de carga: No implementaremos un balanceador de carga real en este laboratorio (es un tema complejo), pero simularemos su comportamiento.

**Escenario simulado:** Tienes un servicio web (servidor Apache2/Nginx) distribuido en dos nodos, y quieres actualizar el sistema operativo en uno de ellos sin interrupción del servicio.

### Configurar el servicio web redundante (Simulado)

### Tarea 1. Instalar y configurar Apache2 en ambos servidores (`servidor_local` y `servidor_remoto`):

En ambos servidores:

        ```bash
        sudo apt install apache2 -y
        sudo systemctl enable apache2
        sudo systemctl start apache2
        ```

Paso 1. Crea una página `index.html` distintiva en cada servidor para identificar cuál está sirviendo la solicitud:

        En `servidor_local`:

            ```bash
            echo "<h1>Hello from Server ONE (Local)</h1>" | sudo tee /var/www/html/index.html
            ```

        En `servidor_remoto`:

            ```bash
            echo "<h1>Hello from Server TWO (Remote)</h1>" | sudo tee /var/www/html/index.html
            ```

Paso 2. Verifica que los servidores web son accesibles: Desde tu máquina local (no desde las VMs), abre un navegador y ve a la IP de tu `servidor_local` y luego a la IP de tu `servidor_remoto`. Deberías ver los mensajes "Hello from Server ONE" y "Hello from Server TWO" respectivamente.

### Tarea 2. Simular el balanceador de carga

Imagina que tienes un balanceador de carga (como Nginx en modo proxy inverso, HAProxy, o un AWS ELB) que dirige el tráfico a `servidor_local` y `servidor_remoto`.

Para simularlo, en lugar de acceder directamente a las IPs, alternaremos manualmente entre ellas para representar que el tráfico puede ir a cualquiera.

### Realizar mantenimiento sin impacto (Simulado)

### Tarea 1. Comunicación (Simulada)

Imagina que has notificado a tu equipo y usuarios sobre la ventana de mantenimiento.

### Tarea 2. Sacar el servidor ONE (Local) del "Balanceador de carga"

En un entorno real, irías a la consola de tu balanceador de carga y sacarías el `servidor_local` del grupo de servidores activos.

**Simulación:** Pretendemos que el tráfico ya no va al `servidor_local`. Ahora, si hicieras solicitudes, todas irían al `servidor_remoto`.

**Pruébalo:** Desde tu máquina local, refresca varias veces la IP de tu `servidor_local` en tu navegador. Debería funcionar. Ahora, imagina que el balanceador ha redirigido todo el tráfico al `servidor_remoto`, y solo accedes a la IP del `servidor_remoto` para verificar que el servicio sigue funcionando.

### Tarea 3. Validaciones pre-mantenimiento en servidor ONE (Local)

Paso 1. En tu `servidor_local`, realiza las validaciones aprendidas en la práctica 8.4.
        
        * `uptime`, `htop`, `df -h`, `free -h`
        * `sudo systemctl status apache2` (debería estar activo)
        * `sudo apt list --upgradable` (revisa qué se va a actualizar)
        * `sudo apachectl configtest`

Paso 2. Revisa `journalctl -xe` para errores.

Paso 3. Confirma que el servidor está en buen estado ANTES de la actualización.

### Tarea 4. Realizar la actualización en servidor ONE (Local)

Ahora que el `servidor_local` está fuera de servicio (para el balanceador) y validado, procede con la actualización.

    * En `servidor_local`:

        ```bash
        sudo apt update
        sudo apt upgrade -y
        sudo apt dist-upgrade -y
        sudo apt autoremove -y
        ```

Si la actualización incluyó un kernel, te pediría reiniciar. Para este laboratorio, vamos a simular un reinicio si es necesario, o simplemente reinicia el servicio Apache.

        ```bash
        sudo systemctl restart apache2 # O sudo reboot si la actualización lo requirió
        ```

### Tarea 5. Validaciones post-mantenimiento en servidor ONE (Local)

Después de la actualización/reinicio del servicio, realiza las mismas validaciones que hiciste antes para asegurar que el `servidor_local` está funcionando correctamente.

        * `uptime` (si reiniciaste), `htop`, `df -h`, `free -h`
        * `sudo systemctl status apache2` (debería estar activo)
        * `sudo apachectl configtest`
Revisa `journalctl -xe` y los logs de Apache (`/var/log/apache2/error.log`).

> **¡Muy importante!** Desde tu máquina local, accede directamente a la IP de tu `servidor_local` para confirmar que tu página web vuelve a cargarse correctamente.

### Tarea 6. Volver a poner el servidor ONE (Local) en el "Balanceador de carga"

En un entorno real, una vez que el `servidor_local` ha sido validado post-mantenimiento, lo volverías a añadir al grupo de servidores activos en tu balanceador de carga.
   
**Simulación:** Ahora ambas IPs (`servidor_local` y `servidor_remoto`) están en servicio.

### Tarea 7. Repetir el proceso para Servidor TWO (Remote)

Paso 1. Si tuvieras más nodos, ahora sacarías el `servidor_remoto` del "balanceador de carga".

Paso 2. Realizarías las validaciones pre-mantenimiento en el `servidor_remoto`.

Paso 3. Harías las actualizaciones en el `servidor_remoto`.

Paso 4. Realizarías las validaciones post-mantenimiento en el `servidor_remoto`.

Paso 5. Finalmente, volverías a poner el `servidor_remoto` en el "balanceador de carga".

### Resultado esperado
Al final de este laboratorio, habrás simulado el proceso de actualización de un componente de un sistema distribuido sin interrupción del servicio. La clave es la **redundancia** (tener múltiples servidores haciendo lo mismo) y la capacidad de **sacar un nodo de servicio**.

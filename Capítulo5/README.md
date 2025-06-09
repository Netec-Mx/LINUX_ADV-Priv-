# Práctica 5.1. Shell Scripting con Bash

## Objetivo de la práctica:
- Crear un script Bash para automatizar la gestión de usuarios y directorios.

### Prerrequisitos:

- Una máquina virtual o servidor con **Ubuntu** (preferiblemente una instalación limpia o un entorno de pruebas).
- Acceso a la terminal como usuario normal y con privilegios `sudo`.
- Conocimientos básicos de navegación en el sistema de archivos de Linux.

### Escenario:

Necesitas un script que cree un nuevo usuario de sistema, establezca un directorio personal con permisos específicos, y luego genere un informe básico.

### Instrucciones:

### Tarea 1. Crear el Script

Paso 1. Abre tu editor de texto favorito (por ejemplo, `nano`):

    ```bash
    nano /tmp/crear_usuario.sh
    ```

Paso 2. Pega el siguiente contenido:

    ```bash
    #!/bin/bash

    # Script: crear_usuario.sh
    # Descripción: Crea un nuevo usuario de sistema, su directorio personal y genera un informe.

    # Asegurarse de que el script se ejecute como root
    if [ "$(id -u)" -ne 0 ]; then
        echo "Este script debe ejecutarse como root. Usa 'sudo'."
        exit 1
    fi

    # 1. Solicitar el nombre de usuario
    read -p "Ingresa el nombre del nuevo usuario: " NOMBRE_USUARIO

    # Validar que se ingresó un nombre de usuario
    if [ -z "$NOMBRE_USUARIO" ]; then
        echo "Error: El nombre de usuario no puede estar vacío."
        exit 1
    fi

    # 2. Crear el usuario (sin directorio home ni shell de login para un usuario de sistema simple)
    echo "Creando usuario $NOMBRE_USUARIO..."
    if ! useradd -r -s /usr/sbin/nologin "$NOMBRE_USUARIO"; then
        echo "Error al crear el usuario $NOMBRE_USUARIO. Podría ya existir."
        exit 1
    fi
    echo "Usuario $NOMBRE_USUARIO creado exitosamente."

    # 3. Crear un directorio específico para este usuario en /var/data (ej. para logs o datos específicos)
    DIRECTORIO_DATOS="/var/data/$NOMBRE_USUARIO"
    echo "Creando directorio de datos: $DIRECTORIO_DATOS"
    mkdir -p "$DIRECTORIO_DATOS"

    # 4. Asignar permisos: propietario el nuevo usuario, grupo el mismo usuario, solo el propietario puede escribir
    chown "$NOMBRE_USUARIO":"$NOMBRE_USUARIO" "$DIRECTORIO_DATOS"
    chmod 750 "$DIRECTORIO_DATOS" # rwx para propietario, rx para grupo, nada para otros

    echo "Permisos establecidos para $DIRECTORIO_DATOS: $(stat -c '%a %U:%G' "$DIRECTORIO_DATOS")"

    # 5. Generar un informe simple
    REPORTE="/var/log/reporte_usuarios_$(date +%Y%m%d_%H%M%S).log"
    echo "--- Reporte de Creación de Usuario ---" >> "$REPORTE"
    echo "Fecha y Hora: $(date)" >> "$REPORTE"
    echo "Usuario Creado: $NOMBRE_USUARIO" >> "$REPORTE"
    echo "UID: $(id -u "$NOMBRE_USUARIO")" >> "$REPORTE"
    echo "GID: $(id -g "$NOMBRE_USUARIO")" >> "$REPORTE"
    echo "Directorio de Datos: $DIRECTORIO_DATOS" >> "$REPORTE"
    echo "Permisos del Directorio: $(stat -c '%a' "$DIRECTORIO_DATOS") ($NOMBRE_USUARIO puede escribir, otros solo leer)" >> "$REPORTE"
    echo "--- Fin del Reporte ---" >> "$REPORTE"

    echo "Script finalizado. Informe generado en: $REPORTE"
    ```

Paso 3. Da permisos de ejecución.

    ```bash
    chmod +x /tmp/crear_usuario.sh
    ```

Paso 4. Ejecuta el Script.

    ```bash
    sudo /tmp/crear_usuario.sh
    ```

Cuando te lo pida, ingresa un nombre de usuario (por ejemplo, `appuser`).

Paso 5. Verifica lo siguiente:

- Verifica si el usuario se creó: `id appuser` (o el nombre que usaste).
- Verifica si el directorio se creó y tiene los permisos correctos: `ls -ld /var/data/appuser`.
- Revisa el archivo de reporte en `/var/log/` (busca uno que empiece con `reporte_usuarios_`).

### Preguntas para la reflexión:

- *¿Qué sucede si intentas ejecutar el script sin `sudo`?*
- *¿Por qué es importante usar rutas absolutas dentro del script?*
- *¿Cómo podrías modificar el script para eliminar un usuario y su directorio?*

---

# Práctica 5.2. Automatización de tareas (`at`, `cron`, etc.)

## Objetivo de la práctica:

- Programar tareas de una sola vez y recurrentes usando `at` y `cron`.

### Escenario:

Necesitas enviar un mensaje de prueba una sola vez en 5 minutos y luego configurar una tarea diaria para limpiar un directorio de logs.

### Instrucciones

### Tarea 1. Tarea única con `at`

Paso 1. Programa un mensaje para que aparezca en tu terminal (usando `wall`) en 5 minutos.

    ```bash
    at now + 5 minutes
    at> echo "¡Tarea 'at' ejecutada a las $(date +%H:%M)!" | wall
    at> <Ctrl+D>
    ```

Paso 2. Verifica la cola de `at`.

    ```bash
    atq
    ```

Espera 5 minutos y observa si el mensaje aparece en tu terminal (si estás logueado).

### Tarea 2. Tarea Recurrente con `cron`

Paso 1. Crea un script de limpieza de logs.

    ```bash
    nano /usr/local/bin/limpiar_logs.sh
    ```

Paso 2. Pega el siguiente contenido.

    ```bash
    #!/bin/bash
    # Script: limpiar_logs.sh
    # Descripción: Elimina archivos de log antiguos de un directorio específico.

    LOG_DIR="/var/log/mi_app_logs" # Cambia esto si quieres probar en otro lugar
    DAYS_OLD="+7" # Archivos modificados hace más de 7 días

    # Crear el directorio si no existe (para la prueba)
    mkdir -p "$LOG_DIR"
    touch "$LOG_DIR/log_$(date +%Y%m%d_%H%M%S).txt" # Crear un log de prueba

    echo "--- Ejecutando limpieza de logs en $LOG_DIR: $(date) ---"
    find "$LOG_DIR" -type f -mtime "$DAYS_OLD" -delete
    echo "--- Limpieza finalizada ---"

    # Redirigir la salida a un archivo para auditoría del cron job
    echo "Script limpiar_logs.sh ejecutado con éxito en $(date)" >> /var/log/cron_limpieza.log
    ```

Paso 3. Da permisos de ejecución al script.

    ```bash
    sudo chmod +x /usr/local/bin/limpiar_logs.sh
    ```

Paso 4. Añade tarea a crontab. Abre tu crontab personal para edición:

    ```bash
    crontab -e
    ```

Paso 5. Añade la siguiente línea al final del archivo para ejecutar el script cada 5 minutos (para propósitos de prueba; en producción, lo harías una vez al día o a la semana).

    ```cron
    */5 * * * * /usr/local/bin/limpiar_logs.sh
    ```

Paso 6. Guarda y cierra el archivo.

Paso 7. Verifica la tarea cron.

    ```bash
    crontab -l
    ```

Paso 8. Observa la ejecución. Espera unos 5 minutos y luego verifica el archivo de log:

    ```bash
    cat /var/log/cron_limpieza.log
    ```

También puedes ver los logs del sistema para ver la ejecución del cron:

    ```bash
    grep CRON /var/log/syslog | tail -n 10
    ```

Paso 9. Crea algunos archivos de prueba en `/var/log/mi_app_logs` y avanza el reloj del sistema para simular el paso del tiempo y ver cómo `find` los elimina si tienen más de 7 días (o reduce `DAYS_OLD` a `+0` para probar con archivos nuevos).

### Preguntas para la reflexión:

- *¿Cuál es la principal diferencia entre `at` y `cron`?*
- *¿Por qué es importante usar rutas absolutas en los scripts de cron?*
- *¿Qué sucede con la salida de un script de cron si no la rediriges?*

---

# Práctica 5.3. Auditoría de Logs (`awk`, `grep`, etc.)

## Objetivo de la práctica.

- Utilizar `grep` y `awk` para extraer información relevante de archivos de log del sistema.

### Escenario:

Necesitas analizar el log de autenticación (`/var/log/auth.log`) para encontrar intentos de login fallidos y los usuarios/IPs involucrados.

### Instrucciones.

### Tarea 1. Generar datos de prueba (simulados)

Paso 1. Abre una nueva terminal y falla algunos intentos de login con usuarios inexistentes o contraseñas incorrectas para generar entradas en `/var/log/auth.log`.

Paso 2. Intenta `ssh usuario_inexistente@localhost` (y presiona `Ctrl+C` si te pide contraseña).

Paso 3. Intenta `sudo -i` y luego introduce una contraseña incorrecta varias veces.

### Tarea 2. Auditoría con `grep`

Paso 1. Busca todos los intentos de login fallidos.

    ```bash
    grep -i "failed password" /var/log/auth.log | tail -n 10
    ```

Paso 2. Cuenta los intentos fallidos.

    ```bash
    grep -i "failed password" /var/log/auth.log | wc -l
    ```

Paso 3. Busca intentos fallidos desde una IP específica (por ejemplo, `127.0.0.1`).

    ```bash
    grep "Failed password" /var/log/auth.log | grep "from 127.0.0.1"
    ```

### Tarea 3. Auditoría con `awk` (extracción y análisis)

Paso 1. Extrae usuario y IP de los intentos fallidos.

    ```bash
    grep "Failed password" /var/log/auth.log | awk '{print "Usuario:", $(NF-5), "desde IP:", $(NF-3)}'
    ```

> Nota: Los números de campo pueden variar ligeramente dependiendo de la versión de Ubuntu o la configuración del log. `$(NF-X)` es útil para contar desde el final si el formato es variable.

Paso 2. Cuenta los intentos fallidos por usuario.

    ```bash
    grep "Failed password" /var/log/auth.log | awk '{users[$(NF-5)]++} END {for (u in users) print u, users[u]}' | sort -nr -k2
    ```

Esto te mostrará cuántas veces falló el login cada usuario, ordenado de mayor a menor.

Paso 3. Cuenta los intentos fallidos por IP de origen.

    ```bash
    grep "Failed password" /var/log/auth.log | awk '{ips[$(NF-3)]++} END {for (ip in ips) print ip, ips[ip]}' | sort -nr -k2
    ```

Esto te mostrará cuántas veces hubo un intento fallido desde cada IP.

Paso 4. Filtra por fecha específica (por ejemplo, 24 de Mayo).

    ```bash
    awk '$1 == "May" && $2 == "24" && /Failed password/ {print}' /var/log/auth.log
    ```

(Ajusta la fecha y el mes según la fecha actual del log.)

### Preguntas para la reflexión:

- *¿Cuándo usarías `grep` en lugar de `awk` para una tarea de auditoría de logs, y viceversa?*
- *¿Cómo puedes combinar `grep` y `awk` de manera efectiva en un **pipeline**?*
- *¿Qué otros tipos de información de seguridad podrías buscar en `/var/log/auth.log`?*

---

# Práctica 5.4. Uso de `jq` (JSON)

## Objetivo de la práctica:

- Procesar y filtrar archivos de log en formato JSON usando `jq`.

### Escenario:

Tienes un log de aplicación ficticio en formato JSON y necesitas extraer información específica y filtrar eventos.

### Instrucciones

### Tarea 1. Instalar `jq`

    ```bash
    sudo apt update
    sudo apt install jq -y
    ```

### Tarea 2. Crear un archivo de log JSON de ejemplo

    ```bash
    nano /tmp/app_log.json
    ```

Pega el siguiente contenido (cada objeto JSON en una línea separada para simular un log):

    ```json
    {"timestamp": "2025-05-24T10:00:00Z", "level": "INFO", "source": "auth_service", "message": "User 'alice' logged in", "user_id": "user-101", "ip_address": "192.168.1.10", "event_id": "abc1"}
    {"timestamp": "2025-05-24T10:01:30Z", "level": "ERROR", "source": "api_gateway", "message": "Failed to connect to backend", "status_code": 500, "request_path": "/api/v1/data", "event_id": "def2"}
    {"timestamp": "2025-05-24T10:02:15Z", "level": "INFO", "source": "payment_service", "message": "Transaction processed successfully", "user_id": "user-102", "amount": 25.50, "currency": "USD", "event_id": "ghi3"}
    {"timestamp": "2025-05-24T10:03:05Z", "level": "WARNING", "source": "db_service", "message": "High disk usage warning", "disk_path": "/var/lib/mysql", "percentage": 92, "event_id": "jkl4"}
    {"timestamp": "2025-05-24T10:04:40Z", "level": "ERROR", "source": "auth_service", "message": "Invalid credentials for 'bob'", "user_id": "user-103", "ip_address": "203.0.113.5", "event_id": "mno5"}
    {"timestamp": "2025-05-24T10:05:10Z", "level": "INFO", "source": "auth_service", "message": "User 'charlie' logged out", "user_id": "user-104", "event_id": "pqr6"}
    ```

### Tarea 3. Visualizar JSON legible (`.`)

    ```bash
    cat /tmp/app_log.json | jq .
    ```

Observa cómo `jq` formatea el JSON para una mejor lectura.

### Tarea 4. Extraer campos específicos

Paso 1. Muestra el `timestamp` y el `message` de cada entrada:

    ```bash
    cat /tmp/app_log.json | jq '.timestamp, .message'
    ```

Paso 2. Muestra el `level` y el `source`:

    ```bash
    cat /tmp/app_log.json | jq '.level, .source'
    ```

### Tarea 5. Filtrar por valor de campo (`select()`)

Paso 1. Muestra solo las entradas de log con `level` "ERROR".

    ```bash
    cat /tmp/app_log.json | jq 'select(.level == "ERROR")'
    ```

Paso 2. Muestra logs del `source` "auth_service".

    ```bash
    cat /tmp/app_log.json | jq 'select(.source == "auth_service")'
    ```

### Tarea 6. Combinar extracción y filtrado

Paso 1. Muestra el mensaje y el usuario (`user_id`) de todos los *logins* de "INFO".

    ```bash
    cat /tmp/app_log.json | jq 'select(.level == "INFO" and .message | contains("logged in")) | .message, .user_id'
    ```

> Nota: `contains()` es una función de `jq` para buscar una subcadena.

### Tarea 7. Transformar la salida (`{}` para objetos nuevos, `[]` para *arrays*)

Paso 1. Crea un resumen de cada log como un objeto JSON más pequeño.

    ```bash
    cat /tmp/app_log.json | jq '{Time: .timestamp, Severity: .level, Details: .message}'
    ```

Paso 2. Crea una lista (*array*) de todos los mensajes de error.

    ```bash
    cat /tmp/app_log.json | jq '[select(.level == "ERROR") | .message]'
    ```

### Tarea 8. Cuenta ocurrencias (con otras herramientas)

Paso 1. Cuenta cuántas veces aparece cada `level` de log.

    ```bash
    cat /tmp/app_log.json | jq -r '.level' | sort | uniq -c | sort -nr
    ```

(Aquí `jq -r` para salida "raw" sin comillas, luego `sort` y `uniq -c` para contar y `sort -nr` para ordenar numéricamente descendente.)

### Preguntas para la reflexión:

- *¿Por qué `jq` es superior a `grep` o `awk` para procesar archivos de log JSON?*
- *¿Cómo combinarías `jq` con un script Bash para automatizar la extracción de informes diarios de logs JSON?*
- *Imagina un log donde el `message` contiene un JSON anidado. ¿Cómo usarías `jq` para acceder a una clave dentro de ese JSON anidado?*

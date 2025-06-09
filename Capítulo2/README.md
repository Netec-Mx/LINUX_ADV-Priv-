# Práctica 2.1. Gestión y monitoreo de procesos y servicios (systemd)

## Objetivos de la práctica:

- Identificar y listar procesos en ejecución en el sistema.
- Monitorear el consumo de recursos (CPU, memoria) de los procesos en tiempo real.
- Utilizar los comandos básicos de `systemctl` para verificar el estado, iniciar, detener y reiniciar servicios del sistema.
- Comprender cómo habilitar y deshabilitar servicios para el inicio automático.
  
### Configuración inicial:
- Asegúrate de tener una máquina virtual Ubuntu o un sistema Linux con acceso `sudo`.
- Abre una terminal.

### Instrucciones

### Tarea 1. Explora los procesos en ejecución

Paso 1. Abre una terminal.
Paso 2. Usa `ps aux` para ver una lista de todos los procesos en ejecución en el sistema. Observa las columnas de CPU (`%CPU`) y memoria (`%MEM`).

    ```bash
    ps aux | head -n 10
    ```

Paso 3. Instala `htop` si no lo tienes (es una versión mejorada de `top`).

    ```bash
    sudo apt install htop
    ```

Paso 4. Ejecuta `htop` y observa cómo se actualiza la información en tiempo real. Intenta ordenar por uso de CPU (`F6` y selecciona `PERCENT_CPU`) o memoria (`PERCENT_MEM`). Presiona `q` para salir.

    ```bash
    htop
    ```

### Tarea 2.  Gestiona un servicio de ejemplo (usaremos el servicio SSH - `sshd`)

Paso 1. Verifica el estado actual del servicio SSH.

    ```bash
    systemctl status sshd
    ```

    (Observa la línea "Active:", debería decir `active (running)`)

Paso 2. Detene el servicio SSH.

    ```bash
    sudo systemctl stop sshd
    ```

Paso 3. Verifica que se detuvo.

    ```bash
    systemctl status sshd
    ```

    (Debería mostrar `inactive (dead)`)

Paso 4. Intenta conectar vía SSH a tu propia máquina (desde otra terminal o usando `ssh localhost`).

    ```bash
    ssh localhost
    ```

    (Debería fallar la conexión, indicando que el servicio no está disponible).

Paso 5. Inicia el servicio SSH.

    ```bash
    sudo systemctl start sshd
    ```

Paso 6. Verifica que se inicio.

    ```bash
    systemctl status sshd
    ```

    (Debería mostrar `active (running)`)

Paso 7. Intenta conectar vía SSH de nuevo.

    ```bash
    ssh localhost
    ```

    (Ahora debería conectarse correctamente. Escribe `exit` para salir de la sesión SSH).

Paso 8. Reinicia el servicio SSH (útil para aplicar cambios en la configuración).

    ```bash
    sudo systemctl restart sshd
    ```

Paso 9. Verifica su estado después de reiniciar.

    ```bash
    systemctl status sshd
    ```

### Tarea 3. Gestiona el inicio automático de un servicio (Habilitar/Deshabilitar)

Paso 1. Verifica si SSH está habilitado para iniciar al arranque.

    ```bash
    systemctl is-enabled sshd
    ```

    (Normalmente dirá `enabled`).

Paso 2. Deshabilita SSH para que NO inicie automáticamente al reiniciar (¡Sólo haz esto si tienes acceso físico a la máquina o no dependes de SSH para acceso remoto!).

    ```bash
    sudo systemctl disable sshd
    systemctl is-enabled sshd # Debería decir 'disabled'
    ```

Paso 3. Habilita SSH de nuevo para que inicie automáticamente al reiniciar.

    ```bash
    sudo systemctl enable sshd
    systemctl is-enabled sshd # Debería decir 'enabled'
    ```

<br/>

---

# Práctica 2.2. Instalación, actualización y mantenimiento de paquetes

## Objetivos de la práctica:

- Dominar los comandos básicos del gestor de paquetes `apt` para actualizar el sistema.
- Instalar y eliminar paquetes de software de forma segura.
- Realizar tareas de limpieza del sistema para liberar espacio y mantener la eficiencia.

### Instrucciones:

### Tarea 1. Actualiza el índice de paquetes y el sistema

Paso 1. Utiliza el siguiente comando antes de instalar o actualizar algo.

    ```bash
    sudo apt update # Descarga la información más reciente sobre los paquetes disponibles
    ```

Paso 2. Actualiza los paquetes instalados a sus últimas versiones disponibles.

    ```bash
    sudo apt upgrade # Revisa la lista de paquetes a actualizar y confirma con 'y' si estás de acuerdo.
    ```

### Tarea 2.  Instala un nuevo paquete de prueba (ej. `neofetch`, una herramienta que muestra información del sistema de forma vistosa)

    ```bash
    sudo apt install neofetch
    ```

Verifica la instalación:

    ```bash
    neofetch # Ejecuta el comando para ver la información del sistema.
    ```

### Tarea 3. Busca información sobre un paquete

    ```bash
    apt show neofetch # Muestra detalles sobre el paquete neofetch
    apt search editor # Busca paquetes relacionados con "editor"
    ```

### Tarea 4.  Elimina el paquete de prueba

Paso 1. Elimina el paquete, pero conserva sus archivos de configuración.

    ```bash
    sudo apt remove neofetch
    ```

Paso 2. Elimina el paquete junto con sus archivos de configuración (limpieza completa).

    ```bash
    sudo apt purge neofetch
    neofetch # Debería decir "command not found"
    ```

### Tarea 5. Realiza una limpieza del sistema (mantenimiento)

Paso 1. Elimina paquetes que se instalaron como dependencias y ya no son necesarios.

    ```bash
    sudo apt autoremove
    ```

Paso 2. Limpia el caché local de paquetes descargados (`.deb`).

    ```bash
    sudo apt clean # Esto libera espacio en disco eliminando los archivos .deb
    ```

 Paso 3. Realiza una actualización completa del sistema (si hay cambios mayores de dependencias o nuevas versiones de kernel/distribución).

    ```bash
    sudo apt full-upgrade # Revisa con cuidado los cambios y confirma con 'y'
    ```

---

# Práctica 2.3. Monitoreo y análisis básico de logs del sistema con `journalctl`

## Objetivos de la práctica:

- Comprender la importancia de los logs del sistema para el diagnóstico.
- Utilizar `journalctl` para visualizar, filtrar y buscar información relevante en los logs.
- Monitorear logs en tiempo real para observar eventos mientras ocurren.

### Instrucciones

### Tarea 1. Visualiza los logs recientes del sistema

Paso 1. Muestra todos los logs desde el último arranque (o persistentes si están configurados para ello).

    ```bash
    journalctl
    ```

Paso 2. Muestra los logs más recientes primero (desde el final del registro).

    ```bash
    journalctl -e
    ```

Paso 3. Muestra los logs en orden inverso cronológico (los más recientes arriba).

    ```bash
    journalctl -r
    ```

### Tarea 2.  Filtra logs por unidad (servicio) o por ejecutable

Paso 1. Muestra logs solo del servicio SSH.

    ```bash
    journalctl -u ssh.service
    ```

Paso 2. Muestra logs solo del ejecutable `sudo`.

    ```bash
    journalctl /usr/bin/sudo
    ```

### Tarea 3. Filtra logs por prioridad (gravedad del mensaje)

Paso 1. Muestra solo mensajes de error.

    ```bash
    journalctl -p err
    ```

Paso 2. Muestra mensajes de advertencia (`warning`) y de niveles más graves (`err`, `crit`, etc.)

    ```bash
    journalctl -p warning
    ```

> Notas: Las prioridades van de 0 a 7, siendo 0 `emerg` (emergencia) el más grave y 7 `debug` el menos grave.

### Tarea 4. Filtra logs por período de tiempo

Paso 1. Muestra logs desde el inicio del día de hoy.

    ```bash
    journalctl --since "today"
    ```

 Paso 2. Muestra logs de la última hora.

    ```bash
    journalctl --since "1 hour ago"
    ```

Paso 3. Muestra logs entre dos fechas y horas específicas.

    ```bash
    journalctl --since "YYYY-MM-DD HH:MM:SS" --until "YYYY-MM-DD HH:MM:SS"
    # Ejemplo: journalctl --since "2025-05-21 10:00:00" --until "2025-05-21 10:30:00"
    ```

### Tarea 5. Sigue los logs en tiempo real (monitoreo en vivo)

    ```bash
    journalctl -f # Presiona Ctrl+C para salir
    ```

Mientras `journalctl -f` está corriendo en una terminal, en otra terminal, realiza alguna acción que genere logs:

Paso 1. Intenta iniciar sesión en SSH (incluso con credenciales incorrectas para generar un error).
Paso 2. Reinicia un servicio (ej. `sudo systemctl restart cron`).

Esto te permitirá ver cómo los nuevos mensajes de log aparecen instantáneamente.

También puedes seguir los logs de un servicio específico en tiempo real.

    ```bash
    journalctl -u ssh.service -f
    ```

### Tarea 6. Revisa logs de arranques anteriores (si el journal está configurado para ser persistente)

    ```bash
    journalctl --list-boots # Lista los arranques del sistema con sus IDs y fechas
    journalctl -b -1 # Muestra los logs del arranque anterior al actual
    journalctl -b 0 # Muestra los logs del arranque actual
    ```


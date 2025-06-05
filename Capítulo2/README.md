# Laboratorio Gestión de Procesos, Systemd y Software

## Objetivo de la práctica:
Al finalizar la práctica, serás capaz de:
- Gestionar procesos y demonios
- Manejar servicios básicos
- Instalar y actualizar paquetes
- Mantener el sistema (limpieza de caché)
- Realizar journalctl básico

## Duración aproximada:
- 80 minutos.

### Configuración Inicial:
* Asegúrate de tener una máquina virtual Ubuntu o un sistema Linux con acceso `sudo`.
* Abre una terminal.

# Laboratorio 2

---

### Laboratorio 2.1: Gestión y Monitoreo de Procesos y Servicios (systemd)

#### Objetivos del Laboratorio:

* Identificar y listar procesos en ejecución en el sistema.
* Monitorear el consumo de recursos (CPU, memoria) de los procesos en tiempo real.
* Utilizar los comandos básicos de `systemctl` para verificar el estado, iniciar, detener y reiniciar servicios del sistema.
* Comprender cómo habilitar y deshabilitar servicios para el inicio automático.

#### Solución (Pasos):

1.  **Explora los procesos en ejecución:**
    * Abre una terminal.
    * Usa `ps aux` para ver una lista de todos los procesos en ejecución en el sistema. Observa las columnas de CPU (`%CPU`) y memoria (`%MEM`).

    ```bash
    ps aux | head -n 10
    ```

    * Instala `htop` si no lo tienes (es una versión mejorada de `top`):

    ```bash
    sudo apt install htop
    ```

    * Ejecuta `htop` y observa cómo se actualiza la información en tiempo real. Intenta ordenar por uso de CPU (`F6` y selecciona `PERCENT_CPU`) o memoria (`PERCENT_MEM`). Presiona `q` para salir.

    ```bash
    htop
    ```

2.  **Gestiona un servicio de ejemplo (usaremos el servicio SSH - `sshd`):**
    * **Verificar el estado actual del servicio SSH:**

    ```bash
    systemctl status sshd
    ```

    (Observa la línea "Active:", debería decir `active (running)`)

    * **Detener el servicio SSH:**

    ```bash
    sudo systemctl stop sshd
    ```

    * **Verifica que se detuvo:**

    ```bash
    systemctl status sshd
    ```

    (Debería mostrar `inactive (dead)`)

    * **Intenta conectar vía SSH a tu propia máquina (desde otra terminal o usando `ssh localhost`):**

    ```bash
    ssh localhost
    ```

    (Debería fallar la conexión, indicando que el servicio no está disponible).

    * **Iniciar el servicio SSH:**

    ```bash
    sudo systemctl start sshd
    ```

    * **Verifica que se inició:**

    ```bash
    systemctl status sshd
    ```

    (Debería mostrar `active (running)`)

    * **Intenta conectar vía SSH de nuevo:**

    ```bash
    ssh localhost
    ```

    (Ahora debería conectarse correctamente. Escribe `exit` para salir de la sesión SSH).

    * **Reiniciar el servicio SSH (útil para aplicar cambios en la configuración):**

    ```bash
    sudo systemctl restart sshd
    ```

    * **Verifica su estado después de reiniciar:**

    ```bash
    systemctl status sshd
    ```

3.  **Gestiona el inicio automático de un servicio (Habilitar/Deshabilitar):**
    * **Verifica si SSH está habilitado para iniciar al arranque:**

    ```bash
    systemctl is-enabled sshd
    ```

    (Normalmente dirá `enabled`).

    * **Deshabilita SSH para que NO inicie automáticamente al reiniciar (¡Sólo haz esto si tienes acceso físico a la máquina o no dependes de SSH para acceso remoto!):**

    ```bash
    sudo systemctl disable sshd
    systemctl is-enabled sshd # Debería decir 'disabled'
    ```

    * **Habilita SSH de nuevo para que inicie automáticamente al reiniciar:**

    ```bash
    sudo systemctl enable sshd
    systemctl is-enabled sshd # Debería decir 'enabled'
    ```

---

### Laboratorio 2.2: Instalación, Actualización y Mantenimiento de Paquetes

#### Objetivos del Laboratorio:

* Dominar los comandos básicos del gestor de paquetes `apt` para actualizar el sistema.
* Instalar y eliminar paquetes de software de forma segura.
* Realizar tareas de limpieza del sistema para liberar espacio y mantener la eficiencia.

#### Solución (Pasos):

1.  **Actualiza el índice de paquetes y el sistema:**
    * Siempre el primer paso antes de instalar o actualizar algo.

    ```bash
    sudo apt update # Descarga la información más reciente sobre los paquetes disponibles
    ```

    * Actualiza los paquetes instalados a sus últimas versiones disponibles.

    ```bash
    sudo apt upgrade # Revisa la lista de paquetes a actualizar y confirma con 'y' si estás de acuerdo.
    ```

2.  **Instala un nuevo paquete de prueba (ej. `neofetch`, una herramienta que muestra información del sistema de forma vistosa):**

    ```bash
    sudo apt install neofetch
    ```

    * **Verifica la instalación:**

    ```bash
    neofetch # Ejecuta el comando para ver la información del sistema.
    ```

3.  **Busca información sobre un paquete:**

    ```bash
    apt show neofetch # Muestra detalles sobre el paquete neofetch
    apt search editor # Busca paquetes relacionados con "editor"
    ```

4.  **Elimina el paquete de prueba:**
    * **Elimina el paquete, pero conserva sus archivos de configuración:**

    ```bash
    sudo apt remove neofetch
    ```

    * **Elimina el paquete junto con sus archivos de configuración (limpieza completa):**

    ```bash
    sudo apt purge neofetch
    neofetch # Debería decir "command not found"
    ```

5.  **Realiza una limpieza del sistema (mantenimiento):**
    * **Elimina paquetes que se instalaron como dependencias y ya no son necesarios:**

    ```bash
    sudo apt autoremove
    ```

    * **Limpia el caché local de paquetes descargados (`.deb`):**

    ```bash
    sudo apt clean # Esto libera espacio en disco eliminando los archivos .deb
    ```

    * **Realiza una actualización completa del sistema (si hay cambios mayores de dependencias o nuevas versiones de kernel/distribución):**

    ```bash
    sudo apt full-upgrade # Revisa con cuidado los cambios y confirma con 'y'
    ```

---

### Laboratorio 2.3: Monitoreo y Análisis Básico de Logs del Sistema con `journalctl`

#### Objetivos del Laboratorio:

* Comprender la importancia de los logs del sistema para el diagnóstico.
* Utilizar `journalctl` para visualizar, filtrar y buscar información relevante en los logs.
* Monitorear logs en tiempo real para observar eventos mientras ocurren.

#### Solución (Pasos):

1.  **Visualiza los logs recientes del sistema:**
    * Muestra todos los logs desde el último arranque (o persistentes si están configurados para ello).

    ```bash
    journalctl
    ```

    * Muestra los logs más recientes primero (desde el final del registro).

    ```bash
    journalctl -e
    ```

    * Muestra los logs en orden inverso cronológico (los más recientes arriba).

    ```bash
    journalctl -r
    ```

2.  **Filtra logs por unidad (servicio) o por ejecutable:**
    * Muestra logs solo del servicio SSH.

    ```bash
    journalctl -u ssh.service
    ```

    * Muestra logs solo del ejecutable `sudo`.

    ```bash
    journalctl /usr/bin/sudo
    ```

3.  **Filtra logs por prioridad (gravedad del mensaje):**
    * Muestra solo mensajes de error.

    ```bash
    journalctl -p err
    ```

    * Muestra mensajes de advertencia (`warning`) y de niveles más graves (`err`, `crit`, etc.).

    ```bash
    journalctl -p warning
    ```

    (Notas: Las prioridades van de 0 a 7, siendo 0 `emerg` (emergencia) el más grave y 7 `debug` el menos grave.)

4.  **Filtra logs por período de tiempo:**
    * Muestra logs desde el inicio del día de hoy.

    ```bash
    journalctl --since "today"
    ```

    * Muestra logs de la última hora.

    ```bash
    journalctl --since "1 hour ago"
    ```

    * Muestra logs entre dos fechas y horas específicas:

    ```bash
    journalctl --since "YYYY-MM-DD HH:MM:SS" --until "YYYY-MM-DD HH:MM:SS"
    # Ejemplo: journalctl --since "2025-05-21 10:00:00" --until "2025-05-21 10:30:00"
    ```

5.  **Sigue los logs en tiempo real (monitoreo en vivo):**

    ```bash
    journalctl -f # Presiona Ctrl+C para salir
    ```

    * Mientras `journalctl -f` está corriendo en una terminal, en otra terminal, realiza alguna acción que genere logs:
        * Intenta iniciar sesión en SSH (incluso con credenciales incorrectas para generar un error).
        * Reinicia un servicio (ej. `sudo systemctl restart cron`).
        * Esto te permitirá ver cómo los nuevos mensajes de log aparecen instantáneamente.

    * También puedes seguir los logs de un servicio específico en tiempo real:

    ```bash
    journalctl -u ssh.service -f
    ```

6.  **Revisa logs de arranques anteriores (si el journal está configurado para ser persistente):**

    ```bash
    journalctl --list-boots # Lista los arranques del sistema con sus IDs y fechas
    journalctl -b -1 # Muestra los logs del arranque anterior al actual
    journalctl -b 0 # Muestra los logs del arranque actual
    ```


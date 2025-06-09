# Práctica 7.1. Troubleshooting básico: Cuellos de botella (CPU, memoria, I/O)

## Objetivo de la práctica:

- Simular y diagnosticar cuellos de botella de CPU, memoria y I/O de disco utilizando `top`, `htop`, `stress-ng`, `iostat` y `iotop`.

### Preparación del entorno

Antes de comenzar, asegúrate de que tu máquina Ubuntu tenga acceso a Internet y los paquetes necesarios estén instalados:

### Actualizar el sistema (si no se ha hecho anteriormente)

    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

### Instalar herramientas adicionales

    ```bash
    sudo apt install htop sysstat iotop ncdu -y
    ```

- **htop:** Alternativa a `top` con más funciones interactivas.
- **sysstat:** Proporciona `iostat` para estadísticas de I/O de disco.
- **iotop:** Monitoriza la I/O de disco por proceso.
- **ncdu:** Herramienta interactiva para analizar el uso de disco.

- **Herramientas a utilizar:** `top`, `htop`, `stress-ng`, `iostat`, `iotop`, `free`.


### Preparación de la práctica 7.1.

### Instalar stress-ng 
Esta herramienta nos permitirá generar cargas artificiales en CPU, memoria y disco.

    ```bash
    sudo apt install stress-ng -y
    ```

### Instrucciones

### Tarea 1. Abrir una terminal y ejecutar htop

    ```bash
    htop
    ```

Observa el uso de CPU (líneas superiores) y el promedio de carga (Load average). Deberían estar bajos (cercanos a 0 o al número de núcleos si no hay actividad).


## Tarea 2. Abrir una segunda terminal y generar carga de CPU 
Vamos a estresar 2 núcleos de CPU durante 60 segundos. Si tienes 4 núcleos, esto debería poner la CPU al 50%. Ajusta 2 al número de núcleos que desees estresar.

    ```bash
    stress-ng --cpu 2 --timeout 60s
    ```

### Tarea 3. Volver a la primera terminal (donde se ejecuta htop)

Paso 1. **Observa el uso de CPU:** Deberías ver uno o más núcleos de CPU subiendo al 100% de uso, y el promedio de carga comenzando a subir significativamente.

Paso 2. **Identifica los procesos culpables:** En la lista de procesos, busca `stress-ng` con un alto porcentaje en la columna `%CPU`.

Paso 3. **Esperar a que stress-ng termine (o cancelarlo con Ctrl+C):** Observa cómo el uso de CPU y el promedio de carga bajan de nuevo a la normalidad en `htop`.

### Tarea 4. En una terminal, ejecutar free -h

    ```bash
    free -h
    ```

Toma nota de la memoria "used", "free" y "available", así como el uso de "Swap".

### Tarea 5. Abrir una segunda terminal y generar carga de memoria

Vamos a asignar 1GB de memoria virtual. Ajusta `1G` si tienes poca RAM o quieres estresar más/menos.

    ```bash
    stress-ng --vm 1 --vm-bytes 1G --timeout 60s
    ```

### Tarea 6. Volver a la primera terminal (donde se ejecuta htop) y observar

Paso 1. **Uso de memoria:** La barra de memoria (`MiB Mem`) debería llenarse. Si el sistema tiene poca RAM, verás que el Swap comienza a usarse (la barra `MiB Swap` se llenará).

Paso 2. **Procesos culpables:** Busca `stress-ng` con un alto porcentaje en la columna `%MEM`.

Paso 3 Mientras stress-ng está activo, en una TERCERA terminal, ejecuta free -h de nuevo.

    ```bash
    free -h
    ```

Paso 4. Compara los valores con los iniciales. Deberías ver un aumento en la memoria "used" y potencialmente en el "Swap".

Paso 5. **Espera a que stress-ng termine (o cancelalo):** Observa cómo la memoria se libera en `htop` y `free -h`.

### Tarea 7. Diagnóstico de cuello de botella en I/O de disco

Paso 1. En una terminal, ejecuta iostat -x 1.

    ```bash
    iostat -x 1
    ```

Paso 2. Observa los valores de `%util` y `avgqu-sz` para tus discos (ej. `sda`). Deberían estar bajos. También observa `%iowait` en la sección `avg-cpu`.

### Tarea 8. Abrir una segunda terminal y generar carga de I/O de disco

Vamos a crear y escribir en un archivo grande en `/tmp`.

    ```bash
    stress-ng --io 1 --hdd 1 --hdd-bytes 1G --timeout 60s --temp-path /tmp
    ```

    * `--io 1`: Genera carga de I/O.
    * `--hdd 1`: Genera carga de I/O de disco.
    * `--hdd-bytes 1G`: Opera con 1GB de datos.
    * `--temp-path /tmp`: Utiliza el directorio `/tmp` para los archivos temporales.

### Tarea 9. Volver a la primera terminal (donde se ejecuta iostat -x 1)

Paso 1. **Observa %util y avgqu-sz:** Deberían subir significativamente (cercano al 100% y un valor alto respectivamente) para tu disco.

Paso 2. **Observa %iowait en avg-cpu:** Este valor indica el tiempo que la CPU espera por operaciones de I/O. Debería aumentar.

### Tarea 10. Abrir una tercera terminal y ejecutar iotop (puede requerir sudo)

    ```bash
    sudo iotop
    ```

Paso 1. Identifica los procesos que están generando la mayor cantidad de lecturas (READ/s) y escrituras (WRITE/s). Deberías ver `stress-ng` en la parte superior.

Paso 2. Esperar a que stress-ng termine (o cancelarlo): Observa cómo los valores de I/O de disco vuelven a la normalidad.

---

# Práctica 7.2. Troubleshooting básico: Espacio crítico (du, df, etc)

## Objetivo de la práctica: 

- Identificar y gestionar situaciones de espacio en disco crítico utilizando `df`, `du`, `ncdu` y realizando acciones de limpieza.

**Herramientas a utilizar:** `df`, `du`, `ncdu`, `dd`, `rm`, `apt`.

### Instrucciones

### Tarea 1. Diagnóstico de espacio en disco crítico

Paso 1. Verifica el espacio actual.

    ```bash
    df -h
    ```

Paso 2. Toma nota del porcentaje de uso en tu partición raíz (`/`). Debería ser relativamente bajo.

Paso 3. **Simular llenado de disco:** Vamos a crear un archivo grande en el directorio `/tmp` (asegúrate de que tu partición raíz tenga suficiente espacio para esto, ajusta `count` si es necesario). ¡Cuidado con llenar completamente tu partición raíz en un sistema de producción!

    ```bash
    sudo dd if=/dev/zero of=/tmp/large_file.bin bs=1G count=3
    ```

    Esto creará un archivo de 3GB lleno de ceros.

### Tarea 2. Verificar el espacio de nuevo

    ```bash
    df -h
    ```

Deberías ver que el porcentaje de uso de tu partición raíz ha aumentado considerablemente.

### Tarea 3. Identificar qué está ocupando espacio con ncdu

    ```bash
    sudo ncdu /
    ```

Paso 1. Usa las flechas para navegar por los directorios.
Paso 2. Identifica `/tmp` y verás el `large_file.bin` ocupando gran parte del espacio.
Paso 3. Presiona `q` para salir de `ncdu`.

### Solución de espacio crítico

### Tarea 1. Eliminar archivos grandes identificados

    ```bash
    sudo rm /tmp/large_file.bin
    ```

### Tarea 2. Verificar la liberación de espacio

    ```bash
    df -h
    ```

El porcentaje de uso de tu partición raíz debería haber vuelto a la normalidad.

**Limpiar la caché de paquetes APT:** Con el tiempo, el gestor de paquetes guarda los `.deb` descargados en `/var/cache/apt/archives`.

    ```bash
    sudo du -sh /var/cache/apt/archives/
    sudo apt clean
    sudo du -sh /var/cache/apt/archives/
    ```

    Observa cómo el tamaño se reduce.

### Tarea 3. Eliminar dependencias innecesarias

    ```bash
    sudo apt autoremove
    ```

Esto puede liberar espacio si hay paquetes instalados como dependencias que ya no son requeridos por ningún software.

**Revisar y gestionar logs antiguos (simulación):** Aunque `logrotate` se encarga de esto, simulemos una revisión manual.

    ```bash
    sudo du -sh /var/log/
    # Explora los subdirectorios grandes con ncdu
    sudo ncdu /var/log/
    # Si encuentras logs muy antiguos o no rotados, puedes eliminarlos (¡con MUCHA precaución!)
    # Ejemplo (NO EJECUTAR EN PRODUCCIÓN SIN SABER QUÉ BORRAS):
    sudo rm /var/log/my_old_app.log.12
    ```

---

# Práctica 7.3. Análisis de logs con filtros y patrones

## Objetivo de la práctica:
- Practicar la extracción de información útil de archivos de logs utilizando `grep`, `awk`, `tail` y `pipes`.

**Herramientas a utilizar:** `grep`, `awk`, `tail`, `sort`, `uniq`, `head`, `journalctl`.

### Preparación del laboratorio 

- **Generar actividad en el log de autenticación:** Vamos a intentar iniciar sesión con un usuario inexistente varias veces para generar entradas "Failed password" en `/var/log/auth.log`.

    ```bash
    ssh nonexistent_user@localhost
    # Ingresa una contraseña cualquiera, se mostrará "Permission denied"
    # Repite este comando 3-5 veces.
    ```

Luego, inicia sesión con tu propio usuario o `exit` de la máquina si estás usando SSH.

### Instrucciones

### Filtrado básico con grep

### Tarea 1. Buscar intentos de inicio de sesión fallidos

    ```bash
    grep "Failed password" /var/log/auth.log
    ```

Deberías ver las líneas de los intentos fallidos que generaste.

### Tarea 2. Buscar errores en los logs de Apache/Nginx (si están instalados)
- Si tienes Apache o Nginx instalados, puedes revisar sus logs de error:

    ```bash
    grep -i "error" /var/log/apache2/error.log  # Para Apache
    # O
    grep -i "error" /var/log/nginx/error.log    # Para Nginx
    ```

### Tarea 3. Mostrar líneas de log relacionadas con la red

    ```bash
    grep -i "network" /var/log/syslog | tail -n 10
    ```

    Esto mostrará las últimas 10 líneas de syslog que contienen la palabra "network" (ignorando mayúsculas/minúsculas).

### Tarea 4. Análisis avanzado con pipes (|)

Paso 1. **Identificar las IPs que más fallaron en el login:** Este es un ejemplo clásico de análisis de seguridad.

    ```bash
    grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head -n 5
    ```

    * `grep "Failed password"`: Filtra las líneas relevantes.
    * `awk '{print $11}'`: Extrae el 11º campo (la IP de origen).
    * `sort`: Ordena las IPs.
    * `uniq -c`: Cuenta las ocurrencias de cada IP única.
    * `sort -nr`: Ordena numéricamente en orden descendente.
    * `head -n 5`: Muestra las 5 IPs con más intentos fallidos.

Paso 2. **Contar ocurrencias de un patrón en logs comprimidos (zgrep):** Busca cuántas veces se menciona "error" en todos los logs de syslog (incluidos los comprimidos).

    ```bash
    zgrep -c "error" /var/log/syslog*
    ```

Paso 3. **Monitoreo en tiempo real de nuevas conexiones SSH:** Abre una terminal y ejecuta:

    ```bash
    tail -f /var/log/auth.log | grep --line-buffered "Accepted password"
    ```

- `--line-buffered`: Asegura que `grep` procese la entrada línea por línea, lo cual es importante con `tail -f`.

Paso 4. En una segunda terminal, intenta iniciar sesión en tu máquina Ubuntu vía SSH (ej. `ssh your_user@localhost`). Verás el mensaje "Accepted password" aparecer en la primera terminal.

#### Uso de journalctl para Análisis de Logs

### Tarea 1. Ver los últimos 200 mensajes del journal

    ```bash
    sudo journalctl -n 200 | less
    ```

### Tarea 2. Ver logs de un servicio específico (ej. ssh.service)

    ```bash
    sudo journalctl -u ssh.service | less
    ```

### Tarea 3. Filtrar logs del kernel por prioridad de error

    ```bash
    sudo journalctl -k -p err | less
    ```

Esto mostrará solo los mensajes del kernel con nivel de error o superior.

### Tarea 4. Ver logs de un arranque anterior (si el sistema se reinició)

    ```bash
    sudo journalctl -b -1 | less
    ```

Esto muestra los logs del arranque anterior. Si no hay un arranque anterior (ej. primera vez que el sistema se reinicia), puede que no muestre nada.

---

# Práctica 7.4. Revisión de errores del sistema (dmesg, kernel logs, etc.)

## Objetivo de la práctica: 

- Aprender a usar `dmesg` y `journalctl` para diagnosticar problemas a nivel de kernel y hardware.

**Herramientas a utilizar:** `dmesg`, `journalctl`.

### Explorando dmesg

### Tarea 1. Ver el buffer completo de dmesg con marcas de tiempo legibles

    ```bash
    sudo dmesg -T | less
    ```

Navega por la salida. Busca líneas que mencionen la detección de CPU, memoria, discos, interfaces de red. Esto te da una idea del hardware detectado al arranque.

### Tarea 2. Filtrar mensajes de dmesg por nivel de error y advertencia

    ```bash
    sudo dmesg -T -l err,warn | less
    ```

Esta es la forma más rápida de ver problemas potenciales. Si el sistema está sano, es posible que no veas muchos (o ningún) mensaje aquí.

### Tarea 3. Buscar errores específicos de hardware en dmesg
Vamos a simular la búsqueda de errores de disco (aunque es poco probable que tu VM tenga errores reales a menos que la host-machine los tenga).

    ```bash
    sudo dmesg -T | grep -Ei "disk|ata|scsi|io error" | less
    ```

Si tuvieras un problema de I/O de disco, aquí lo verías.

### Tarea 4. Monitorear dmesg en tiempo real

Paso 1. Abre una terminal y ejecuta.

    ```bash
    sudo dmesg -w
    ```

Paso 2. En una segunda terminal, puedes probar a conectar una unidad USB virtual (si tu hypervisor lo permite) o realizar alguna acción de bajo nivel para ver si aparecen mensajes.

### Usando journalctl para logs del Kernel

### Tarea 1. Ver solo los mensajes del kernel en journalctl (equivalente a dmesg pero persistente)

    ```bash
    sudo journalctl -k | less
    ```

Compara la salida con `dmesg`. Debería ser muy similar.

### Tarea 2. Filtrar mensajes del kernel por nivel de prioridad crítica o de error

    ```bash
    sudo journalctl -k -p crit,err | less
    ```

Si el sistema ha experimentado "kernel panics" o errores graves, aparecerán aquí.

### Tarea 3. Ver mensajes del kernel desde un período de tiempo específico

    ```bash
    sudo journalctl -k --since "2 hours ago" | less
    ```

Esto es útil si sabes cuándo ocurrió un problema.

### Tarea 4. Ver mensajes de un módulo del kernel específico (ej. ext4 para el sistema de archivos)

    ```bash
    sudo journalctl -k | grep "ext4" | less
    ```

Esto te mostrará mensajes relacionados con el sistema de archivos Ext4, útil si tienes problemas de montaje o corrupción de datos.

### Tarea 5. Limpiar el buffer de dmesg para una nueva depuración (opcional)

    ```bash
    sudo dmesg -c
    ```

> **¡Cuidado!** Esto borra el buffer actual. Solo hazlo si quieres empezar a depurar un problema nuevo y quieres que `dmesg` solo muestre los mensajes desde ese momento.

---

¡Felicidades! Has completado los laboratorios básicos de troubleshooting y operación en producción en Linux Ubuntu. Estos ejercicios te dan una base sólida para diagnosticar y resolver problemas comunes en un entorno de servidor. Recuerda que la práctica constante y la combinación de estas herramientas te harán más eficiente en la administración de sistemas.

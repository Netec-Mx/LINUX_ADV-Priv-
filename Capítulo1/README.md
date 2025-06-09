# Práctica 1.1. Arquitectura de Linux (repaso y observación)

## Objetivo de la práctica: 
- Comprender de forma práctica la separación entre el espacio de usuario y el kernel, y cómo interactúan las aplicaciones.

### Configuración inicial (antes de empezar cualquier laboratorio):
 -Asegúrate de tener una máquina virtual o un sistema Ubuntu real funcionando. Para la mayoría de estos laboratorios, necesitarás acceso de superusuario (sudo).
- Abre una terminal: Puedes hacerlo pulsando `Ctrl + Alt + T` o buscando "Terminal" en el menú de aplicaciones.

### Instrucciones:

### Tarea 1. Observar el Kernel:

Paso 1. Abre una terminal.
Paso 2. Ejecuta el comando `uname -a`. Esto te mostrará información detallada sobre el kernel que estás utilizando (nombre, versión, fecha de compilación, etc.)
        * `uname -r` te dará solo la versión del kernel.
        
> **Concepto:** Esta es la "base" de tu sistema operativo. Todas las aplicaciones en el espacio de usuario interactúan con el kernel para acceder al hardware, gestionar procesos, etc.

### Tarea 2. Observar procesos en el espacio de usuario:

Paso 1. Ejecuta el comando `ps aux`. Esto listará todos los procesos que se están ejecutando actualmente en tu sistema. Observa la gran cantidad de procesos, la mayoría de los cuales son aplicaciones o servicios en el espacio de usuario.

> **Concepto:** Cada una de estas entradas representa un programa o servicio que se ejecuta en el espacio de usuario, utilizando los recursos y servicios proporcionados por el kernel.

* **Reflexión:** ¿Cómo crees que un programa como Firefox (espacio de usuario) interactúa con el hardware de red para cargar una página web? Piensa en el rol del kernel.

</br>

---

# Práctica 1.2. Sistema de archivos: Jerarquía FHS (Filesystem Hierarchy Standard)

## Objetivo de la práctica:
- Explorar la jerarquía estándar del sistema de archivos de Linux (FHS) y comprender la importancia de directorios clave.

### Instrucciones:

### Tarea 1. Explorar el directorio raíz (`/`)

Paso 1. Ejecuta `cd /` para ir al directorio raíz.
Paso 2. Ejecuta `ls -l` para listar su contenido.

* **Preguntas para reflexionar:**
- *¿Qué directorios observas?*
- *¿Qué tipo de información crees que se almacena en `/bin`, `/etc`, `/home`, `/var`?*
- *Investiga qué es el FHS y por qué es importante.*
  
### Tarea 2. Explorar `/home`**

Paso 1. Ejecuta `cd /home`.
Paso 2. Ejecuta `ls -l`. Verás tu directorio de usuario.
Paso 3. Ejecuta `cd tu_usuario` (reemplaza `tu_usuario` con tu nombre de usuario).
Paso 4. Ejecuta `ls -l`. Aquí es donde se guardan tus archivos personales, configuraciones, etc.


### Tarea 3. Explorar `/etc`:

Paso 1. Ejecuta `cd /etc`.
Paso 2. Ejecuta `ls -l`.

>**Concepto:** Este directorio contiene archivos de configuración del sistema. Son cruciales para el funcionamiento de Linux.

Paso 3. Ejecuta `cat /etc/passwd`. Este archivo contiene información sobre los usuarios del sistema. (No modifiques este archivo a menos que sepas exactamente lo que haces).

### Tarea 4. Explorar `/var`:

Paso 1. Ejecuta `cd /var`.
Paso 2. Ejecuta `ls -l`.

> **Concepto:** Este directorio contiene datos variables, como archivos de registro (logs), bases de datos, correos electrónicos, etc. Los logs son vitales para la resolución de problemas.

Paso 3. Ejecuta `ls -l /var/log`. Aquí encontrarás muchos archivos de registro. Puedes usar `tail -f /var/log/syslog` para ver los mensajes del sistema en tiempo real (usa `Ctrl + C` para salir).

* **Reflexión:** ¿Por qué crees que es importante tener una estructura de directorios estandarizada en Linux?

<br/>

--- 

# Práctica 1.3. Línea de comandos (CLI) - Comandos básicos

## Objetivo de la práctica: 
- Familiarizarse con los comandos esenciales de la CLI para navegar, manipular archivos y obtener ayuda.
    
### Navegación:

        * `pwd`: Muestra el directorio de trabajo actual.
        * `cd ..`: Sube un nivel en la jerarquía de directorios.
        * `cd ~` o `cd`: Te lleva a tu directorio personal (`/home/tu_usuario`).
        * `ls`: Lista el contenido del directorio actual.
        * `ls -l`: Lista el contenido en formato largo (detalles).
        * `ls -a`: Lista también archivos y directorios ocultos.
        * `ls -lh`: Lista en formato largo y con tamaño legible.
        
### Manipulación de archivos y directorios:

        * Crea un directorio de pruebas: `mkdir mi_directorio_de_pruebas`.
        * Entra en él: `cd mi_directorio_de_pruebas`.
        * Crea un archivo vacío: `touch mi_primer_archivo.txt`.
        * Edita el archivo con un editor de texto simple (nano): `nano mi_primer_archivo.txt`. Escribe algo, guarda (`Ctrl + O`, `Enter`) y sal (`Ctrl + X`).
        * Visualiza el contenido del archivo: `cat mi_primer_archivo.txt`.
        * Copia el archivo: `cp mi_primer_archivo.txt mi_copia_del_archivo.txt`.
        * Mueve el archivo: `mv mi_copia_del_archivo.txt ../otro_nombre.txt`. (Intenta moverlo un nivel arriba y cambiarle el nombre).
        * Vuelve a tu directorio de pruebas: `cd mi_directorio_de_pruebas`.
        * Borra el archivo: `rm mi_primer_archivo.txt`.
        * Intenta borrar el directorio (vacío): `rmdir mi_directorio_de_pruebas`. (Solo funciona si está vacío).
        * Crea un archivo y un directorio dentro del directorio de pruebas. Luego, borra el directorio y su contenido de forma recursiva y forzada: `rm -rf mi_directorio_de_pruebas`. (¡MUCHO CUIDADO con `rm -rf`! Puede borrar cosas importantes sin pedir confirmación).
        
### Obtener ayuda:
        * `man ls`: Muestra la página de manual del comando `ls`. Presiona `q` para salir.
        * `ls --help`: Muestra un resumen de las opciones del comando `ls`.
        * `whatis ls`: Proporciona una descripción de una línea del comando.
        
* **Reflexión:** ¿Qué ventajas tiene usar la línea de comandos sobre una interfaz gráfica para estas tareas?, ¿qué desventajas?

<br/>

---

# Práctica 1.4: Permisos de archivos y directorios

## Objetivo de la práctica: 
- Comprender el modelo de permisos de Linux (propietario, grupo, otros) y cómo modificarlos.

### Instrucciones

### Tarea 1. Observar permisos

Paso 1. Crea un archivo: `touch permisos_prueba.txt`.
Paso 2. Ejecuta `ls -l permisos_prueba.txt`.


### Análisis de la salida
- La primera columna (`-rw-r--r--`) son los permisos.
- Los siguientes campos son: número de enlaces, propietario, grupo, tamaño, fecha, nombre del archivo.
- `d` al principio indica un directorio. `-` indica un archivo.
- Los permisos se dividen en 3 bloques de 3 caracteres:
       - 1er bloque: Propietario (`u` - user)
       - 2do bloque: Grupo (`g` - group)
       - 3er bloque: Otros (`o` - others)
            - `r` (read - lectura), `w` (write - escritura), `x` (execute - ejecución).

  
### Tarea 2. Cambiar permisos (método simbólico)

Paso 1. Dale permiso de escritura a "otros": `chmod o+w permisos_prueba.txt`.
Paso 2. Verifica: `ls -l permisos_prueba.txt`.
Paso 3. Quítale permiso de ejecución a "propietario": `chmod u-x permisos_prueba.txt`. (No tendrá efecto si ya no tenía `x`).
Paso 4. Dale permiso de ejecución a "todos" (propietario, grupo, otros): `chmod a+x permisos_prueba.txt`.

### Tarea 3. Cambiar permisos (método octal)

- **Concepto:**
       - `r = 4`
       - `w = 2`
       - `x = 1`

Paso 1. Suma los valores para cada conjunto (propietario, grupo, otros).
        * Por ejemplo: `rwx` es `4+2+1 = 7`. `rw-` es `4+2+0 = 6`. `r--` es `4+0+0 = 4`.
Paso 2. Establece permisos `rw-r--r--` (644): `chmod 644 permisos_prueba.txt`.
Paso 3. Establece permisos `rwxr-xr-x` (755) para un script o un directorio: `chmod 755 mi_directorio`.
    
### Tarea 4. Cambiar propietario y grupo

Paso 1. Cambia el propietario a `root` (necesitarás `sudo`): `sudo chown root permisos_prueba.txt`.
Paso 2. Cambia el grupo a `root` (necesitarás `sudo`): `sudo chgrp root permisos_prueba.txt`.
Paso 3. Vuelve a cambiar el propietario y grupo a tu usuario: `sudo chown tu_usuario:tu_usuario permisos_prueba.txt`.

* **Reflexión:** ¿Por qué es tan importante la gestión de permisos en un sistema multiusuario como Linux? ¿Qué riesgos existen si los permisos no se configuran correctamente?

<br/>

 ---

# Práctica 1.5. Gestión de procesos

### Objetivo de la práctica: 
- Aprender a listar, monitorear y terminar procesos en Linux.

### Instrucciones

- **Listar procesos:**
       - `ps aux`: Muestra todos los procesos que se están ejecutando en el sistema, incluyendo los de otros usuarios.
       - `ps -ef`: Similar a `ps aux`, pero con un formato diferente que a veces es más legible.
       -`pstree`: Muestra los procesos en una vista de árbol, lo que ayuda a ver las relaciones padre-hijo.

### Tarea 1. Monitorear procesos en tiempo real
        * `top`: Muestra una vista dinámica y en tiempo real de los procesos que consumen más CPU y memoria. Presiona `q` para salir.
        * `htop` (si no está instalado, instálalo con `sudo apt update && sudo apt install htop`): Una alternativa más interactiva y visual a `top`. Presiona `F10` o `q` para salir.

### Tarea 2. Terminar procesos

Identificar un proceso para terminar 
Paso 1. Abre una segunda terminal. En ella, ejecuta un comando que se quede "colgado" o que puedas identificar fácilmente, por ejemplo: `sleep 600` (esto hará que el proceso espere 600 segundos).

Paso 2. En la primera terminal, usa `ps aux | grep sleep` para encontrar el PID (ID de proceso) del comando `sleep`.
        * **Terminar suavemente:** `kill PID` (reemplaza `PID` con el número de proceso). Esto envía la señal `SIGTERM` (15), pidiendo al proceso que termine limpiamente. El proceso `sleep` debería terminar.
        * **Terminar forzosamente (último recurso):** Si un proceso no responde a `kill`, puedes forzar su terminación con `kill -9 PID` (envía la señal `SIGKILL` - 9). Esto no permite que el proceso realice ninguna limpieza, así que úsalo con precaución.
        * **Terminar por nombre:** `pkill nombre_proceso` o `killall nombre_proceso`. Por ejemplo, `pkill firefox`. (¡Cuidado con estos, pueden cerrar múltiples instancias!)
* **Reflexión:** ¿Cuándo usarías `kill` vs `kill -9`? ¿Por qué es importante monitorear los procesos en un servidor?

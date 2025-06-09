# Linux para Administración, Redes y Troubleshooting

**Plataforma de Laboratorios**

Bienvenido a la **Plataforma de Laboratorios** del curso **Linux para Administración, Redes y Troubleshooting**. Aquí podrás explorar diferentes tecnologías a través de prácticas guiadas. ¡Desarrolla tus habilidades y lleva tus conocimientos al siguiente nivel!

---

## 🌟 **Lista de Laboratorios**

Cada uno de estos laboratorios está diseñado para ofrecerte una experiencia práctica. Haz clic en los enlaces para comenzar.

---
 
## Índice:

## [Fundamentos del sistema Linux en Ubuntu](./Capítulo1/README.md)

 **- Práctica 1.1. Arquitectura de Linux (repaso y observación).**
   - **Descripción**:Comprender de forma práctica la separación entre el espacio de usuario y el kernel, y cómo interactúan las aplicaciones.
   - ⏱️ **Duración estimada**: 11 minutos.
  
 **- Práctica 1.2. Sistema de archivos: Jerarquía FHS (Filesystem Hierarchy Standard).**
   - **Descripción**: Explorar la jerarquía estándar del sistema de archivos de Linux (FHS) y comprender la importancia de directorios clave.
   - ⏱️ **Duración estimada**: 15 minutos.

 **- Práctica 1.3. Línea de comandos (CLI) - Comandos básicos.**
   - **Descripción**: Familiarizarse con los comandos esenciales de la CLI para navegar, manipular archivos y obtener ayuda.
   - ⏱️ **Duración estimada**: 20 minutos.

 **- Práctica 1.4. Permisos de archivos y directorios.**
   - **Descripción**: Comprender el modelo de permisos de Linux (propietario, grupo, otros) y cómo modificarlos.
   - ⏱️ **Duración estimada**: 25 minutos.

 **- Práctica 1.5. Gestión de procesos.**
   - **Descripción**: Aprender a listar, monitorear y terminar procesos en Linux.
   - ⏱️ **Duración estimada**: 25 minutos.

</br>

## [Gestión de procesos, systemd y software](./Capítulo2/README.md)

 **- Práctica 2.1. Gestión y monitoreo de proceos y servicios (systemd).**
   - **Descripción**: Identificar y listar procesos en ejecución en el sistema, monitorear el consumo de recursos (CPU, memoria) de los procesos en tiempo real y comprender cómo habilitar y deshabilitar servicios para el inicio automático.
   - ⏱️ **Duración estimada**: 30 minutos.

 **- Práctica 2.2. Instalación, actualización y mantenimiento de paquetes.**
   - **Descripción**: Dominar los comandos básicos del gestor de paquetes apt para actualizar el sistema, instalar y eliminar paquetes de software de forma segura y realizar tareas de limpieza del sistema.
   - ⏱️ **Duración estimada**: 25 minutos.

 **- Práctica 2.3. Monitoreo y análisis básico de logs del sistema con journalctl.**
   - **Descripción**: Comprender la importancia de los logs del sistema para el diagnóstico, utilizar journalctl para visualizar, filtrar y buscar información relevante en los logs.
   - ⏱️ **Duración estimada**: 25 minutos.

<br/>

## [Redes en Linux](./Capítulo3/README.md)

 **- Práctica 3.1. Comandos clave para diagnóstico y visibilidad de red.**
   - **Descripción**: Utilizar ip para inspeccionar la configuración de interfaces de red, direcciones IP y tablas de enrutamiento, así como emplear ss y netstat para listar conexiones de red activas y puertos en escucha.
   - ⏱️ **Duración estimada**: 15 minutos.
  
 **- Práctica 3.2. Configuración básica de interfaces de red (IP estática y DHCP).**
   - **Descripción**: Identificar la configuración de red actual (DHCP vs. Estática) y realizar cambios persistentes en la configuración de red utilizando Netplan en Ubuntu.
   - ⏱️ **Duración estimada**: 20 minutos.

 **- Práctica 3.3. Implementación de subinterfaces (VLANs).**
   - **Descripción**: Configurar una subinterfaz VLAN en una interfaz física existente y asignar una dirección IP y verificar la configuración de la subinterfaz.
   - ⏱️ **Duración estimada**: 25 minutos.


 **- Práctica 3.4. Configuración básica del firewall UFW.**
   - **Descripción**: Configurar una subinterfaz VLAN en una interfaz física existente y asignar una dirección IP y verificar la configuración de la subinterfaz.
   - ⏱️ **Duración estimada**: 20 minutos.


 **- Práctica 3.5. Análisis básico de tráfico de red con tcpdump.**
   - **Descripción**: Configurar una subinterfaz VLAN en una interfaz física existente y asignar una dirección IP y verificar la configuración de la subinterfaz.
   - ⏱️ **Duración estimada**: 20 minutos.

<br/>

## [Servicios de red](./Capítulo4/README.md)

 **- Práctica 4.1. Configuración de servicios (SSH y NTP) y gestión de puertos con UFW.**
   - **Descripción**: Instalar y configurar el servicio SSH en servidor-Ubuntu, nstalar y configurar el servicio NTP (systemd-timesyncd) y configurar reglas de firewall UFW para SSH y verificar puertos. 
   - ⏱️ **Duración estimada**: 35 minutos.
  

 **- Práctica 4.2. Configuración de NTP (systemd-timesyncd).**
   - **Descripción**: Instalar y configurar el servicio NTP.
   - ⏱️ **Duración estimada**: 30 minutos.


 **- Práctica 4.3. Gestión avanzada de puertos sockets (ss, lsof).**
   - **Descripción**: Identificar puertos en estado de escucha y sus procesos asociados usando ss, usar lsof para encontrar qué proceso está usando un puerto específico.  
   - ⏱️ **Duración estimada**: 30 minutos.

<br/>

## [Automatización](./Capítulo5/README.md)

 **- Práctica 5.1. Shell Scripting con Bash.**
   - **Descripción**: Crear un script Bash para automatizar la gestión de usuarios y directorios.
   - ⏱️ **Duración estimada**: 45 minutos.

 **- Práctica 5.2. Automatización de tareas (at, cron, etc.)**
   - **Descripción**: Programar tareas de una sola vez y recurrentes usando at y cron.  
   - ⏱️ **Duración estimada**: 35 minutos.

 **- Práctica 5.3. Auditoría de logs (awk, grep, etc.)**
   - **Descripción**: Utilizar grep y awk para extraer información relevante de archivos de log sistema.
   - ⏱️ **Duración estimada**: 50 minutos.

 **- Práctica 5.4. Uso de jq (JSON).**
   - **Descripción**: Procesar y filtrar archivos de log en formato JSON usando jq. 
   - ⏱️ **Duración estimada**: 50 minutos.

<br/>

## [Seguridad básica en Linux](./Capítulo6/README.md)

 **- Práctica 6.1. Permiso y sudo.**
   - **Descripción**: Comprender cómo los permisos afectan el acceso a archivos y practicar el uso seguro de sudo.
   - ⏱️ **Duración estimada**: 35 minutos.

**- Práctica 6.2. Auditoría básica.**
   - **Descripción**: Aprender a usar last, who, w, ps aux y a revisar logs para una auditoría básica.
   - ⏱️ **Duración estimada**: 20 minutos.

**- Práctica 6.3. Revisar procesos de ejecucuón.**
   - **Descripción**:Listar y explorar todos los procesos del sistema.
   - ⏱️ **Duración estimada**: 20 minutos.

**- Práctica 6.4. Hardering básico del sistema.**
   - **Descripción**: Aplicar medidas básicas de endurecimiento a SSH y al kernel.  
   - ⏱️ **Duración estimada**: 30 minutos.

<br/>

## [Troubleshooting y operación en producción](./Capítulo7/README.md)

 **- Práctica 7.1. Troubleshooting básico: Cuellos de botella (CPU, Memoria, I/O).**
   - **Descripción**: Simular y diagnosticar cuellos de botella de CPU, memoria y I/O de disco utilizando top, htop, stress-ng, iostat y iotop.
   - ⏱️ **Duración estimada**: 20 minutos.

 **- Práctica 7.2. Troubleshooting básic: Espacio crítico (du, df, etc.)**
   - **Descripción**: Identificar y gestionar situaciones de espacio en disco crítico utilizando df, du, ncdu y realizando acciones de limpieza.
   - ⏱️ **Duración estimada**: 20 minutos.

 **- Práctica 7.3. Análisis de logs con filtros y patrones.**
   - **Descripción**: Practicar la extracción de información útil de archivos de logs utilizando grep, awk, tail y pipes.
   - ⏱️ **Duración estimada**: 25 minutos.

 **- Práctica 7.4. Revisión de errores del sistema (dmesg, kernel logs, etc.)**
   - **Descripción**:  Aprender a usar dmesg y journalctl para diagnosticar problemas a nivel de kernel y hardware.
   - ⏱️ **Duración estimada**: 25 minutos.

<br/>

## [Buenas prácticas y mantenimiento en producción](./Capítulo8/README.md)

 **- Práctica 8.1. Backups de configuración y scripts (rsync, tar, scp).**
   - **Descripción**: Practicar la creación de backups de configuraciones y scripts usando las utilerías rsync, tar y scp. 
   - ⏱️ **Duración estimada**: 45 minutos.

**- Práctica 8.2. Versionado con Git para scripts y configuraciones.**
   - **Descripción**: Establecer un flujo de trabajo básico para versionar configuraciones y scripts usando Git, con un repositorio remoto. 
   - ⏱️ **Duración estimada**: 45 minutos.
 
**- Práctica 8.3. Control de cambios en el sistema.**
   - **Descripción**: Aprender a rastrear y monitorear los cambios realizados en el sistema, más allá de Git. 
   - ⏱️ **Duración estimada**: 25 minutos.

**- Práctica 8.4. Validaciones antes de reinicios o actualizaciones.**
   - **Descripción**: Realizar una serie de verificaciones críticas antes de un reinicio o una actualización importante para asegurar la estabilidad del sistema. 
   - ⏱️ **Duración estimada**: 30 minutos.

**- Práctica 8.5. Planificación de mantenimiento sin impacto.**
   - **Descripción**: Simular y entender cómo se logra el mantenimiento sin impacto en un entorno de producción, centrándose en el concepto de redundancia y despliegue gradual. 
   - ⏱️ **Duración estimada**: 35 minutos.

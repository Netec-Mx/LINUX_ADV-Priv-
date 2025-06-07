# Práctica 3.1. Comandos clave para diagnóstico y visibilidad de red

## Objetivos de la práctica:

- Utilizar `ip` para inspeccionar la configuración de interfaces de red, direcciones IP y tablas de enrutamiento. 
- Emplear `ss` y `netstat` para listar conexiones de red activas y puertos en escucha. 
- Diagnosticar conectividad básica con `ping` y `traceroute`. 
- Consultar la caché ARP y realizar consultas DNS con `dig`.

### Instrucciones

### Tarea 1.  Inspección de interfaces y direcciones IP (`ip addr show`):

Paso 1. Abre una terminal.
Paso 2. Muestra todas las interfaces de red y sus direcciones IP configuradas.
Paso 3. Identifica tu interfaz principal (ej., `eth0`, `enp0s3`, `ens33`) y su dirección IP. 
    * `ip addr show`
    * `# O la forma más concisa: ip a`


### Tarea 2. Visualización de la tabla de enrutamiento (`ip route show`):

Muestra la tabla de enrutamiento del kernel, incluyendo la puerta de enlace predeterminada (`default via`). 
    - `ip route show`


### Tarea 3. Monitoreo de conexiones y puertos (`ss`, `netstat`):**

Paso 1. Lista todos los puertos de escucha y conexiones activas para TCP y UDP (se prefiere `ss` por ser más moderno y eficiente):
        * `ss -tuln` # TCP y UDP, en escucha, numérico
        * `ss -ant` # TCP, todas las conexiones, numérico

Paso 2. Filtra para ver si un servicio es específico (ej., SSH en puerto 22) está escuchando:
        * `ss -tuln | grep :22` [cite: 9]
        * `(Alternativa más antigua: netstat -tuln)`

### Tarea 4.  Diagnóstico de conectividad (`ping`, `traceroute`):

Paso 1. Haz un ping a tu puerta de enlace para verificar la conectividad local (reemplaza `<DIRECCION_IP_DE_TU_GATEWAY>` con la IP obtenida en el paso 1):
        * `ping -c 4 <DIRECCION_IP_DE_TU_GATEWAY>`

Paso 2. Haz un ping a un host externo (ej., Google DNS) para verificar la conectividad a Internet:
        * `ping -c 4 8.8.8.8`

Paso 3. Rastrea la ruta a un host externo para ver los saltos (instala `traceroute` si no lo tienes: `sudo apt install traceroute`):
        * `traceroute 8.8.8.8`

### Tarea 5. Consulta de la Caccé ARP (`ip neigh show`):
    
Paso 1. Muestra las entradas en la caché ARP (mapeo de direcciones IP a MAC).
    * `ip neigh show`

### Tarea 6. Resolución de nombres DNS (`dig`):

Paso 1. Resuelve un nombre de dominio (ej., `google.com`) a su dirección IP usando los DNS configurados en tu sistema:
        * `dig google.com`
Paso 2. Realiza una consulta a un servidor DNS específico (ej., `1.1.1.1` - Cloudflare DNS):
        * `dig @1.1.1.1 example.com`

---

# Práctica 3.2. Configuración básica de interfaces de red (IP Estática y DHCP)

## Objetivos de la práctica:

- Identificar la configuración de red actual (DHCP vs. Estática).
- Configurar una dirección IP estática en una interfaz de red de forma temporal (en memoria).
- Restaurar la configuración de DHCP temporalmente. 
- Realizar cambios persistentes en la configuración de red utilizando Netplan en Ubuntu. 

### Prerrequisitos:

- Una máquina virtual (VM) de Ubuntu. Esto es altamente recomendable para evitar problemas de conectividad en tu máquina principal. 
- Acceso a la consola de la VM o a una conexión SSH alternativa estable (ej. a través de una segunda interfaz de red o un cliente SSH que no se vea afectado por los cambios). 
  
### Instrucciones:

### Tarea 1. Identifica tu interfaz de red principal y su configuración actual:
    
    * `ip a show` # Identifica el nombre de la interfaz (ej. 'enp0s3')

Paso 1. Anota la IP actual, la máscara de red y la puerta de enlace (si aplica). 
Paso 2. Revisa tu archivo Netplan para entender la configuración persistente (normalmente en `/etc/netplan/`): 
        * `ls /etc/netplan/`
        * `cat /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml` # Ej. `00-installer-config.yaml` 


### Tarea 2. Configura una IP estática temporalmente (en RAM, no persistente):

> ¡ADVERTENCIA!** Esto puede cortar tu conexión actual si no estás en la consola.

Paso 1. Reemplaza `<NOMBRE_INTERFAZ>`, `<IP_ESTATICA_EJEMPLO>`, `<PREFIJO_EJEMPLO>`, `<IP_GATEWAY_EJEMPLO>` con valores adecuados para tu red local. 

```
    # Baja la interfaz
    sudo ip link set <NOMBRE_INTERFAZ> down
    # Elimina cualquier IP existente (opcional pero recomendado si cambias de tipo)
    sudo ip addr flush dev <NOMBRE_INTERFAZ>
    # Asigna la nueva IP estática
    sudo ip addr add <IP_ESTATICA_EJEMPLO>/<PREFIJO_EJEMPLO> dev <NOMBRE_INTERFAZ>` # Ej: `192.168.1.200/24
    # Levanta la interfaz
    sudo ip link set <NOMBRE_INTERFAZ> up
    # Añade la ruta por defecto (gateway)
    sudo ip route add default via <IP_GATEWAY_EJEMPLO>` # Ej: `192.168.1.1
```

Paso 2. Verifica la nueva IP y la ruta:

```
   ip a show <NOMBRE_INTERFAZ>
   ip route show
   ping -c 3 8.8.8.8` # Prueba la conectividad externa
````


### Tarea 3.  Restaurar a DHCP temporalmente:

```
    Libera la IP y solicita una nueva por DHCP:
        sudo ip addr flush dev <NOMBRE_INTERFAZ> # Elimina todas las IPs de la interfaz 
        sudo dhclient -r <NOMBRE_INTERFAZ> # Libera DHCP lease anterior (si aplicaba)
        sudo dhclient <NOMBRE_INTERFAZ> # Solicita una nueva IP DHCP
    Verifica la IP asignada por DHCP:
        ip a show <NOMBRE_INTERFAZ>
```


### Tarea 4. Configuración Persistente con Netplan (Archivos .yaml):

    Los cambios hechos con `ip` y `dhclient` son temporales y se perderán al reiniciar.
    Netplan gestiona la configuración persistente. 

Paso 1. Haz una **COPIA DE SEGURIDAD** de tu archivo `.yaml` actual:
        `sudo cp /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml.bak`

Paso 2. Edita el archivo `.yaml` para configurar una IP estática (ejemplo):

        ```yaml
        # ADAPTA ESTE EJEMPLO A TU INTERFAZ Y RED LOCAL
        network:
          version: 2
          renderer: networkd
          ethernets:
            <NOMBRE_INTERFAZ>: # Reemplaza con tu interfaz (ej., enp0s3)
              dhcp4: no # Deshabilita DHCP para IPv4
              addresses: [<IP_ESTATICA_EJEMPLO>/<PREFIJO_EJEMPLO>] # Ej: [192.168.1.201/24]
              routes:
                - to: default
                  via: <IP_GATEWAY_EJEMPLO> # Tu gateway (ej: 192.168.1.1)
              nameservers:
                addresses: [8.8.8.8, 8.8.4.4] # Servidores DNS (ej: Google DNS)
        ```
        * `sudo nano /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml`
        
Paso 1. Aplica la configuración de Netplan:
        * `sudo netplan apply`
     
Si hay errores, `netplan apply` los mostrará. Corrígelos en el `.yaml`. 

Paso 2. Verifica la nueva IP y la ruta persistentes:
        * `ip a show <NOMBRE_INTERFAZ>`
        * `ip route show`
    * Para volver a DHCP de forma persistente (edita el archivo `.yaml` de nuevo):
        ```yaml
        network:
          version: 2
          renderer: networkd
          ethernets:
            <NOMBRE_INTERFAZ>:
              dhcp4: yes # Habilita DHCP para IPv4
              # Elimina las líneas 'addresses', 'routes', 'nameservers' que añadiste para estática
        ```
        * `sudo nano /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml`
        * `sudo netplan apply`
        * `ip a show <NOMBRE_INTERFAZ>` # Confirma que obtuviste una IP DHCP

---

# Práctica 3.3. Implementación de subinterfaces (VLANs)

## Objetivos de la práctica:

- Comprender el concepto y la utilidad de las VLANs para la segmentación de red. 
- Configurar una subinterfaz VLAN en una interfaz física existente. 
- Asignar una dirección IP y verificar la configuración de la subinterfaz. 

**Prerrequisitos:**

- Máquina Virtual (VM) de Ubuntu. Este laboratorio es complejo en hardware físico sin una configuración de switch gestionado. 
- En una VM (ej. VirtualBox, VMware), la interfaz de red debe estar en modo "Adaptador Puente" o "Red Interna" y la configuración de la red virtualizada debe permitir el `tagging` de VLANs si quieres que sea realista con otros VMs/dispositivos. 
- Para este laboratorio, la crearemos a nivel de SO, asumiendo que la capa inferior (switch/virtualizador) la soportaría. 
- Paquete `vlan` instalado: `sudo apt install vlan`. 

### Instrucciones

### Tarea 1.  Identifica tu interfaz física principal:
    
    * `ip a`

Paso 1. Toma nota del nombre de tu interfaz principal (ej., `enp0s3`, `ens33`). 
Paso 2. Asegúrate de que no es `lo` (loopback). 

### Tarea 2.  Configura la subinterfaz VLAN (temporalmente):

Paso 1. Crea una subinterfaz para la VLAN ID 10. 
Paso 2. Reemplaza `<NOMBRE_INTERFAZ>` con tu interfaz física. 

```
    # Añade la subinterfaz VLAN con ID 10
    sudo ip link add link <NOMBRE_INTERFAZ> name <NOMBRE_INTERFAZ>.10 type vlan id 10
    # Levanta la subinterfaz
    sudo ip link set <NOMBRE_INTERFAZ>.10 up
    # Asigna una dirección IP a la subinterfaz (ej. en el rango 192.168.10.x/24)
    sudo ip addr add 192.168.10.1/24 dev <NOMBRE_INTERFAZ>.10

```

### Tarea 3. Verifica la nueva subinterfaz y su dirección IP:
    * `ip a show <NOMBRE_INTERFAZ>.10`

### Tarea 4. Configuración Persistente de Subinterfaz (Netplan):

Los cambios anteriores son temporales, para hacerlos persistentes, edita tu archivo Netplan.
    
Paso 1. Haz una **COPIA DE SEGURIDAD** de tu archivo `.yaml` actual:
        * `sudo cp /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml.bak`

Paso 2. Edita el archivo `.yaml` para añadir la VLAN (ejemplo):
        ```yaml
        # ADAPTA ESTE EJEMPLO A TU INTERFAZ Y RED LOCAL
        network:
          version: 2
          renderer: networkd
          ethernets:
            <NOMBRE_INTERFAZ>: # Tu interfaz física (ej. enp0s3)
              dhcp4: no # O 'yes' si tu red física principal la usa
              # addresses: [<IP_FISICA>/<PREFIJO_FISICO>] # Si tu interfaz física tiene IP estática
              # routes: ... # Rutas para la interfaz física si es necesario
          vlans:
            <NOMBRE_INTERFAZ>.10: # Nombre de la subinterfaz VLAN (ej. enp0s3.10)
              id: 10 # El ID de la VLAN
              link: <NOMBRE_INTERFAZ> # La interfaz física a la que se asocia
              dhcp4: no # La VLAN tendrá IP estática
              addresses: [192.168.10.1/24] # IP para esta VLAN
              # routes: # Opcional: Rutas específicas para esta VLAN
              #   - to: default
              #     via: 192.168.10.254 # Gateway para esta VLAN si la hubiera
              nameservers:
                addresses: [8.8.8.8] # DNS para esta VLAN
        ```
        
        * `sudo nano /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml`
   
Paso 3. Aplica la configuración de Netplan:
        * `sudo netplan apply`
   
    * Si hay errores, corrígelos en el `.yaml`.

Paso 4. Verifica la subinterfaz después de aplicar Netplan:
        * `ip a show <NOMBRE_INTERFAZ>.10`

### Tarea 5. Prueba la conectividad (requiere otro dispositivo en la misma VLAN):

Si tienes otra VM o un dispositivo físico configurado en la misma VLAN (ID 10) y segmento de red (ej., 192.168.10.x), intenta hacer ping a su dirección IP desde esta VM:
        * `ping -c 3 192.168.10.X` # Reemplaza con la IP del otro dispositivo en la VLAN

### Tarea 6.  Limpia la configuración de VLAN (después del laboratorio):

Paso 1. Edita tu archivo `.yaml` de Netplan y elimina la sección `vlans:` correspondiente a `<NOMBRE_INTERFAZ>.10`. 
    * `sudo nano /etc/netplan/<TU_ARCHIVO_NETPLAN>.yaml`

Paso 2. Aplica la configuración modificada:
        * `sudo netplan apply`

Paso 3. Verifica que la subinterfaz ya no existe:
        * `ip a show <NOMBRE_INTERFAZ>.10` # Debería decir "Does not exist"

---

# Práctica 3.4. Configuración básica del firewall UFW

## Objetivos de la práctica:

- Habilitar y deshabilitar el firewall UFW. 
- Establecer las políticas por defecto de UFW para el tráfico entrante y saliente. 
- Permitir y denegar conexiones por puerto y por servicio.
- Gestionar y restablecer las reglas del firewall. 

**Prerrequisitos:**

- Acceso a tu máquina Ubuntu (se recomienda VM para practicar sin afectar tu máquina principal). 

> **IMPORTANTE:** Si te conectas a la VM por SSH, **ASEGÚRATE** de añadir una regla para permitir el puerto SSH (22) **ANTES** de habilitar el firewall, o te bloquearás a ti mismo. 

### Instrucciones:

### Tarea 1.  Verificar el estado inicial de UFW:
    * `sudo ufw status verbose`
    * Lo más probable es que esté `inactive` o `active` con reglas predeterminadas. 

### Tarea 2. Establecer políticas por defecto de UFW:

Paso 1 Establece la política por defecto para denegar todo el tráfico entrante y permitir todo el tráfico saliente (recomendado para seguridad).
    * `sudo ufw default deny incoming`
    * `sudo ufw default allow outgoing`

Paso 2. Verifica las políticas aplicadas:
        * `sudo ufw status verbose`

### Tarea 3. Permitir el tráfico esencial (SSH):

> *¡Paso Crítico para conexiones SSH!

Paso 1. Permite el puerto 22 (SSH) para no perder el acceso remoto. 
    * `sudo ufw allow ssh` # UFW reconoce el servicio 'ssh' y lo asocia al puerto 22/tcp
    * `# O de forma explícita: sudo ufw allow 22/tcp`

Paso 2.  Verifica que la regla se añadió:
        * `sudo ufw status verbose`

### Tarea 4.  Habilitar el firewall UFW:

    * `sudo ufw enable`
Paso 1. Se te preguntará si estás seguro. Confirma con `y`.

Paso 2. Verifica que UFW está activo:
        * `sudo ufw status verbose` # Debería mostrar 'Status: active'

### Tarea 5. Añadir reglas para otros servicios comunes:
    
Paso 1. Permite el tráfico web (puertos 80 para HTTP y 443 para HTTPS):
        * `sudo ufw allow http`
        * `sudo ufw allow https`
        * `sudo ufw status verbose`

Paso 2. Permite un puerto no estándar específico (ej., puerto 8080 para una aplicación web):
        * `sudo ufw allow 8080/tcp`
        * `sudo ufw status verbose`

### Tarea 6. Denegar tráfico explícitamente:
    * Denegar un puerto inseguro como Telnet (puerto 23):
        * `sudo ufw deny 23/tcp`
        * `sudo ufw status verbose`

### Tarea 7. Probar las reglas del firewall:
    * *Desde otra máquina:*

Paso 1. Si tienes un servidor web (Apache/Nginx) instalado y corriendo en tu VM, intenta acceder a él desde un navegador. Debería funcionar (puertos 80/443 abiertos). 

Paso 2. Intenta conectar al puerto 23 de tu VM (Telnet) desde otra máquina (ej., usando `telnet <IP_DE_TU_UBUNTU> 23`). Debería fallar (conexión denegada).

Desde la misma VM (para verificar el tráfico saliente):

Paso 3. Intenta hacer un ping a un sitio web (debería funcionar, ya que el tráfico saliente está permitido): `ping -c 3 google.com`

### Tarea 8. Gestionar y restablecer las reglas del firewall (después del laboratorio):

Paso 1. Lista las reglas con números para facilitar su eliminación:
        * `sudo ufw status numbered`

Paso 2. Borra una regla específica por su número (ej., si la regla de Telnet es la número 3):
        * `sudo ufw delete 3`

Paso 3. Restablecer UFW a su estado predeterminado (elimina todas las reglas añadidas):
        * `sudo ufw reset`

Paso 4. Esto te preguntará si estás seguro. Confirma con `y`. 
    * Esto dejará UFW deshabilitado si no lo habilitaste después de reset. 

Paso 5. Deshabilitar UFW completamente:
        * `sudo ufw disable`

---

# Práctica 3.5. Análisis básico de tráfico de red con tcpdump

## Objetivos de la práctica:

- Instalar y utilizar la herramienta `tcpdump` para capturar tráfico de red. 
- Aplicar filtros para capturar tráfico específico (por host, puerto, protocolo). 
- Guardar y leer capturas de tráfico en formato `.pcap`.
- Comprender cómo `tcpdump` puede ayudar en el diagnóstico de problemas de red. 

**Prerrequisitos:**

- Paquete `tcpdump` instalado: `sudo apt install tcpdump`.
- Un entorno donde puedas generar tráfico de red (ej. otra terminal para `ping`, un navegador web, un servidor web). 

### Instrucciones:

### Tarea 1.  Identifica tu interfaz de red principal:

Paso 1. Necesitarás saber qué interfaz está manejando tu tráfico (ej., `enp0s3`, `eth0`, `wlan0`).
    * `ip a show`
    * `# O para listar interfaces que tcpdump puede ver:`
    * `sudo tcpdump -D`

Paso 2. Toma nota del nombre de tu interfaz (usaremos `<NOMBRE_INTERFAZ>` en los ejemplos). 

### Tarea 2. Captura todo el tráfico en una interfaz (interrumpe con Ctrl+C):
Esto mostrará todos los paquetes que pasan por la interfaz.

Observa las direcciones IP, puertos y protocolos.
    * `sudo tcpdump -i <NOMBRE_INTERFAZ>`


### Tarea 3. Filtra el tráfico por host:

Paso 1. Captura solo el tráfico hacia/desde una dirección IP específica (ej., `8.8.8.8` - Google DNS):
     
        * `sudo tcpdump -i <NOMBRE_INTERFAZ> host 8.8.8.8`
Mientras este comando está corriendo en una terminal, en otra terminal, genera tráfico:

Paso 2. Haz un `ping -c 3 8.8.8.8` para ver los paquetes ICMP. 

Paso 3. Haz un `dig google.com @8.8.8.8` para ver los paquetes DNS. 

Paso 4. Observa cómo `tcpdump` solo muestra el tráfico relacionado con `8.8.8.8`. 

### Tarea 4. Filtra el tráfico por puerto:

Paso 1. Captura solo el tráfico HTTP (puerto 80). 

Paso 2. Si tienes un servidor web en tu VM, accede a él desde un navegador en otra máquina. 
     `sudo tcpdump -i <NOMBRE_INTERFAZ> port 80`

Paso 3. Captura solo el tráfico SSH (puerto 22). 

Paso 4. Si te conectas por SSH, abre una nueva sesión SSH a tu VM. 
    * `sudo tcpdump -i <NOMBRE_INTERFAZ> port 22`

### Tarea 5. Filtra el tráfico por protocolo:

Paso 1. Captura solo el tráfico ICMP (usado por `ping`):
         `sudo tcpdump -i <NOMBRE_INTERFAZ> icmp`

Paso 2. Mientras corre, haz un `ping localhost` o `ping 8.8.8.8` en otra terminal.

Paso 3. Captura solo el tráfico DNS (usado por `dig`). DNS utiliza UDP en el puerto 53: 
        * `sudo tcpdump -i <NOMBRE_INTERFAZ> udp port 53`

Paso 4. Mientras corre, haz un `dig example.com` en otra terminal. 

### Tarea 6.  Guardar y leer capturas de tráfico:

Paso 1. Guarda una captura en un archivo `.pcap` (útil para análisis posterior o para abrir con Wireshark).

Paso 2. Captura 100 paquetes (`-c 100`) y guárdalos en `mi_captura.pcap`:
        * `sudo tcpdump -i <NOMBRE_INTERFAZ> -w mi_captura.pcap -c 100`

Paso 3. Lee el contenido del archivo `.pcap` que acabas de crear:
        * `sudo tcpdump -r mi_captura.pcap`

### Tarea 7. Limpia (elimina el archivo de captura):
    * `rm mi_captura.pcap`


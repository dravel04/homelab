# Dockerlab
Guia para configurar como homelab una laptop vieja (2009), para trabjar con Docker

- [Especificaciones](#especificaciones)
- [Instalacion de Ubuntu](#instalacion-de-ubuntu)
- [Configuracion de red](#configuracion-de-red)
- [Instalacion de Docker](#instalacion-de-docker)
- [Enlaces](#enlaces)
- [Changelog](#changelog)

## Especificaciones
La laptop usada es del 2009, bastante antigua, cuenta con:
- Intel(R) Pentium(R) Dual  CPU  T3200  @ 2.00GHz
- 4GB RAM a 800Mhz
- 500GB HDD 3,5" a 5600rpm
```sh
bronzeqq@dockerlab:~$ lscpu 
Architecture:           x86_64
  CPU op-mode(s):       32-bit, 64-bit
  Address sizes:        36 bits physical, 48 bits virtual
  Byte Order:           Little Endian
CPU(s):                 2
  On-line CPU(s) list:  0,1
Vendor ID:              GenuineIntel
  Model name:           Intel(R) Pentium(R) Dual  CPU  T3200  @ 2.00GHz
    CPU family:         6
    Model:              15
    Thread(s) per core: 1
    Core(s) per socket: 2
    Socket(s):          1
    Stepping:           13
    CPU max MHz:        2000.0000
    CPU min MHz:        1000.0000
...
bronzeqq@dockerlab:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.7Gi       236Mi       2.8Gi       1.0Mi       685Mi       3.2Gi
Swap:            9Gi          0B         9Gi
```

## Instalacion de Ubuntu
Para mi homelab he elegido Ubuntu 22 LTS, por facilidad de uso e instalación.
1. Elegir el tipo de versión a instalar: **Ubuntu Server**, ya que necesitaremos logar en la máquina
2. Configuración de red: En mi caso, ya que mi laptop la tengo conectada por Wi-Fi y no me reconocía la tarjeta de red `Continue without network`
    1. Configuración del proxy: En mi caso no aplica, lo dejamos en blanco
    2. Configuración Ubuntu Archive Mirror: *Valor por defecto*
    3. [Opcional] Instalar actualizaciones: Como no configuré internet, no aplica `Continue without updating`
3. Particionado del disco: Posiblemente una de los pasos mas dificiles. En mi caso decidí una opción sencilla pero manual. El resumen de mi asignación de espacio:
    - **/ (raíz)**: `150GB`: Dejando un espacio suficiente para el sistema operativo y las aplicaciones, permitiendo espacio adicional para actualizaciones y futuras instalaciones de software.
    - **/boot**: `2GB`: Ya que estoy utilizando una laptop antigua con un solo núcleo. Debería ser suficiente para mantener los archivos del cargador de arranque y los núcleos del sistema operativo.
    - **/home**: `250GB`: Esta partición es generosa y proporcionará mucho espacio para almacenar datos personales, archivos y configuraciones de usuario. Es bueno tener un espacio amplio en /home para evitar problemas de almacenamiento a largo plazo.
    - **swap**: `10GB`: Cantidad más que de sobra para el sistema en caso de que la memoria RAM se llene.
    - `52GB` de `espacio libre sin asignar`, reserva de espacio adicional para utilizar en el futuro si fuese necesario. Este espacio libre también puede ser útil para operaciones de mantenimiento del sistema o para realizar ajustes en las particiones más adelante, si fuese necesario.
```sh
bronzeqq@dockerlab:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       147G  4.2G  135G   3% /
/dev/sda4       2.0G  251M  1.6G  14% /boot
/dev/sda5       246G   96K  233G   1% /home
```
    
4. Configuración de Usuario/Hostname: Configuramos el nombre del equipo (se puede modificar en cualquier momento) y las credenciales para realizar el primer login tras la instalación.
> [!TIP] 
> Por defecto las credenciales de root no vienen configuradas. Para ello, cuando loguemos con nuestro usuario tenemos que setearla mediante sudo passwd root
5. Configuración SSH: Recomiendo marcar la opción `Install OpenSSH Server`, así podremos conectar vía SSH nada mas terminar la instalación.
6. Lanzar la instalación y reiniciar

## Configuracion de red
Algunas consideraciones previas:
- En mi caso, decido asignar IP fija para no estar modificando el `/etc/hosts` de mi equipo al alias asignado
- opción de configuración `optional: true` para evitar el error `a start job is running for wait for network to be configured`

1. Indentificamos el nombre de nuestras interfaces de red: `ls /sys/class/net`
```sh
bronzeqq@dockerlab:~$ ls /sys/class/net
enp2s0  lo  wlx002163aa274c
```
2. Modificamos los ficheros de la ruta `/etc/netplan/`. Por simplicidad, borré los ficheros y dejé solo uno:
```sh
bronzeqq@dockerlab:~$ ls /etc/netplan/
00-installer-config.yaml
```
El fichero de configuración debe quedar similar a [00-installer-config.yaml](./network/00-installer-config.yaml) (He dejado algunos placeholder para facilitar el trabajo).
En caso de no querer configurar IP estáticas (no recomendado), el fichero de configuración se simplificaría:
```YML
network:
  ethernets:
    ETH_ADAPTER_HERE:
      dhcp4: true
      optional: true
  wifis:
    WIFI_ADAPTER_HERE:
      dhcp4: true
      optional: true
      access-points:
        "SSID_NAME_HERE":
          password: "PASSWORD_HERE"
  version: 2
```
> [!TIP]
> Si solo tienes interfaz ethernet o wifi, no hace falta que configures ambas

3. Aplicamos los cambios: `sudo netplan apply`
4. Para verificar la asignación de IP, podemos lanzar el comando `ip a`

## Instalacion de Docker
Siguiendo la documentación oficial:
1. Configuramos el repositorio Docker de `apt`:
```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. Instalamos la última versión de Docker:
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> [!IMPORTANT]  
> Tras la instalación debemos agregar nuestro usuario en grupo `docker` para no tener que estar usando el comando `sudo` [link](https://docs.docker.com/engine/install/linux-postinstall/)
> 1. Creamos el grupo `docker` (suele crearse automaticamente durante la instalacion) y añadimos nuestro usuario al grupo `docker`: 
> ```sh
> sudo groupadd docker
> sudo usermod -aG docker $USER
> ```
> 2. Aplicamos los cambios sin reiniciar y validamos
> ```sh
> newgrp docker
> docker run hello-world
> ```

## Enlaces
Enlances de referencia que he usado durante el proceso:
- [How To Install Ubuntu 22.04 LTS Server Edition](https://ostechnix.com/install-ubuntu-server/)
- [A start job is running...](https://askubuntu.com/questions/972215/a-start-job-is-running-for-wait-for-network-to-be-configured-ubuntu-server-17-1)
- [Connect to WiFi from command line](https://linuxconfig.org/ubuntu-20-04-connect-to-wifi-from-command-line)
- [Instalacion Docker](https://docs.docker.com/engine/install/linux-postinstall/)

## Changelog
**DATE_FORMAT**: *%d/%m/%Y*
- **06/03/2023** - Configuración del laboratorio

### TODO
- [ ] Usar `envsubst < 00-installer-config.yaml` para poblar el fichero de configuración de red
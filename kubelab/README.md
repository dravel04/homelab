# Kubelab
Guía, para trabajar con Kubernetes

- [Especificaciones](#especificaciones)
- [Instalacion de Ubuntu](#instalacion-de-ubuntu)
- [Configuracion de red](#configuracion-de-red)
- [Instalacion de Docker](#instalacion-de-docker)
- [Enlaces](#enlaces)
- [Changelog](#changelog)

## Especificaciones
MiniPC cuenta con:
- Intel(R) N100  @ 700MHz
- 16GB RAM
- 500GB SDD
```sh
bronzeqq@kubelab:~# lscpu
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          39 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   4
Vendor ID:                GenuineIntel
  Model name:             Intel(R) N100
    CPU family:           6
    Model:                190
    Thread(s) per core:   1
    Core(s) per socket:   4
    Socket(s):            1
    CPU max MHz:          3400.0000
    CPU min MHz:          700.0000
...
bronzeqq@kubelab:~# free -h
               total        used        free      shared  buff/cache   available
Mem:            15Gi       527Mi        14Gi        11Mi       368Mi        14Gi
Swap:            9Gi          0B         9Gi
```

## Instalacion de Debian
Para mi homelab he elegido Debian 12.5, ya que he tenido muchos problemas a la hora de instalar distros como RHEL 9.3 o Ubuntu 22/24.
1. Elegir el tipo de versión a instalar: **Debian 12.5**
2. Particionado del disco: Posiblemente una de los pasos mas dificiles. En mi caso decidí una opción sencilla pero manual. El resumen de mi asignación de espacio:
  - **/ (raíz)**: `120GiB`: Dejando un espacio suficiente para el sistema operativo y las aplicaciones, permitiendo espacio adicional para actualizaciones y futuras instalaciones de software.
  - **/boot** y **/boot/efi**: `2GiB`: Debería ser suficiente para mantener los archivos del cargador de arranque y los núcleos del sistema operativo.
  - **/home**: `80GiB`: Como la mayoría de los datos irán en los volumen asignados a kuberentes, el /home apenas tendrá uso, aún así le he asignado un generoso espacio
  - **swap**: `10GiB`: Cantidad más que de sobra para el sistema en caso de que la memoria RAM se llene.
  - **/tmp**: `10GiB`: He montado una partición para el `/tmp` con la idea de tener mayor aislamiento y seguridad
  - `270GiB` de `espacio libre sin asignar`, la idea es crear un volumen LVM para tener diferentes particiones a modo de Persistent Volume (PV) en kubernetes
```sh
bronzeqq@kubelab:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1  118G  1.8G  110G   2% /
/dev/nvme0n1p2   79G   40K   75G   1% /home
/dev/nvme0n1p3  1.2G   96M 1002M   9% /boot
/dev/nvme0n1p5  9.8G   40K  9.3G   1% /tmp
/dev/nvme0n1p6  818M  5.9M  812M   1% /boot/efi
```

## Configuracion de red
Algunas consideraciones previas:
- En mi caso, decido asignar IP fija para no estar modificando el `/etc/hosts` de mi equipo al alias asignado
- La tarjeta el driver preinstalado falla con la tarjeta de red que monta el miniPC

### Problemas con el driver `r8169` y tarjetas de red **Realtek**
Descargamos el driver compatible `r8168` 
1. Añadimos el repositorio recomendado 
```
echo -e "\n# Repo to install r8168 eth driver\ndeb http://ftp.es.debian.org/debian bookworm main non-free" | sudo tee -a /etc/apt/sources.list

sudo apt update
sudo apt install -y r8168-dkms
```
2. Reiniciamos la máquina
```
sudo reboot
```

## Links de interés
- [Fix Ethernet](https://medium.com/@dawnbreather/fixing-realtek-rtl8111-8168-8411-pci-express-gigabit-ethernet-controller-issues-after-upgrading-to-d7a985ae49ef)
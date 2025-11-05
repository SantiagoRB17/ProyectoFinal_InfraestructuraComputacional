# Bitácora 2 – Configuración de RAID

## 1. Objetivo

En esta segunda etapa del proyecto de **Infraestructura Computacional**, se implementaron los **arreglos RAID 1** sobre el servidor **Ubuntu Server 24.04.3 LTS** previamente configurado.  

El objetivo de esta etapa fue construir un entorno de almacenamiento seguro, redundante y flexible, que sirviera como base para los servicios que se desplegarán en contenedores Docker y Podman durante las próximas etapas del proyecto.

---

## 2. Instalación de herramientas necesarias

El primer paso de esta etapa fue instalar las herramientas necesarias para la administración de RAIDs y volúmenes lógicos, en este proyecto se utilizaran las herramientas **mdadm** que permite crear y gestionar arreglos RAID por software y **lvm2** que permite administrar volúmenes lógicos y grupos de volúmenes. Para instalaran estas herramientas se utilizaron los siguiente comandos:

- sudo apt update && sudo apt upgrade -y
- sudo apt install mdadm lvm2 -y 

El primer comando **(apt update && apt upgrade -y)** actualiza los repositorios e instala las últimas versiones de los paquetes del sistema, asegurando que Ubuntu tenga la base más reciente y estable. Con el segundo comando **(sudo apt install mdadm lvm2 -y)** se instala mdadm, la herramienta que permite combinar discos físicos para crear arreglos RAID por software, y lvm2, el conjunto de utilidades encargado de administrar los volúmenes lógicos.

Esta combinación de proporciona las herramientas necesarias para construir un almacenamiento flexible y tolerante a fallos.

**Evidencia:**
- *Figura 1.* Actualizacion del sistema – `actualizacion_sistema.png`  
- *Figura 2.* Instalación de herramientas RAID y LVM – `instalacion_herramientas.png`  
---

## 3. Verificación de los discos disponibles

Antes de iniciar la configuración, se listaron los discos del sistema con:

- sudo fdisk -l

El servidor cuenta con siete discos: uno principal de 20 GB para el sistema operativo y seis discos de 2 GB destinados a los RAIDs.

**Evidencia:**
- *Figura 3.* Listado de discos detectados – `listado_discos.png` 
---

## 4. Creación de los tres arreglos RAID 1

Una vez verificados los discos disponibles, se procedió con la creación de tres arreglos RAID, cada uno compuesto por dos discos de 2 GB. Para este proyecto, se asignó un arreglo RAID 1 (mirroring) a cada servicio (Apache, MySQL y Nginx) con el fin de garantizar la redundancia total de la información.
Los arreglos raid permiten replicar automáticamente los datos entre ambos discos del arreglo, de modo que, si uno de ellos presenta una falla física, el otro mantiene una copia exacta y operativa, asegurando la disponibilidad continua del servicio, para crear estos 3 arreglos se utilizo el siguiente comando:

- sudo mdadm --create --verbose /dev/mdX --level=1 --raid-devices=2 /dev/sdX /dev/sdY

Donde:
 - --create indica la creación de un nuevo arreglo RAID.

 - --verbose muestra información detallada del proceso.

 - /dev/mdX define el nombre del nuevo dispositivo RAID (md0, md1, md2).

 - --level=1 establece el tipo de RAID (nivel 1 – espejo).

 - --raid-devices=2 indica que el arreglo estará compuesto por dos discos.

Los tres arreglos creados fueron:

RAID 1 para Apache:
- sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
RAID 1 para MySQL:
- sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
RAID 1 para Nginx:
- sudo mdadm --create --verbose /dev/md2 --level=1 --raid-devices=2 /dev/sdf /dev/sdg

Durante la ejecución, el sistema muestra una advertencia informando que estos arreglos tienen metadatos al inicio del disco, lo cual puede impedir su uso como dispositivos de arranque (boot). Sin embargo, esto es normal, ya que los RAIDs creados se usarán únicamente para almacenamiento, no para iniciar el sistema.

Después de confirmar la creación (respondiendo “y” en cada caso), se observa cómo el sistema asigna el tamaño de cada arreglo (~2 GB) y crea automáticamente las estructuras de metadatos (version 1.2 metadata).

Como resultado, se muestran los mensajes de confirmación:

- mdadm: array /dev/md0 started.
- mdadm: array /dev/md1 started.
- mdadm: array /dev/md2 started.

Estos mensajes indican que los tres arreglos fueron creados correctamente y se encuentran activos, listos para ser utilizados en la configuración de LVM.

Posteriormente, se verificó el estado de los arreglos con:

- cat /proc/mdstat
- sudo mdadm --detail /dev/md0
- sudo mdadm --detail /dev/md1
- sudo mdadm --detail /dev/md2

Estos comandos muestran el progreso de la sincronización y los detalles internos de cada RAID, como su tamaño, estado y discos participantes.

**Evidencias:**
- *Figura 4.* Creacion de los arreglos RAID1 – `creacion_raids.png` 
- *Figura 5.* Verificación del estado de los RAIDs y verificacion del estado del raid md0 - `estado_raids_y_raid_md0.png` 
- *Figura 6* Verificacion del estado del RAID md1 - `estado_raid_md1.png` 
- *Figura 7* Verificacion del estado del RAID md2 - `estado_raid_md2.png`

---

## 5. Registro de los arreglos RAID en el sistema

Después de crear y verificar los tres arreglos RAID, se procedió a registrarlos en el sistema operativo para que fueran reconocidos automáticamente en cada inicio.

Primero, se obtuvo el identificador UUID de cada arreglo a partir de la verificación realizada en el paso anterior con el comando:

- sudo mdadm --detail /dev/md0
- sudo mdadm --detail /dev/md1
- sudo mdadm --detail /dev/md2

Con estos identificadores UUID, se procedió a editar el archivo de configuración principal de mdadm con el siguiente comando:
- sudo nano /etc/mdadm/mdadm.conf

Dentro del archivo, se añadió una nueva línea por cada arreglo RAID:
- ARRAY /dev/md0 UUID=<UUID_del_RAID_0>
- ARRAY /dev/md1 UUID=<UUID_del_RAID_1>
- ARRAY /dev/md2 UUID=<UUID_del_RAID_2>

De esta forma, cada arreglo queda identificado de manera única por su UUID, permitiendo que el sistema los reconozca correctamente en cada reinicio.

Finalmente, para aplicar los cambios y asegurar su persistencia, se actualizó la imagen del sistema de arranque con el comando:
- sudo update-initramfs -u

Este paso recompila el initramfs incluyendo la configuración recién añadida, garantizando que los arreglos RAID estén disponibles automáticamente al iniciar el sistema.

**Evidencias:**
- *Figura 8.* Edición manual del archivo /etc/mdadm/mdadm.conf con los UUID de los RAIDs – `edicion_archivo_mdadm.conf.png` 
- *Figura 9.* Actualización del initramfs luego de registrar los RAIDs – `actualizacion_imagen_sistema.png`

---

## Conclusion

A través de la creación de tres arreglos RAID 1 (mirroring), se logró establecer un sistema capaz de replicar la información entre discos, garantizando la disponibilidad de los datos incluso ante una posible falla física. El resultado es una infraestructura sólida y confiable que servirá como base para las siguientes fases del proyecto, donde los volúmenes creados se integrarán con el sistema LVM para ofrecer mayor flexibilidad y control del almacenamiento.
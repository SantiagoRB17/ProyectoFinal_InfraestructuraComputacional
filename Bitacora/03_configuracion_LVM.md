# Bitácora 3 – Configuracion de Volúmenes Lógicos (LVM)

## 1. Objetivo

El objetivo principal de esta etapa es consolidar un sistema de almacenamiento redundante y flexible, integrando las ventajas de la seguridad ofrecida por RAID 1 con la capacidad de expansión dinámica de LVM, lo cual permitirá preparar el entorno para el despliegue de servicios como Apache, MySQL y Nginx en etapas posteriores.

---

## 2. Herramientas
Las herramientas necesarias para la gestión de LVM (el paquete lvm2) fueron instaladas previamente durante la bitácora de configuración de los RAIDs, por lo que en esta etapa se retoma el entorno ya preparado para iniciar directamente con la creación de los volúmenes físicos, grupos de volúmenes y volúmenes lógicos.

---

## 3. Creación de los volúmenes físicos (PV)

El primer paso en la jerarquía de LVM consiste en convertir los dispositivos RAID en volúmenes físicos (Physical Volumes, PV), que serán la base del sistema lógico de almacenamiento.

Para ello se ejecutaron los siguientes comandos:

- sudo pvcreate /dev/md0
- sudo pvcreate /dev/md1
- sudo pvcreate /dev/md2

Cada ejecución devuelve el mensaje “Physical volume successfully created”, indicando que los tres arreglos RAID fueron correctamente inicializados como volúmenes físicos.

**Evidencia:**
- *Figura 1.* Creación de volúmenes físicos a partir de los arreglos RAID - `creacion_volumenes_fisicos.png`
  ![Figura 1 - Creación de volúmenes físicos a partir de los arreglos RAID](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\creacion_volumenes_logicos.png)  

---

## 4. Creación de los grupos de volúmenes (VG)

Una vez definidos los volúmenes físicos, se agrupan en grupos de volúmenes (Volume Groups, VG), los cuales funcionan como contenedores de espacio administrable.

En este caso, se creó un grupo de volúmenes por servicio:

- sudo vgcreate vg_apache /dev/md0
- sudo vgcreate vg_mysql /dev/md1
- sudo vgcreate vg_nginx /dev/md2

Cada comando devuelve una confirmación con el mensaje “Volume group successfully created”, señalando la creación de los grupos de volúmenes vg_apache, vg_mysql y vg_nginx.

**Evidencias:**
- *Figura 2.* Creación del grupo de volumen para apache - `creacion_grupo_volumen_apache.png` 
  ![Figura 2 - Creación del grupo de volumen para apache](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\creacion_grupo_volumen_apache.png) 
- *Figura 3.* Creación del grupo de volumen para mysql - `creacion_grupo_volumen_mysql.png` 
  ![Figura 3 - Creación del grupo de volumen para mysql](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\creacion_grupo_volumen_mysql.png) 
- *Figura 4.* Creación del grupo de volumen para nginx - `creacion_grupo_volumen_nginx.png`
  ![Figura 4 - Creación del grupo de volumen para nginx](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\creacion_grupo_volumen_nginx.png) 

---

## 5. Creación de los volúmenes lógicos (LV)

A continuación, se crearon los volúmenes lógicos (Logical Volumes, LV) dentro de cada grupo de volúmenes.
Estos actúan como unidades virtuales que pueden formatearse y montarse de forma independiente.

Se decidió utilizar todo el espacio disponible en cada grupo, mediante el parámetro -l 100%FREE:

- sudo lvcreate -l 100%FREE -n lv_apache vg_apache
- sudo lvcreate -l 100%FREE -n lv_mysql vg_mysql
- sudo lvcreate -l 100%FREE -n lv_nginx vg_nginx

Al finalizar, el sistema reportó “Logical volume created successfully” para cada uno.

**Evidencia:**
- *Figura 5.* Creación de volúmenes lógicos - `creacion_volumenes_logicos.png`
  ![Figura 5 - Creación de volúmenes lógicos](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\creacion_volumenes_logicos.png) 
---

## 6. Formateo y montaje de los volúmenes

Con los volúmenes lógicos creados, el siguiente paso fue formatearlos con el sistema de archivos ext4 y montarlos en rutas específicas para cada servicio.

- sudo mkfs.ext4 /dev/vg_apache/lv_apache
- sudo mkfs.ext4 /dev/vg_mysql/lv_mysql
- sudo mkfs.ext4 /dev/vg_nginx/lv_nginx

- sudo mkdir /mnt/apache
- sudo mkdir /mnt/mysql
- sudo mkdir /mnt/nginx

- sudo mount /dev/vg_apache/lv_apache /mnt/apache
- sudo mount /dev/vg_mysql/lv_mysql /mnt/mysql
- sudo mount /dev/vg_nginx/lv_nginx /mnt/nginx

De esta forma, cada volumen lógico quedó montado en su punto de acceso correspondiente dentro del sistema de archivos.

**Evidencias:**
- *Figura 6.* Formateo de los volumenes lógicos con el sistema de archivos ext4 - `formateo_a_ext4.png`
  ![Figura 6 - Formateo de los volumenes lógicos con el sistema de archivos ext4](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\formateo_a_ext4.png) 
- *Figura 7.* Montaje de los volúmenes lógicos en /mnt - `creacion_rutas_y_montaje_volumenes.png`
  ![Figura 7 - Montaje de los volúmenes lógicos en /mnt](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\creacion_rutas_y_montaje_volumenes.png) 

---

## 7. Configuración del montaje automático

Para asegurar que los volúmenes se monten automáticamente al reiniciar el servidor, se añadieron las siguientes líneas al archivo /etc/fstab con:

- sudo nano /etc/fstab

Las lineas agregadas fueron:

- /dev/vg_apache/lv_apache /mnt/apache ext4 defaults 0 2
- /dev/vg_mysql/lv_mysql /mnt/mysql ext4 defaults 0 2
- /dev/vg_nginx/lv_nginx /mnt/nginx ext4 defaults 0 2

Con esta configuración, el sistema montará los volúmenes de manera automática en cada inicio, garantizando su disponibilidad sin intervención manual.

**Evidencia:**
- *Figura 8.* Configuración del archivo /etc/fstab - `configuracion_archivo_fstab.png`
  ![Figura 7 - Configuración del archivo /etc/fstab](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\configuracion_archivo_fstab.png) 

---

## 8. Verificación del estado de LVM

Finalmente, se verificó el estado de los volúmenes físicos, grupos y lógicos creados, junto con el espacio disponible, mediante los siguientes comandos:

- sudo pvs
- sudo vgs
- sudo lvs
- df -h

Las salidas confirmaron que los tres volúmenes estaban montados correctamente y utilizando todo el espacio de sus respectivos arreglos RAID.

**Evidencia:**
- *Figura 9.* Verificación del estado de los volúmenes y puntos de montaje - `verificacion_volumenes_y_montajes.png`
  ![Figura 9 - Verificación del estado de los volúmenes y puntos de montaje](ProyectoFinal_InfraestructuraComputacional\Evidencias\capturas_LVM\verificacion_volumenes_y_montajes.png) 

## Conclusion
La implementación del sistema LVM sobre los arreglos RAID 1 permitió consolidar una infraestructura de almacenamiento redundante, flexible y fácilmente escalable.
El uso combinado de estas tecnologías ofrece un equilibrio entre seguridad de los datos (gracias al mirroring) y gestión eficiente del espacio (por medio de LVM), sentando las bases para la implementación de servicios y contenedores en etapas posteriores del proyecto.
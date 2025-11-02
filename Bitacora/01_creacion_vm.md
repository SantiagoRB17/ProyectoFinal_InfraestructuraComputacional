## 1. Objetivo

El propósito de esta primera etapa es crear la máquina virtual base con **Ubuntu Server**, la cual servirá como entorno principal para el desarrollo del proyecto final.  
En esta máquina se implementarán posteriormente los sistemas **RAID 1**, **LVM** y los **contenedores Docker y Podman** con sus respectivos servicios.

---

## 2. Creación e instalación de la máquina virtual

En esta fase se configuró una máquina virtual en **VirtualBox**, la cual actuará como el servidor central del proyecto.  
Se asignaron los siguientes recursos para garantizar estabilidad y rendimiento adecuados:

- **Sistema operativo:** Ubuntu Server 24.04.3 LTS (Noble Numbat)  
- **Memoria RAM:** 2 GB  
- **Procesadores:** 2 núcleos  
- **Almacenamiento principal:** 20 GB (VDI dinámico)  

Una vez completada la instalación, el sistema inició correctamente y mostró la consola de inicio de sesión

**Evidencias:**  
- *Figura 1.* Selección de la ISO y configuración inicial – `iso_y_nombre`  
- *Figura 2.* Configuración del disco principal – `disco_principal`  
- *Figura 3.* Asignación de recursos de hardware – `recursos_hardware`  
- *Figura 4.* Creación de credenciales de usuario – `credenciales`  
- *Figura 5.* Proceso de instalación en curso – `instalacion`  
- *Figura 6.* Primer inicio de sesión exitoso – `primer_login`  

---

## 3. Actualización y preparación del entorno

Después de iniciar sesión, se realizaron las actualizaciones necesarias para dejar el servidor actualizado y funcional.  
Los comandos utilizados fueron:

- sudo apt update 
- sudo apt upgrade

**Evidencia:**
- *Figura 7.* Sistema actualizado correctamente – sistema_actualizado

---

## 4. Creación de los discos adicionales
Una vez configurado y actualizado el sistema base, se procedió a añadir seis discos adicionales, que serán utilizados posteriormente para la creación de los arreglos RAID 1 y sus respectivos volúmenes LVM.

Para esta primera etapa, los discos fueron simplemente añadidos y verificados, sin configurarse aún los arreglos ni volúmenes.
Las herramientas y configuraciones correspondientes a RAID y LVM se implementarán en la siguiente etapa.
- sdb	RAID 1 (Apache) – disco 1	2 GB	Primer disco del RAID destinado a Apache.
- sdc	RAID 1 (Apache) – disco 2	2 GB	Segundo disco espejo del RAID de Apache.
- sdd	RAID 1 (MySQL) – disco 1	2 GB	Primer disco del RAID destinado a MySQL.
- sde	RAID 1 (MySQL) – disco 2	2 GB	Segundo disco espejo del RAID de MySQL.
- sdf	RAID 1 (Nginx) – disco 1	2 GB	Primer disco del RAID destinado a Nginx.
- sdg	RAID 1 (Nginx) – disco 2	2 GB	Segundo disco espejo del RAID de Nginx.

Para verificar que todos los discos fueron detectados correctamente, se utilizó el siguiente comando:
- sudo fdisk -l
El resultado mostró los siete dispositivos (/dev/sda a /dev/sdg), confirmando que el sistema reconoce cada uno de los discos agregados.

**Evidencias:**
- *Figura 8.* Agregación de los seis discos adicionales en VirtualBox – `discos_agregados`
- *Figura 9.* Listado de discos detectados con fdisk -l – `lista_discos`

## Conclusion
En esta primera etapa se logró la instalación y configuración exitosa de la máquina virtual base con Ubuntu Server 24.04.3 LTS.
El sistema fue actualizado correctamente y se verificó el funcionamiento de los servicios esenciales.

Además, se añadieron los seis discos adicionales de 2 GB que servirán para la creación de los tres arreglos RAID 1 en la siguiente fase del proyecto.
Las herramientas para la configuración de RAID y LVM se instalaron, pero su aplicación se realizará en la próxima etapa, dedicada exclusivamente al almacenamiento.

Con esto se completó la preparación del entorno inicial, quedando listo para la implementación del almacenamiento mediante RAID y LVM.
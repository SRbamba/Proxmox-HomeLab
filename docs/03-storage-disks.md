# 📂 Documentación del Proyecto: Expansión de Almacenamiento (HDD)

## 🚀 1. Introducción y Hardware
Para transformar el nodo en un servidor de archivos y entorno de desarrollo integral, se integró una **Dock Station** externa conectada vía USB al host físico. Se añadieron dos unidades de disco mecánico de 2 TB cada una para separar cargas de trabajo y asegurar redundancia.

* **Hardware:** Dock Station Dual-Bay.
* **Discos:** 2x HDD (2 TB c/u).
* **Identificación en SO:** `/dev/sdb` y `/dev/sdc`.

---

## 🧹 2. Preparación y Limpieza (Wipe)
Antes de integrar los discos, se procedió a eliminar cualquier tabla de particiones previa desde la interfaz de Proxmox para asegurar un montaje limpio y evitar conflictos de lectura o "unidades fantasma".

1. Navegación a `svhome` -> `Disks`.
2. Selección de unidades y ejecución de **Wipe Disk**.

![Limpieza de los discos en Proxmox](../images/Captura%20desde%202026-03-31%2023-13-33.png)

---

## 🏗️ 3. Aprovisionamiento de Directorios (Ext4)
Se optó por formatear ambas unidades con el sistema de archivos **Ext4**, ideal para asegurar estabilidad en conexiones USB y manejar grandes volúmenes de datos en Linux. 

Para mantener la integridad de la infraestructura, se crearon dos puntos de montaje separados con propósitos específicos:

* **`HDD-Datos` (/dev/sdb):** Destinado al almacenamiento principal. Será el entorno para alojar los contenedores de desarrollo, las bases de datos PostgreSQL de Mabu, imágenes ISO, y repositorios de apuntes de ingeniería de la UTN.
* **`HDD-Backups` (/dev/sdc):** Unidad dedicada de forma exclusiva a las copias de seguridad periódicas (Snapshots) de los contenedores críticos (Gateway Tailscale, servicios DNS, etc.).

![Creación de los directorios Ext4](../images/Captura%20desde%202026-03-31%2023-18-30.png)

---

## 📊 4. Verificación de Montaje
Una vez inicializados, Proxmox monta automáticamente los volúmenes en el directorio `/mnt/pve/`. Ambos discos quedaron operativos y visibles en el árbol de recursos del Datacenter, listos para ser asignados a las máquinas virtuales y contenedores.

![Discos montados y operativos en el clúster](../images/Captura%20desde%202026-03-31%2023-36-34.png)

---

> **✅ Resultado Final:** Capacidad total del clúster expandida exitosamente a 4 TB adicionales, estableciendo una arquitectura de almacenamiento profesional que separa el sistema operativo, los datos de los proyectos y las

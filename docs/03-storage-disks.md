# 📂 Documentación del Proyecto: Expansión de Almacenamiento (HDD)

## 🚀 1. Introducción y Hardware
Para transformar el nodo en un servidor de archivos y centro de datos, se integró una **Dock Station** externa conectada vía USB 3.0 al host físico. Se añadieron dos unidades de disco mecánico para almacenamiento masivo y backups.

* **Hardware:** Dock Station Dual-Bay.
* **Discos:** 2x HDD Seagate/Western Digital de 2 TB cada uno.
* **Identificación en OS:** `/dev/sdb` y `/dev/sdc`.

---

## 🛠️ 2. Preparación y Limpieza (Wipe)
Antes de integrar los discos al ecosistema de Proxmox, se procedió a eliminar cualquier tabla de particiones previa (NTFS/GPT) para evitar conflictos de montado.

1.  Navegación a `svhome` -> `Disks`.
2.  Selección de unidades y ejecución de **Wipe Disk**.
3.  Inicialización con GPT para soportar volúmenes de 2 TB.

---

## 🏗️ 3. Configuración del Almacenamiento
Se optó por una arquitectura de **Directorio Nativo (Ext4)** por su estabilidad y compatibilidad con dispositivos conectados por USB.

### Configuración de la Unidad de Datos:
* **File System:** Ext4.
* **Punto de Montaje:** Automático por Proxmox en `/mnt/pve/`.
* **Nombre en el clúster:** `HDD-Storage-01`.

```bash
# Verificación de montado desde la shell del host
df -h | grep /dev/sdb
```

---

## 📊 4. Jerarquía de Almacenamiento Actual
Tras la expansión, la arquitectura de almacenamiento del nodo se divide según el tipo de carga de trabajo:

| Unidad | Capacidad | Tipo | Propósito |
| :--- | :--- | :--- | :--- |
| **SSD Local** | 256 GB | LVM-Thin | Sistema Operativo y Discos de Contenedores (LXC/VM). |
| **HDD 01** | 2 TB | Ext4 | Almacenamiento de archivos, ISOs y despliegues de Mabu. |
| **HDD 02** | 2 TB | Ext4 | Backups programados y dump de bases de datos. |



---

> **📌 Nota de Ingeniería:** Al utilizar una conexión USB, se recomienda habilitar el monitoreo S.M.A.R.T. para vigilar la salud de los discos mecánicos ante posibles desconexiones físicas o picos de tensión.

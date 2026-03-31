# 📂 Documentación del Proyecto: Infraestructura Base (Proxmox VE)

## 1. Resumen del Sistema Host
El servidor físico se encuentra operando bajo **Proxmox Virtual Environment 8.1.1**, montado sobre un procesador Intel i3 con almacenamiento local en SSD para el sistema operativo y las imágenes de los contenedores.

- **Nodo:** `svhome`
- **IP Local:** `192.168.0.50`
- **Storage:** `local` (ISO/Templates) y `local-lvm` (Discos virtuales).

## 2. Aprovisionamiento del Contenedor de Gestión (`gw-scale`)
Para la administración de la red y la salida a internet vía VPN, se desplegó el contenedor **ID 100**. Este nodo actúa como el "Gateway" lógico del laboratorio.

### Especificaciones del Despliegue:
Se utilizó la plantilla oficial de Debian para garantizar estabilidad y bajo consumo de recursos.

| Parámetro | Configuración |
| :--- | :--- |
| **OS Template** | `debian-13-standard_13.1-2_amd64.tar.zst` |
| **Hostname** | `gw-scale` |
| **Privilegios** | Unprivileged (Seguridad mejorada) |
| **Memoria** | 512 MB RAM / 512 MB Swap |

![Resumen de confirmación del LXC](Captura desde 2026-03-31 00-07-52.png)

## 3. Configuración de Red del Contenedor
El direccionamiento se configuró inicialmente vía **DHCP** sobre el puente virtual `vmbr0`. Se activó el Firewall a nivel de interfaz para permitir el filtrado selectivo de paquetes en capas superiores.

- **Interfaz:** `eth0`
- **Bridge:** `vmbr0` (Mapeado a la placa física del i3)
- **Modo:** DHCP (IPv4) / SLAAC (IPv6)

![Configuración de Interfaz de Red](Captura desde 2026-03-31 00-07-12.png)

## 4. Consola y Primer Inicio
Tras la creación, se verificó el acceso mediante la consola integrada de Proxmox (noVNC). Se procedió con la actualización de repositorios y la preparación del entorno para la instalación de herramientas de red.

![Terminal de primer inicio - gw-scale](Captura desde 2026-03-31 00-18-15.png)

> **Nota de Sysadmin:** Se detectó que el contenedor requiere la característica `nesting=1` para permitir la ejecución de ciertos servicios de red dentro de capas de virtualización LXC. Esto se configuró en la pestaña "Options" del contenedor.

## 5. Gestión de Logs y Tareas
En la parte inferior de la interfaz, se puede observar el historial de tareas (`Task History`) donde se confirma que el aprovisionamiento de los discos y la red finalizó con estado **OK**.

![Historial de Tareas de Proxmox](Captura desde 2026-03-31 00-34-41.png)

---
**Siguiente paso:** [Configuración de Red VPN y Subnet Routing (02-network-vpn.md)](02-network-vpn.md)

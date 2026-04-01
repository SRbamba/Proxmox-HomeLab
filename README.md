# Proxmox HomeLab: Infraestructura y Servicios Críticos
Este repositorio documenta el despliegue, configuración y mantenimiento de mi laboratorio de infraestructura personal (HomeLab). El objetivo de este proyecto es centralizar servicios de gestión empresarial, almacenamiento masivo y laboratorios de ciberseguridad bajo un entorno de virtualización profesional de grado empresarial.

## 🛠️ Especificaciones del Hardware
El nodo principal corre sobre una arquitectura compacta pero potente, optimizada para un bajo consumo energético y alta disponibilidad.

| Componente | Detalle | Función |
| :--- | :--- | :--- |
| **CPU** | Intel Core i3-12100 (4C/8T @ 3.30GHz) | Cómputo y segmentación de hilos para VMs |
| **RAM** | 16GB DDR4 (Plan de expansión a 32GB) | Gestión dinámica de memoria para contenedores LXC |
| **SSD (Boot)** | 256GB M.2 NVMe | Sistema Operativo (PVE) y almacenamiento de plantillas |
| **HDD (Storage)** | 2TB HGST Enterprise (Docked) | Almacenamiento masivo (ZFS / TrueNAS) |
| **Network** | Gigabit Ethernet (Intel) | Backbone de conectividad LAN |

---

## 🌐 Arquitectura de Red y Conectividad
Para garantizar el acceso seguro desde cualquier ubicación (incluyendo redes bajo **CGNAT**), se implementó una topología híbrida que prioriza la seguridad y la latencia.

### Diagrama de Conectividad


* **IP Local:** `192.168.0.50/24` (Estática)
* **Acceso Remoto Seguro:** Se utiliza **Tailscale (WireGuard)** como capa de VPN Mesh, permitiendo la administración del nodo desde redes externas sin necesidad de apertura de puertos (Port Forwarding).
* **Publicación de Servicios:** Implementación de **Cloudflare Tunnels** para exponer servicios web (HTTPS) con cifrado SSL de extremo a extremo, mitigando ataques de denegación de servicio (DDoS).

---

## 🏗️ Stack de Software y Servicios
La infraestructura se divide en contenedores ligeros (LXC) y máquinas virtuales (VM) para maximizar el aprovechamiento de los 16GB de RAM.

### 1. Gestión y Almacenamiento
* **CasaOS Core (LXC):** Gestión del disco de 2TB mediante EXT4, proporcionando redundancia y snapshots de datos.
* **Nextcloud / Immich:** Nube privada para sincronización de archivos y galería fotográfica automatizada.

### 2. Desarrollo (SaaS Mabu)
* **Backend Environment:** Entorno dedicado para aplicaciones Java/Spring Boot con persistencia en PostgreSQL.
* **CI/CD Pipeline:** (En progreso) Automatización de despliegue desde GitHub a Proxmox.

### 3. Seguridad y Observabilidad
* **AdGuard Home (LXC):** DNS Sinkhole para filtrado de publicidad y control de tráfico interno.
* **Zabbix + Grafana:** Monitoreo en tiempo real de la salud del hardware y métricas de red.


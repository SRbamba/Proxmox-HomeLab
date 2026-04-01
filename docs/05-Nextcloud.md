# Documentación Integral: Despliegue de Nextcloud y Acceso Global VPN

Este documento detalla el Procedimiento Operativo Estándar (SOP) para la instalación de la nube privada Nextcloud y la configuración del Gateway VPN (Tailscale) para acceso remoto seguro desde el exterior.

---

## 1. Arquitectura del Sistema

| Componente | Especificación Técnica | Dirección IP |
| :--- | :--- | :--- |
| **Hipervisor** | Proxmox VE | `192.168.0.50` |
| **Gateway VPN** | LXC 100 (`gw-scale`) | `192.168.0.100` |
| **Servidor NAS** | LXC 101 (`casaos-nas`) | `192.168.0.51` |
| **Endpoint App** | Docker (Nextcloud v28+) | Port: `7580` |
| **Almacenamiento** | HDD 2TB (Mapeado a `/data`) | Físico (SATA) |

---

## 2. Solución de Errores de Inicialización (Vía CLI)

Debido a fallos en el auto-aprovisionamiento, se utilizaron comandos `occ` directamente en la consola del LXC 101 para estabilizar el servicio.

**2.1. Autorización de IP (Trusted Domains)**
Para resolver el error "Acceso a través de un dominio del que no se confía":

![Error de Dominio Confiable](../images/Captura%20desde%202026-04-01%2012-23-30.png)

    docker exec -u 33 -it nextcloud php /var/www/html/occ config:system:set trusted_domains 1 --value=192.168.0.51

**2.2. Aprovisionamiento Forzado de Superusuario**
Para asegurar el acceso administrativo inicial:

![Creación de Administrador vía CLI](../images/Captura%20desde%202026-04-01%2012-32-16.png)

    # Crear usuario administrador (Interactivo)
    docker exec -u 33 -it nextcloud php /var/www/html/occ user:add --group="admin" admin

**2.3. Activación Manual de Módulos**
Se forzó la activación del plugin de almacenamiento externo para eludir problemas de DNS en la tienda de apps:

    docker exec -u 33 -it nextcloud php /var/www/html/occ app:enable files_external

---

## 3. Gestión de Almacenamiento y Usuarios

Se configuró el aislamiento de datos sobre el disco físico de 2TB mediante el módulo *External Storage*.

### 3.1. Estructura de Usuarios y Cuotas
* **Bamba:** 250 GB
* **Sol / Nora / Jorge:** 100 GB c/u
* **Admin:** Ilimitado

### 3.2. Mapeo de Carpetas Físicas
Cada usuario tiene su propio punto de montaje local para garantizar la privacidad:

![Configuración Final de Almacenamiento Externo](../images/Captura%20desde%202026-04-01%2013-02-18.png)

| Nombre en App | Ruta Física (Host) | Restringido a |
| :--- | :--- | :--- |
| Mis Archivos | `/data/Bamba` | Bamba |
| Mis Archivos | `/data/Sol` | Sol |

---

## 4. Acceso Remoto Seguro (Subnet Routing)

Para acceder a Nextcloud desde redes externas (4G/LTE), se configuró el LXC 100 como nodo de salida de la red local.

### 4.1. Configuración de Privilegios en Proxmox (Host)
Es mandatorio habilitar el reenvío de paquetes en el archivo de configuración del contenedor en el host físico (`/etc/pve/lxc/100.conf`):

    lxc.sysctl.net.ipv4.ip_forward = 1
    lxc.sysctl.net.ipv6.conf.all.forwarding = 1

### 4.2. Activación del Gateway (LXC 100)
Dentro del contenedor de Tailscale, se aplicaron las reglas de persistencia y el anuncio de la subred local:

    # Aplicar IP Forwarding en el SO
    echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
    sysctl -p /etc/sysctl.d/99-tailscale.conf

    # Anunciar ruta de la red LAN de Lanús
    tailscale up --advertise-routes=192.168.0.0/24

> **Nota:** La ruta debe ser aprobada manualmente en el panel de Tailscale (Machines > Edit route settings).

---

## 5. Implementación en Dispositivos Móviles

1. **VPN:** Conectar el cliente Tailscale en el móvil.
2. **Dirección del Servidor:** Utilizar la IP local directa `http://192.168.0.51:7580` (la VPN rutea el tráfico automáticamente).
3. **Subida Automática:** Activar en la App de Nextcloud solo bajo conexión Wi-Fi para optimizar el plan de datos.

---
**Documentación Consolidada:** 01 de Abril, 2026
**Responsable:** Ingeniería de Infraestructura - HomeLab

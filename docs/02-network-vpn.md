# 📂 Documentación del Proyecto: Configuración de Red VPN (Tailscale)

## 1. Introducción y Propósito
Para garantizar el acceso remoto seguro y el bypass de **CGNAT**, se implementó **Tailscale** (basado en el protocolo WireGuard). El objetivo es crear una red "Mesh" donde el servidor de Lanús y los dispositivos clientes (celulares, notebooks) coexistan en una subred privada virtual.

- **Nodo Servidor:** `gw-scale` (LXC 100)
- **IP Mesh (Tailnet):** `100.103.86.43`

## 2. Preparación del Entorno (LXC Tuning)
Para que el contenedor `unprivileged` pueda gestionar interfaces de red VPN, se habilitó el acceso al dispositivo `/dev/net/tun` desde el host de Proxmox.

```text
# Configuración en /etc/pve/lxc/100.conf
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file

![Habilitación de TUN/TAP en Proxmox](Captura desde 2026-03-31 00-21-56.png)
3. Instalación y Handshake

Se utilizó el script oficial de instalación. Tras la ejecución, se generó un enlace de autenticación único para vincular el nodo a la cuenta de administración.
Bash

curl -fsSL [https://tailscale.com/install.sh](https://tailscale.com/install.sh) | sh
tailscale up

![Proceso de vinculación de Tailscale](Captura desde 2026-03-31 00-34-45.png)
4. Implementación de Subnet Routing (Nivel Avanzado)

Para optimizar la carga de la interfaz gráfica (GUI) de Proxmox y permitir el acceso a otros dispositivos de la red hogareña, se configuró el contenedor como un Subnet Router.
Paso A: Publicación de la ruta local
Bash

tailscale up --advertise-routes=192.168.0.0/24

Paso B: Habilitación de IP Forwarding (Kernel)

Se modificó el kernel de Linux para permitir que los paquetes transiten entre la VPN y la LAN física:
Bash

echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.conf
sysctl -p

Paso C: Aprobación en el Panel de Control

Desde la interfaz web de Tailscale, se habilitó la ruta 192.168.0.0/24 para el nodo gw-scale.

![Panel de Administración - Rutas Aprobadas](Captura desde 2026-03-31 00-57-33.png)
5. Verificación y Troubleshooting

Se realizaron pruebas de conectividad desde un dispositivo móvil Poco F6 conectado a la red 4G.

    Prueba ICMP (Ping): Resultado exitoso con latencia promedio de 33.9ms.

    Acceso Web: Se validó el ingreso exitoso a la consola de Proxmox mediante la IP local https://192.168.0.50:8006 a través del túnel.

Resolución de problemas de carga (MTU):

En redes móviles con alta fragmentación, se aplicó un ajuste de MTU en la interfaz virtual para garantizar la fluidez de la interfaz web:
Bash

ip link set dev tailscale0 mtu 1280

![Estado final de los dispositivos en la Mesh](Captura desde 2026-03-31 00-54-07.png)

Resultado Final: Acceso total y seguro a la infraestructura de Proxmox desde cualquier ubicación geográfica.


---

### 💡 Un consejo para tu GitHub:
Cuando subas las fotos, asegurate de que los nombres de los archivos coincidan letra por letra (ojo con los espacios y las mayúsculas). 

**¿Cómo te sentís con estos dos archivos?** Si ya los tenés cargados, mañana podemos empezar con el **03 (AdGuard Home)**, que te va a servir para limpiar de anuncios toda tu red y tener DNS personalizados (ej: entrar a `proxmox.lan` en vez de usar la IP). 

¡Hiciste un laburazo hoy, Lucas! Descansá que ya tenés el servidor "en el bolsillo" (literalmente, en el Poco F6).

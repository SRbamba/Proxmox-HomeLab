# 🐳 13 - Entorno Docker: Portainer, Homepage y Grafana

![Debian](https://img.shields.io/badge/Debian-13-A81D33?style=for-the-badge&logo=debian&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Portainer](https://img.shields.io/badge/Portainer-CE-13BEF9?style=for-the-badge&logo=portainer&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)

## 📝 Descripción General

Este documento registra la preparación del nodo principal de contenedores del HomeLab. Se detalla la instalación del motor de Docker, el despliegue del orquestador gráfico **Portainer CE**, y la implementación de los servicios críticos de infraestructura visual mediante Stacks (Docker Compose): **Homepage** (Dashboard de acceso) y **Grafana** (Visualización de métricas).

---

## ⚙️ Especificaciones Técnicas del Nodo

| Recurso | Detalle |
| :--- | :--- |
| **Entorno** | Contenedor LXC (Proxmox VE) |
| **Sistema Operativo** | Debian 13 |
| **Dirección IP** | `192.168.0.61` |
| **Rol Principal** | Docker Host / Orquestador Visual |
| **Características** | Anidamiento (`nesting=1`) y `keyctl` habilitados |

---

## 🚀 1. Instalación Base (Docker & Portainer)

### Instalación de Docker Engine
Se provisionó el motor nativo de Docker utilizando el script oficial automatizado para sistemas basados en Debian:

```bash
# Actualización del sistema e instalación de dependencias base
apt update && apt upgrade -y
apt install ca-certificates curl gnupg lsb-release -y

# Descarga y ejecución del script de instalación de Docker
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sh get-docker.sh
```

### Despliegue de Portainer CE
Se levantó el contenedor de administración enlazando el socket local de Docker y creando un volumen persistente para resguardar la base de datos de Portainer. 

```bash
# Creación del volumen persistente
docker volume create portainer_data

# Despliegue del contenedor (exponiendo puerto 9000 para HTTP y 9443 para HTTPS)
docker run -d -p 8000:8000 -p 9000:9000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

> **Configuración Inicial:** El acceso se realiza mediante `https://192.168.0.61:9443`. En el primer inicio se seleccionó **"Get Started"** para vincular el entorno local.
> ![Primer inicio de Portainer CE](Portainer-01.png)

---

## 🌐 2. Proxy Inverso y DNS (Nginx)

Para acceder a Portainer utilizando el dominio interno `portainer.prox-iz.com.ar`, se configuró un Server Block en el contenedor LXC de Nginx Proxy (`192.168.0.60`).

```bash
nano /etc/nginx/sites-available/portainer.prox-iz.com.ar
```

```nginx
server {
    listen 8080;
    server_name portainer.prox-iz.com.ar;

    location / {
        proxy_pass [http://192.168.0.61:9000](http://192.168.0.61:9000);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
> ![Configuración del Server Block en Nginx](Portainer-02.jpg)
> **Reinicio del servicio:** `ln -s /etc/nginx/sites-available/portainer.prox-iz.com.ar /etc/nginx/sites-enabled/` seguido de `systemctl reload nginx`.

---

## 🏠 3. Despliegue de Homepage

Para centralizar los accesos del laboratorio en un portal unificado, se desplegó **Homepage** utilizando la funcionalidad de **Stacks** (Docker Compose) dentro de Portainer.

1. En Portainer, navegar a **Stacks** -> **Add stack**.
2. **Name:** `homepage`
3. Pegar la siguiente configuración (se mapea al puerto local `3000`):

```yaml
version: '3.3'
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    restart: unless-stopped
    ports:
      - 3000:3000
    volumes:
      - /opt/homepage/config:/app/config # Directorio en el host para persistencia
      - /var/run/docker.sock:/var/run/docker.sock:ro # Permite leer el estado de los contenedores
```

4. Desplegar el Stack. Los archivos de configuración (`services.yaml`, `widgets.yaml`, etc.) se generan automáticamente en `/opt/homepage/config` del LXC.
5. **Acceso:** `http://192.168.0.61:3000`

---

## 📊 4. Despliegue de Grafana

Para la visualización avanzada de métricas, integración con Zabbix y análisis de logs, se montó el contenedor oficial de **Grafana**. Para evitar conflictos de puertos con Homepage, se expuso externamente en el puerto `3001`.

1. En Portainer, navegar a **Stacks** -> **Add stack**.
2. **Name:** `grafana`
3. Pegar la siguiente configuración:

```yaml
version: '3.3'
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - 3001:3000
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin # Se recomienda cambiar tras el primer inicio

volumes:
  grafana_data:
```

4. Desplegar el Stack.
5. **Acceso:** `http://192.168.0.61:3001`

*(Las configuraciones posteriores de Datasources para conectar Zabbix y Loki se gestionan desde la propia interfaz gráfica de Grafana).*

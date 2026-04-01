# Documentación 05: Despliegue y Configuración de Nextcloud

Este documento detalla el Procedimiento Operativo Estándar (SOP) para la instalación, corrección de errores y configuración de Nextcloud como servidor de almacenamiento privado, integrado con un volumen físico persistente de 2TB.

---

## 1. Arquitectura del Sistema

| Componente | Especificación Técnica |
| :--- | :--- |
| **Hipervisor** | Proxmox VE |
| **Entorno de Ejecución** | LXC 101 (Ubuntu Server / CasaOS) |
| **Plataforma** | Contenedor Docker (Nextcloud v28+) |
| **Endpoint de Red** | `http://192.168.0.51:7580` |
| **Directorio de Montaje Físico** | `/data/` (Mapeado hacia el HDD de 2TB) |
| **Base de Datos** | SQLite (Integrada localmente) |

---

## 2. Solución de Errores de Inicialización (Vía CLI)

Debido a fallos en el auto-aprovisionamiento del contenedor, la configuración inicial y la resolución de bloqueos se ejecutaron directamente en el binario `occ` de Nextcloud a través de la consola del LXC.

**2.1. Autorización de IP (Trusted Domains)**
Para resolver el error "Acceso a través de un dominio del que no se confía":

![Error de Dominio Confiable](../images/Captura%20desde%202026-04-01%2012-23-30.png)

    docker exec -u 33 -it nextcloud php /var/www/html/occ config:system:set trusted_domains 1 --value=192.168.0.51

**2.2. Aprovisionamiento Forzado de Superusuario**
Para resolver la ausencia de credenciales base o base de datos corrupta, se utilizó la terminal de Proxmox:

![Creación de Administrador vía CLI](../images/Captura%20desde%202026-04-01%2012-32-16.png)

    # Comando para crear el usuario administrador (Solicitará contraseña interactiva)
    docker exec -u 33 -it nextcloud php /var/www/html/occ user:add --group="admin" admin

    # Comando alternativo para resetear la contraseña si el usuario ya figuraba en la DB
    docker exec -u 33 -it nextcloud php /var/www/html/occ user:resetpassword admin

**2.3. Activación Manual de Módulos (Bypass de App Store)**
Para resolver la falta de resolución DNS interna que impedía descargar aplicaciones desde la tienda UI, se forzó la activación del plugin de almacenamiento externo:

    docker exec -u 33 -it nextcloud php /var/www/html/occ app:enable files_external

---

## 3. Gestión de Identidades y Cuotas (Vía Web UI)

Una vez estabilizado el acceso, se procedió a configurar la estructura de usuarios desde la interfaz web.

| Usuario (Login) | Nombre Mostrado | Grupo | Cuota de Disco |
| :--- | :--- | :--- | :--- |
| `admin` | Administrador | `admin` | Ilimitado |
| `Bamba` | Bamba | `Familia` | **250 GB** |
| `Sol` | Sol | `Familia` | **100 GB** |
| `Nora` | Nora | `Familia` | **100 GB** |
| `Jorge` | Jorge (Rulo) | `Familia` | **100 GB** |

---

## 4. Mapeo de Almacenamiento Externo (Aislamiento Físico)

Para garantizar la persistencia de datos en el HDD de 2TB, se enlazaron los directorios físicos mediante el módulo *External Storage*. Cada usuario tiene acceso restringido a su propia carpeta para asegurar la privacidad.

![Configuración Final de Almacenamiento Externo](../images/Captura%20desde%202026-04-01%2013-02-18.png)

| Nombre de la Carpeta | Almacenamiento Externo | Configuración (Ruta) | Restrict to (Permisos) |
| :--- | :--- | :--- | :--- |
| Mis Archivos | Local | `/data/Bamba` | Bamba, admin |
| Mis Archivos | Local | `/data/Sol` | Sol, admin |
| Mis Archivos | Local | `/data/Nora` | Nora, admin |
| Mis Archivos | Local | `/data/Jorge` | Jorge, admin |

> **Validación de Integridad:** El sistema confirma la correcta lectura/escritura (RW) mediante el indicador de estado verde en cada montaje.

---

## 5. Configuración de Endpoint Móvil (Clientes Finales)

Para la sincronización de archivos desde los dispositivos móviles, se aplica la siguiente configuración en la aplicación oficial:

1. **Host de Conexión:** `http://192.168.0.51:7580`
2. **Autenticación:** Credenciales individuales generadas en el Paso 3.
3. **Sincronización de Medios (Auto-Upload):**
   * Configurar directorio origen (`DCIM`).
   * **Requisito mandatorio:** Marcar la casilla **"Solo en Wi-Fi"** para prevenir el consumo de datos móviles.

---
**Elaborado por:** Ingeniería de Infraestructura
**Documento:** Tomo 05 - Almacenamiento y Nube Privada

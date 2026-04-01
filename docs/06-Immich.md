# 06 - Servidor de Fotos Immich: Almacenamiento, Actualización y Usuarios

---

## 1. Gestión de Almacenamiento y Permisos

El objetivo principal es evitar que las fotos y videos saturen el almacenamiento interno del contenedor (SSD) y redirigir todo el tráfico de Immich hacia el disco externo de 2TB (`HDD-Backups`).

### 1.1. Problema de Saturación Inicial
Por defecto, Immich comenzó a guardar los archivos en el disco virtual del contenedor, lo que provocó una alerta de falta de espacio en CasaOS casi inmediata.
![Error de falta de espacio en CasaOS](../images/Captura%20desde%202026-04-01%2013-58-06.png)

### 1.2. Problema de Permisos en LXC Unprivileged
Al intentar redirigir los datos al disco de 2TB, el sistema devolvió el error `Operation not permitted`. Esto ocurre porque el usuario de Docker (UID 1000) dentro de un contenedor LXC sin privilegios no tiene autorización para escribir en un disco montado por Proxmox. Esto causó que el contenedor de Immich quedara en estado *Unhealthy* y fallaran las copias de archivos.

### 1.3. Solución: Bind Mount y Permisos Globales
Para solucionar esto, se aplicó una configuración a nivel de hipervisor y un engaño a nivel de sistema de archivos:

1. Se otorgaron permisos totales desde el Host (Proxmox) al disco de 2TB físico para evitar conflictos de mapeo de usuarios. Ejecutado en la consola de Proxmox:

       chmod -R 777 /mnt/pve/HDD-Backups

2. Se realizó un "Bind Mount" persistente editando la configuración del LXC 101 (`/etc/pve/lxc/101.conf`). Esto permite que CasaOS escriba en el disco externo creyendo que es su directorio local:

       mp1: /mnt/pve/HDD-Backups,mp=/DATA/Gallery/immich

3. Se verificó el montaje correcto utilizando el comando `df -h` dentro del contenedor, confirmando que la ruta local ahora dispone de 1.8T libres.
![Comprobación de Almacenamiento en consola](../images/Captura%20desde%202026-04-01%2013-42-53.png)
![Almacenamiento de 1.8T verificado](../images/Captura%20desde%202026-04-01%2014-24-13.png)

---

## 2. Actualización Crítica del Servidor a v2.6.3

Para garantizar la compatibilidad con las aplicaciones móviles actualizadas de los usuarios, fue necesario subir la versión del servidor Immich.

### 2.1. Backup de Seguridad Preventivo
Dado que Proxmox no permite tomar *snapshots* en contenedores con *Mount Points* activos, se respaldó la base de datos y la configuración manualmente mediante consola antes de actualizar:

    cp -r /DATA/AppData/immich /DATA/AppData/immich_backup

### 2.2. Resolución de Error de Tags en Docker
Al intentar forzar la versión en los ajustes de CasaOS utilizando los tags `v2.6.3` o `release`, el motor de Docker arrojó un error de referencia no encontrada (`failed to resolve reference... not found`), ya que los desarrolladores de Immich descontinuaron el uso de esos tags específicos en su repositorio.

**Solución Definitiva:**
Se reemplazó el tag de las imágenes de `immich-server` e `immich-machine-learning` por la etiqueta maestra **`v2`**. Esto instruye a Docker a descargar y compilar automáticamente la última versión estable de la rama 2.x.
![Configuración de Tag v2 en CasaOS](../images/Captura%20desde%202026-04-01%2014-38-11.png)

Tras reiniciar los contenedores, el servidor se actualizó exitosamente y reconoció el espacio de almacenamiento asignado.
![Panel de Immich Actualizado](../images/Captura%20desde%202026-04-01%2013-50-41.png)

---

## 3. Configuración de Usuarios y Cuotas

Para mantener la organización en el sistema de archivos del disco de 2TB y evitar que un solo usuario acapare todo el almacenamiento, se configuraron perfiles individuales en el panel de administración. 

A cada usuario se le asignó una cuota en GiB y un **Storage Label** (Etiqueta de almacenamiento) específico. Esto último obliga a Immich a guardar las fotos de cada persona en una carpeta con su nombre, en lugar de mezclar los archivos.

**Distribución de Cuotas:**
* **Nora:** 500 GiB (Storage Label: `nora`)
* **Sol:** 400 GiB (Storage Label: `sol`)
* **Rulo:** 100 GiB (Storage Label: `rulo`)
* **Bamba:** 100 GiB (Storage Label: `bamba`)

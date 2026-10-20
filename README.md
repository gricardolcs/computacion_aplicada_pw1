# Trabajo Práctico Integrador Grupal - Computación Aplicada

Este repositorio contiene la entrega y documentación técnica correspondiente al **Trabajo Práctico Integrador Grupal** de la asignatura Computación Aplicada / Administración de Sistemas de la **Universidad de Palermo**.

---

##  Integrantes del Grupo
* **Ricardo Gonzalez** ([@gricardolcs](https://github.com/gricardolcs))
* **Axel Geslin** ([@AxelGes](https://github.com/AxelGes))
* **Rodrigo Alberto Leiva**
* **Emiliano Veron**
* **Ismael Martinez**

---

##  Resumen del Proyecto

El proyecto consistió en el despliegue, actualización, configuración y securización de un entorno de servidor basado en **GNU/Linux Debian**, ejecutado sobre una máquina virtual en Oracle VirtualBox.

###  Ficha Técnica del Servidor
* **Hostname:** `TPServer`
* **Sistema Operativo:** Debian 12.15 (Bookworm)
* **IP Estática:** `192.168.1.100/24`
* **Gateway:** `192.168.1.1`
* **Servicios Activos:** SSH, Apache2 (con PHP 8.2 y `php-mysql`), MariaDB Server

---

##  Configuración e Implementación Realizada

### 1. Configuración del Entorno y Sistema Operativo
* **Recuperación y Blanqueo de Root:** Se obtuvo acceso root editando los parámetros de arranque en GRUB (`init=/bin/bash`) y modificando la contraseña del superusuario a `palermo` mediante el comando `passwd`.
* **Hostname:** Se estableció el nombre del sistema como `TPServer` mediante `hostnamectl set-hostname TPServer` y se actualizó `/etc/hosts`:
  ```text
  127.0.0.1	localhost
  127.0.1.1	TPServer
  ```
* **Actualización del SO:** Se ajustó el archivo `/etc/apt/sources.list` apuntando a los repositorios oficiales de **Debian 12 (Bookworm)** y se realizó un upgrade completo del sistema (`apt update && apt full-upgrade`).

### 2. Configuración de Servicios
* **SSH:** Se instaló y configuró `openssh-server`. El usuario `root` accede con la clave pública de la cátedra, cargada en `/root/.ssh/authorized_keys` (permisos 600). Desde la máquina física:
  ```bash
  ssh -i clave_privada.txt root@192.168.1.100
  ```
* **Servidor Web (Apache + PHP):** Se instaló Apache2 con PHP 8.2 (`libapache2-mod-php`) y la extensión `php-mysql`, que `index.php` necesita para conectarse a MariaDB con `mysqli`. Se sirven `index.php` y `logo.png` desde `/www_dir`.
* **Base de Datos (MariaDB):** Se instaló MariaDB Server y se importó el script `db.sql` disponible en el Home de root. Crea la base `ingenieria` con las tablas `alumnos`, `modulos` y `notas`, y el usuario `lcars`.

### 3. Configuración de Red
Se configuró la interfaz `enp0s3` con IP estática en `/etc/network/interfaces`, en el mismo rango que la máquina física:
```text
auto lo
iface lo inet loopback

allow-hotplug enp0s3
iface enp0s3 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
```

### 4. Almacenamiento y Particionamiento
Se añadió un disco virtual de 10 GB (`/dev/sdc`) con dos particiones estándar (tipo 83, ext4):

| Partición | Tamaño | Punto de montaje | Uso |
|---|---|---|---|
| `/dev/sdc1` | 3 GB | `/www_dir` | DocumentRoot de Apache (`index.php` y `logo.png`) |
| `/dev/sdc2` | 6 GB | `/backup_dir` | Destino de los backups |

Ambas se montan automáticamente al iniciar mediante `/etc/fstab`:
```text
UUID=710912be-264f-4f68-a3f6-edce661474f6 /www_dir    ext4   defaults 0 2
UUID=ca72cb78-5efa-44d7-8803-013bf49dbae9 /backup_dir ext4   defaults 0 2
```

Apache apunta a la nueva ubicación con `DocumentRoot /www_dir` en `000-default.conf` y este bloque en `apache2.conf`:
```apache
<Directory /www_dir/>
   Options Indexes FollowSymLinks
   AllowOverride None
   Require all granted
</Directory>
```

El contenido de `/proc/partitions` se guarda en `/opt/particion` en cada arranque (ver crontab más abajo).

### 5. Automatización de Backups
El script `/opt/scripts/backup_full.sh` (permisos 755):
* Recibe el origen y el destino como argumentos: `backup_full.sh <origen> <destino>`.
* Tiene la opción `-help` con las instrucciones de uso.
* Antes de ejecutar, valida que el origen y el destino existan, que el destino sea un sistema de archivos montado (`mountpoint`) y con permisos de escritura, y que el origen esté montado si figura en `/etc/fstab`.
* Genera `<nombre>_bkp_YYYYMMDD.tar.gz` en el destino. Por ejemplo, `/var/log` genera `log_bkp_20240302.tar.gz`.

```bash
/opt/scripts/backup_full.sh -help
/opt/scripts/backup_full.sh /var/log /backup_dir
/opt/scripts/backup_full.sh /www_dir /backup_dir
```

#### Tareas Programadas (crontab de root)
```text
# Carga de reglas de firewall al inicio
@reboot /usr/sbin/iptables-restore /root/myfw.txt

# Backup diario de logs a las 00:00 hs
0 0 * * * /opt/scripts/backup_full.sh /var/log /backup_dir

# Backup del sitio web los lunes, miércoles y viernes a las 23:00 hs
0 23 * * 1,3,5 /opt/scripts/backup_full.sh /www_dir /backup_dir

# Registro del estado de particiones al inicio
@reboot cat /proc/partitions > /opt/particion
```

La consigna menciona `/var/logs`, pero ese directorio no existe en Debian; los logs del sistema están en `/var/log`, que es lo que corresponde al ejemplo `log_bkp_YYYYMMDD.tar.gz`.

---

##  Archivos Incluidos en este Repositorio

| Archivo | Contenido |
|---|---|
| `root.tar.gz` | `/root` |
| `etc.tar.gz` | `/etc` |
| `opt.tar.gz` | `/opt` (incluye `scripts/backup_full.sh` y `particion`) |
| `www_dir.tar.gz` | `/www_dir` |
| `backup_dir.tar.gz` | `/backup_dir` |
| `var.tar.gz.part_00` a `var.tar.gz.part_03` | `/var`, dividido en partes de 20 MB |

Para reconstruir y extraer `/var`:
```bash
cat var.tar.gz.part_* > var.tar.gz
tar -xzf var.tar.gz
```

# Trabajo Práctico Integrador Grupal - Computación Aplicada

Este repositorio contiene la entrega y documentación técnica correspondiente al **Trabajo Práctico Integrador Grupal** de la asignatura Computación Aplicada / Administración de Sistemas de la **Universidad de Palermo**.

---

## 👥 Integrantes del Grupo
* **R. Gonzalez** ([@gricardolcs](https://github.com/gricardolcs))

---

## 📋 Resumen del Proyecto

El proyecto consistió en el despliegue, actualización, configuración y securización de un entorno de servidor basado en **GNU/Linux Debian**, ejecutado sobre una máquina virtual en Oracle VirtualBox. 

### 🧰 Ficha Técnica del Servidor
* **Hostname:** `TPServer`
* **Sistema Operativo:** Debian 12 (Bookworm)
* **IP Estática:** `192.168.1.100/24`
* **Gateway:** `192.168.1.1`
* **Servicios Activos:** SSH, Apache2 (con PHP 8.2), MariaDB Server

---

## 🛠️ Configuración e Implementación Realizada

### 1. Configuración del Entorno y Sistema Operativo
* **Recuperación y Blanqueo de Root:** Se obtuvo acceso root editando los parámetros de arranque en GRUB (`init=/bin/bash`) y modificando la contraseña del superusuario a `palermo` mediante el comando `passwd`.
* **Hostname:** Se estableció el nombre del sistema como `TPServer` mediante `hostnamectl set-hostname TPServer` y se actualizó el archivo `/etc/hosts`.
* **Actualización del SO:** Se ajustó el archivo `/etc/apt/sources.list` apuntando a los repositorios oficiales de **Debian 12 (Bookworm)** y se realizó un upgrade completo del sistema (`apt update && apt full-upgrade`).

### 2. Configuración de Servicios
* **SSH:** Se instaló y configuró `openssh-server`. Se habilitó el acceso al usuario `root` mediante autenticación por clave pública/privada presente en `/root/.ssh/authorized_keys`.
* **Servidor Web (Apache + PHP):** Se instaló Apache2 con soporte para PHP. Se configuraron los archivos `index.php` y `logo.png` para ser servidos por el servidor web.
* **Base de Datos (MariaDB):** Se instaló MariaDB Server y se importó exitosamente el script SQL `db.sql` disponible en el Home de root, estructurando la base de datos de la aplicación (`ingenieria`).

### 3. Configuración de Red
* Se configuró la interfaz de red `enp0s3` con direccionamiento IP estático en `/etc/network/interfaces`:
  ```text
  auto lo
  iface lo inet loopback

  allow-hotplug enp0s3
  iface enp0s3 inet static
      address 192.168.1.100
      netmask 255.255.255.0
      gateway 192.168.1.1

4. Almacenamiento y ParticionamientoSe añadió un segundo disco virtual de 10 GB y se crearon dos particiones estándar (tipo 83):/www_dir (3 GB):
 
<img width="567" height="268" alt="image" src="https://github.com/user-attachments/assets/02806246-aeec-4c48-aa32-74cc58e8d6b2" />


   5. Automatización de Backups
<img width="564" height="235" alt="image" src="https://github.com/user-attachments/assets/57f803c9-82f1-482b-a6b0-b0229c474d1b" />

     
Tareas Programadas (crontab)Bash# Carga de reglas de firewall al inicio
@reboot /usr/sbin/iptables-restore /root/myfw.txt

# Backup diario de logs a las 00:00 hs
0 0 * * * /opt/scripts/backup_full.sh /var/logs /backup_dir

# Backup del sitio Web los días Lunes, Miércoles y Viernes a las 23:00 hs
0 23 * * 1,3,5 /opt/scripts/backup_full.sh /www_dir /backup_dir

# Registro de estado de particiones al inicio
@reboot cat /proc/partitions > /opt/particion
📁 Archivos Incluidos en este RepositorioEn cumplimiento con las consignas del trabajo práctico, se adjuntan los siguientes paquetes comprimidos con la estructura del servidor:
 
<img width="623" height="412" alt="image" src="https://github.com/user-attachments/assets/4edfcc43-be71-43a0-8e37-f55343b58227" />


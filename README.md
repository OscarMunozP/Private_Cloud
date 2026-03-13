# Private_Cloud
Nube privada corporativa basada en Nextcloud integrada con Active Directory (Windows Server 2019) para autenticación centralizada. Incluye scripts de copia de seguridad con Restic, automatización mediante cron y documentación técnica para despliegue, mantenimiento y restauración del sistema.

# Arquitectura del sistema
La infraestructura se compone de dos servidores independientes.
### El primero es un Ubuntu Server, en el que se alojara el Nextcloud. Este servidor necesitará las siguientes funciones:
- Apache2
- PHP
- MariaDB
- Nextcloud
- Restic
- Integración con Active Directory mediante LDAP
### El segundo servidor será un Windows Server 2019, con las diguientes características:
- Active Directory Domain Services (AD DS)
- DNS integrado
- Gestión de usuarios y grupos
- Políticas de acceso centralizadas
#### La autenticación de Nextcloud se realiza contra el dominio de Windows mediante LDAP.

# Requisitos del sistema
**Hardware recomendado**
- Ubuntu Server: 4 CPU, 8 GB RAM, 60 GB almacenamiento
- Windows Server 2019: 4 CPU, 8 GB RAM, 60 GB almacenamiento

**Software**
- Ubuntu para Nextcloud
- Windows Server 2019 con AD DS
- Apache2, PHP, MariaDB
- Restic
- Cron

# Preparación de Ubuntu para el servidor Nextcloud
Primero instalamos Apache, PHP y MariaDB
```bash 
sudo apt install apache2 mariadb-server php php-mysql php-xml php-zip php-curl php-gd php-mbstring php-intl php-bz2 php-imagick -y
```
Después creamos la base de datos para Nextcloud
```bash
CREATE DATABASE nextcloud;
CREATE USER 'user'@'localhost' IDENTIFIED BY 'contraseña';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'user'@'localhost';
FLUSH PRIVILEGES;
```
Finalmente instalamos Nextcloud <br>
Hay que descargar, descomprimir y mover a /var/www/nextcloud.

# Instalación y configuración del servidor Windows Server 2019
### Instalación de AD DS
1. Abrir Server Manager
2. Añadir roles → Active Directory Domain Services
3. Promocionar el servidor a Controlador de Dominio
4. Crear el dominio corporativo (ejemplo: empresa.local)

### Creación de usuarios y grupos
En Active Directory Users and Computers:

Crear OU “Usuarios”

Crear OU “Grupos”

Crear usuarios corporativos

Asignar grupos según roles

### Configuración de DNS

El servidor AD DS configura DNS automáticamente.

# Integración de Nextcloud con Active Directory
 Entramos en el Nextcloud a traves de la cuenta administrador, nos dirigimos hacia ajustes y entramos en Integración LDAP.

 Configurar:
- Servidor LDAP: IP del Windows Server
- Puerto: 389 (LDAP) o 636 (LDAPS)
- DN base: DC=empresa,DC=local
- DN de usuario de consulta (si se usa)
- Filtro de usuarios: (objectClass=user)

Terminado esto, probamos conexión y guardamos. Los usuarios del dominio deberían aparecer automaticamente en Nextcloud.

# Sistema de copias de seguridad con Restic
### Inicialización del repositorio
```bash
restic init --repo /ruta/repositorio
```
### Script de backup (backup.sh)
```bash
#!/bin/bash
export RESTIC_PASSWORD="contraseña"
restic --repo /ruta/repositorio backup /var/www/nextcloud
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6
restic prune
```
Permisos:
```bash
chmod +x backup.sh
```
# Automatización con cron
```bash
crontab -e
```
Añadir a la hora que se quiera hacer la backup:

```bash
0 3 * * * /ruta/backup.sh
```
# Restauración de copias de seguridad
### Listar snapshots
```bash
restic snapshots
```
### Restaurar
```bash
restic restore ID --target /ruta/restauracion
```
# Conclusión

Aunque Git no se utilizó como herramienta de control de versiones durante el desarrollo, este repositorio se entrega como contenedor final del proyecto, incluyendo scripts, configuraciones y documentación técnica. El README proporciona todos los pasos necesarios para desplegar, integrar y mantener el sistema.

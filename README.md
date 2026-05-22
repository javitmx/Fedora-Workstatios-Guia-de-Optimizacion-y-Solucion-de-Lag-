# Fedora-Workstatios-Guia-de-Optimizacion-y-Solucion-de-Lag-
Esta guía contiene los pasos técnicos para solucionar micro-congelamientos en la terminal/interfaz bajo Fedora (relacionados con Btrfs y servicios del sistema) y cómo acelerar drásticamente el gestor de paquetes DNF.
## 1. Solución a Micro-congelamientos (systemd-tmpfiles-clean)

### Problema
El servicio `systemd-tmpfiles-clean` genera picos masivos de operaciones de lectura/escritura (I/O Bottleneck) al escanear directorios temporales, lo que causa retrasos temporales donde los comandos se "pegan" y reaccionan de golpe (especialmente notable en sistemas de archivos Btrfs).

### Solución
Dado que es una unidad estática, la forma definitiva de desactivarla y mitigar el impacto en el rendimiento del disco es enmascarando el temporizador:

```bash
# Detener el temporizador en la sesión actual
sudo systemctl stop systemd-tmpfiles-clean.timer

# Enmascarar para evitar que el sistema lo invoque automáticamente
sudo systemctl mask systemd-tmpfiles-clean.timer
``` 
## 2. Solucion de la Base de Datos RPM (INterrupcion de DNF)
Si el gestor de paquetes se interrumpe abruptamente (ej. Ctrl + C), la base de datos de paquetes (rpmdb) puede quedar bloqueada o corrupta en un estado de sueño ininterrumpible (D State). Para repararla de forma segura tras un reinicio, se ejecutan:

# 1. Eliminar bloqueos fantasmas de memoria
``bash
sudo rm -f /var/lib/rpm/.rpmdb.lock /var/lib/dnf/lock
``
# 2. Reconstruir el índice de la base de datos RPM
``bash
sudo rpm --rebuilddb
``
# 3. Sincronizar y reparar paquetes instalados a medias
``bash
sudo dnf distro-sync -y 
``
## 3. Optimizacion Extrema de DNF (Descargas 5x mas rapidas)
Por defecto, DNF descarga los paquetes de uno en uno y actualiza constantemente los metadatos desde la red.

# Paso 1:Configuración del DNF Clásico
Editar el archivo de configuración en /etc/dnf/dnf.conf e inclu[main]

# Habilitar descargas simultáneas (máximo 10)
``bash
max_parallel_downloads=10
``
# Habilitar Delta RPMs (descarga solo diferencias de código)
``bash
deltarpm=True
``
# Cambiar la confirmación por defecto a "Sí"
``bash
defaultyes=True
``
# Evitar la descarga constante de metadatos (expira cada 24 horas)
``bash
metadata_expire=86400
``
# Limpiar el cache para aplicar los cambios: 
``bash
sudo dnf clean all
``
# paso 2: Migracion a DNF5 (EL motor ultra veloz)
Para maximizar la velocidad de procesamiento eliminando el delay de Python, instalamos y utilizamos el nuevo motor escrito en C++:

# Instalar el nuevo motor
``bash
sudo dnf install dnf5 -y
``
# Comando de actualización definitivo a partir de ahora
``bash
sudo dnf5 upgrade -y
``

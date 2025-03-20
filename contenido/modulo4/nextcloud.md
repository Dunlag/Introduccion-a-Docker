# Ejemplo 1: Contenedor NextCloud con almacenamiento persistente

[NextCloud](https://nextcloud.com/es/) es una aplicación escrita en PHP que nos permite construir una nube privada para guardar nuestros archivos, además de ofrecer otros servicios como agenda, calendario, etc.

Vamos a desplegar un contenedor con NextCloud. Para simplificar la instalación, usaremos una base de datos SQLite. Según la documentación de la imagen [`nextcloud`](https://hub.docker.com/_/nextcloud) en Docker Hub, la forma más sencilla de no perder la información es crear un volumen para almacenar el directorio `/var/www/html` del contenedor. 

Realizaremos el ejercicio usando tanto **volúmenes Docker** como **bind mount**.

---

## 📌 Ejemplo con volúmenes

### 1️⃣ Crear un volumen

```bash
$ docker volume create nextcloud
nextcloud
```

### 2️⃣ Crear el contenedor con volumen

Ejecutamos el contenedor asignando el volumen creado al directorio `/var/www/html` dentro del contenedor:

```bash
$ docker run -d -p 80:80 -v nextcloud:/var/www/html --name contenedor_nextcloud nextcloud:28.0.1
```

### 3️⃣ Comprobar la persistencia de datos

1. Accedemos a la aplicación y completamos la configuración inicial.
2. Subimos un archivo a NextCloud.
3. Eliminamos el contenedor y lo volvemos a crear con el mismo volumen, pero ahora usando `--mount`:

```bash
$ docker rm -f contenedor_nextcloud

$ docker run -d -p 80:80 --mount type=volume,src=nextcloud,dst=/var/www/html --name contenedor_nextcloud nextcloud:28.0.1
```

✅ **Al acceder nuevamente a la aplicación, veremos que la configuración y los archivos subidos se mantienen.**

---

## 📌 Ejemplo con bind mount

### 1️⃣ Crear un directorio en el host

```bash
mkdir /opt/datos_nextcloud
```

### 2️⃣ Crear el contenedor usando `-v`

```bash
$ docker run -d -p 80:80 -v /opt/datos_nextcloud:/var/www/html --name contenedor_nextcloud nextcloud:28.0.1
```

También podemos usar `--mount`:

```bash
$ docker run -d -p 80:80 --mount type=bind,src=/opt/datos_nextcloud,dst=/var/www/html --name contenedor_nextcloud nextcloud:28.0.1
```

### 3️⃣ Comprobar la persistencia de datos

Accedemos a la aplicación, configuramos y subimos un archivo. Luego, verificamos que los archivos están disponibles en el host:

```bash
$ cd /opt/datos_nextcloud/
$ ls
3rdparty  COPYING  config  core  custom_apps  index.html  lib  ocm-provider  ocs-provider  remote.php  robots.txt  themes
```

✅ **Si eliminamos y recreamos el contenedor usando el mismo bind mount, la configuración y los archivos subidos no se perderán.**

---

## 📌 Ejemplo con Windows (WSL2)

Para sistemas Windows con WSL2, utilizamos la ruta adaptada para acceder a las unidades de Windows:

```bash
docker run -d -p 8080:80 -v /mnt/c/RUTA_DE_TU_ORDENADOR_HOST/datos_nextcloud:/var/www/html/data nextcloud
```

🔹 **Explicación:**
- `/mnt/c/RUTA_DE_TU_ORDENADOR_HOST/datos_nextcloud`: Reemplazar con la ruta real en el host.
- `-d`: Ejecuta en segundo plano.
- `-p 8080:80`: Mapea el puerto 8080 del host al 80 del contenedor.
- `/var/www/html/data`: Carpeta donde NextCloud almacena los datos.
- `nextcloud`: Imagen de Docker.

✅ **Este formato permite a otros usuarios adaptar fácilmente el comando a su propia configuración.**


# Despliegue de Base de Datos con Docker

Docker es una herramienta de código abierto que permite empaquetar aplicaciones dentro de contenedores, lo que facilita su despliegue, escalabilidad y administración. En esta guía, te enseñaremos cómo crear un servicio web con una base de datos MySQL utilizando Docker, asegurando que tanto la base de datos como la aplicación web estén correctamente configuradas y comunicadas entre sí.

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Prerrequisitos](#prerrequisitos)
3. [Creación de la Base de Datos](#creación-de-la-base-de-datos)
   - [Crear una Red Docker](#crear-una-red-docker)
   - [Crear un Volumen Docker](#crear-un-volumen-docker)
   - [Crear el Contenedor de MySQL](#crear-el-contenedor-de-mysql)
   - [Acceder al Contenedor de MySQL](#acceder-al-contenedor-de-mysql)
   - [Conceder Privilegios a la Base de Datos](#conceder-privilegios-a-la-base-de-datos)
4. [Desplegar la Aplicación Web](#desplegar-la-aplicación-web)
   - [Crear el Contenedor de Red (Netshoot)](#crear-el-contenedor-de-red-netshoot)
   - [Lanzar el Contenedor de la Web Service](#lanzar-el-contenedor-de-la-web-service)
5. [Verificación y Pruebas](#verificación-y-pruebas)
   - [Verificar los Registros del Contenedor](#verificar-los-registros-del-contenedor)
   - [Acceder a la Aplicación Web](#acceder-a-la-aplicación-web)
   - [Verificar los Datos en la Base de Datos](#verificar-los-datos-en-la-base-de-datos)
6. [Conclusión](#conclusión)

---

## Introducción

El uso de Docker en el despliegue de aplicaciones permite a los desarrolladores automatizar la configuración, el despliegue y la gestión de las dependencias. En esta guía, vamos a crear una infraestructura básica para una aplicación web que se conecta a una base de datos MySQL, todo dentro de contenedores Docker, lo que nos asegura una solución escalable y fácilmente replicable.

---

## Prerrequisitos

Antes de comenzar, asegúrate de tener instalados los siguientes elementos en tu máquina:

- Docker y Docker Compose (para gestionar contenedores de manera eficiente).
- Acceso a la terminal o línea de comandos de tu sistema operativo.
- Conocimientos básicos sobre contenedores y redes.

---

## Creación de la Base de Datos

### Crear una Red Docker

En primer lugar, necesitamos crear una red personalizada para asegurar que nuestros contenedores puedan comunicarse entre sí. Esto es fundamental para la interacción entre la base de datos y la aplicación web.

```bash
docker network create mateo-todo-app
```

Para verificar que la red se ha creado correctamente, ejecuta el siguiente comando:

```bash
docker network ls
```

Esto debería mostrar todas las redes activas en tu sistema Docker, incluida la `mateo-todo-app`.

### Crear un Volumen Docker

El siguiente paso es crear un volumen de Docker que persistirá los datos de la base de datos MySQL. Este volumen estará vinculado entre la máquina virtual y los contenedores.

```bash
docker volume create mateo-todo-app-sql
```

### Crear el Contenedor de MySQL

Ahora, creamos el contenedor de la base de datos MySQL con los parámetros necesarios, tales como la contraseña de root y el nombre de la base de datos:

```bash
docker run -d --network mateo-todo-app --network-alias mysql -v mateo-todo-app-sql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:5.7
```

Este comando lanzará un contenedor MySQL con la versión 5.7 y la base de datos `todos`. La contraseña para el usuario root es `secret`. Para verificar que el contenedor está en funcionamiento, puedes utilizar:

```bash
docker ps -a
```

### Acceder al Contenedor de MySQL

Para acceder al contenedor de MySQL y verificar que la base de datos se ha creado correctamente, ejecuta:

```bash
docker exec -it ce0dc79ce27f mysql -u root -p
```

Cuando se te pida la contraseña, ingresa `secret`. Luego, dentro de MySQL, ejecuta:

```sql
SHOW DATABASES;
```

Esto te mostrará las bases de datos disponibles, incluida la base de datos `todos`. Sal del contenedor con el comando `exit`.

### Conceder Privilegios a la Base de Datos

Para asegurarte de que la aplicación web puede interactuar correctamente con la base de datos, es necesario conceder los privilegios adecuados al usuario root:

```bash
docker exec -it ce0dc79ce27f mysql -u root -p
ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret';
FLUSH PRIVILEGES;
EXIT;
```

---

## Desplegar la Aplicación Web

### Crear el Contenedor de Red (Netshoot)

Para gestionar las herramientas de red y facilitar la depuración, utilizaremos un contenedor adicional con la imagen `nicolaka/netshoot`. Este contenedor proporciona herramientas útiles como `dig`, `nslookup`, entre otras.

```bash
docker run -it --network mateo-todo-app nicolaka/netshoot
```

Dentro del contenedor, puedes probar la conectividad con MySQL usando:

```bash
dig mysql
```

Esto verificará que la red está configurada correctamente. Luego, sal del contenedor con el comando `exit`.

### Lanzar el Contenedor de la Web Service

Ahora lanzamos el contenedor que ejecutará nuestra aplicación web. Este contenedor estará basado en Node.js y se conectará a la base de datos MySQL para interactuar con ella.

```bash
sudo apt update
git clone https://github.com/docker/getting-started.git
```

A continuación, ejecuta el siguiente comando para iniciar la web service en el puerto 3000:

```bash
docker run -dp 3000:3000 -w /app -v "/home/ubuntu/getting-started/app/:/app" --network mateo-todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine sh -c "yarn install && yarn run dev"
```

Este comando realiza lo siguiente:

- Lanza la aplicación web en el puerto 3000.
- Vincula los archivos del directorio local con el contenedor.
- Conecta el contenedor a la red Docker.
- Proporciona las credenciales de la base de datos MySQL para que la aplicación pueda interactuar con ella.

---

## Verificación y Pruebas

### Verificar los Registros del Contenedor

Puedes revisar los registros generados por el contenedor con el siguiente comando. Esto es útil para depurar problemas si la aplicación no está funcionando correctamente.

```bash
docker logs ID_CONTAINER
```

Reemplaza `ID_CONTAINER` con el ID del contenedor de la aplicación web.

### Acceder a la Aplicación Web

Una vez que el contenedor de la aplicación esté en funcionamiento, puedes acceder a la aplicación web a través de tu navegador. Solo tienes que usar la IP pública de la máquina virtual y el puerto 3000:

```bash
http://IP_PUBLICA:3000
```

Desde la interfaz de la aplicación, podrás interactuar con la base de datos y almacenar información en ella.

### Verificar los Datos en la Base de Datos

Si deseas verificar los datos almacenados en la base de datos desde dentro del contenedor MySQL, utiliza el siguiente comando:

```bash
docker exec -it $MYSQL_ID mysql -u root -p
SELECT * FROM todo_items;
```

Esto mostrará todos los elementos guardados en la tabla `todo_items`.

---

## Conclusión

Con estos pasos, has desplegado con éxito una aplicación web que interactúa con una base de datos MySQL utilizando Docker. Este enfoque asegura que tu aplicación sea fácilmente escalable, portable y que pueda ser replicada en cualquier entorno que soporte Docker. Docker no solo simplifica la configuración del entorno, sino que también facilita el manejo de dependencias, el despliegue y la administración de tu infraestructura.

Para proyectos más grandes, puedes considerar el uso de Docker Compose para gestionar múltiples contenedores de forma más eficiente.

---

Gracias por seguir esta guía. ¡Feliz desarrollo con Docker!

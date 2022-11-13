# Despliegue de base de datos con Docker
Docker es una herramienta opensource en la cual se puede empaquetar aplicaciones dentro de contenedores, en este guia se mostrara la creacion de una web services con una base de datos en Mysql utilizando Docker
## Creación de base de datos paso a paso
En primer lugar se agregara una red ligada al nombre de mateo-todo-app

```bash
Docker network create mateo-todo-app
```
Ya con la red creada se procede a revisar si esta si esta desplegada

```bash
Docker network ls
```
se podra observar todas las redes desplegadas por docker, en la que se encontrara la mensionada mateo-todo-app, ahora se creara un volumen con el cual estaran unidos los archivos fisicos de la maquina virtual y los contenedores

```bash
Docker volume create mateo-todo-app-sql
```
Ahora se creara el contenedor de la base de datos con mysql de la sigueinte forma

```bash
docker run -d --network mateo-todo-app --network-alias mysql -v mateo-todo-app-sql:/var/lib/msql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:5.7
```
Se puede evidenciar la contraseña para acceder a la base de datos `MYSQL_ROOT_PASSWORD=secret`, tambien el nombre `MYSQL_DATABASE=todos`, y la version de mysql que es la 5.7, se revisara con `docker ps -a` si el contener esta arriba, despues se procedera a acceder a al conteneder de la base de datos

```bash
docker exec -it ce0dc79ce27f mysql -u root -p
```
Donde ce0dc79ce27f es el ID del contenedor, nos pedira la contraseña que en este caso es secret, dentro se escribira la siguiente linea de codigo `show databases;` y exit con el fin de corroborar si la base de datos esta en funcionamiento. 

Ahora se procede a crear el contenedor para netshoot, este trae varias herramientes de red que nos permitira administrar de una manera mas optima nuestro servicio y de igual manera evitar problemas

```bash
docker run -it network mateo-todo-app nicolaka/netshoot
```
Dentro del contenedor se escribira `dig mysql` y exit, con esto ya tendremos las herramientas de red necesarias para desplegar la web services, pero antes se accedera a la base de datos para conceder privilegios a esta

```bash
docker exec -it ce0dc79ce27f mysql -u root -p
ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret’;
flush privileges;
exit
```
Y por ultimo se lanzara el contenedor de la web services con el siguiente comando 

```bash
sudo apt update
git clone https://github.com/docker/getting-started.git 

docker run -dp 3000:3000 -w /app -v "/home/ubuntu/getting-started/app/:/app" --network mateo-todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine sh -c "yarn install && yarn run dev"

```

En el comande de docker run le estamos diciendo que se cree un contenedor sobre el puerto 3000 y ademas de que cree un espacio fisico en el contenedor que estara ligado con de la maquina virtual en este caso nuestro host, el volume, que haga uso de los recursos de red, ademas se le dan los datos de la base de datos que ese contenedor este sobre node:12 version alpine y ejecuto los comandos de `yarn install` y `yarn run dev`

Se mirara los registros generados por ese contenedor con el siguente comando

```bash
docker logs ID_CONTAINER
```

Por ultimo accederemos a la pagina con la ip publica de la maquina virtual en conjunto con el puerto donde se lanzo la web service que para este caso es 3000 `http://IP_PUBLICA:3000`

La pagina genera tiene la funcion de guardar lo que se digite en la base de datos, por lo cual si se quiere revisar que esta almacendo dentro del contenedor se accede a este y se digita lo siguiente

```bash
docker exec –it $MYSQL_ID mysql –u root –p
select*from todo_items;
```


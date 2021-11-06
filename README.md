
# Manual de Configuración - Proyecto

- Laboratorio Bases de Datos 2
- Proyecto 2
- Grupo 11

## Entorno de Desarrollo

### Recusos del sistema

* **Sistema Operativo:** ubuntu 20.04 LTS
* **Memoria RAM:** 8 GB
* **Proveedor:** Google Cloud Platform (E2)

## Requerimientos Previos
- instalación de Docker
- Instalación de Docker-Compose
- Tener acceso a Mongo Compass

## Arquitectura
![1](https://user-images.githubusercontent.com/78786478/140566038-509406ea-dc80-47e8-b343-3e88d4441175.png)



## Procedimiento


### Creación de carpetas

```
mkdir mongodb1/data
mkdir mongodb2/data
mkdir mongodb3/data
mkdir mongodb4/data
mkdir mongodb5/data

```

### Imágenes
Descarga de las imágenes para los contenedores
![2](https://user-images.githubusercontent.com/78786478/140566040-06d2bd29-d161-4f50-a36b-9ea37d19c0d6.png)

```
docker pull mongo:4
```
![3](https://user-images.githubusercontent.com/78786478/140566025-686f760c-8547-47ff-b967-06a76d5417d9.png)

### Se crea la red para los contenedores

```
docker network create mongo-g11-cluster

```
![4](https://user-images.githubusercontent.com/78786478/140566026-8191b003-fe74-4691-b130-10724162d264.png)

### Levantar los contenedores:
Levantamos los contenedores y  a darles un lugar en nuestro volumen

El contenedor número uno sera nuestro maestro, y los demás del 2 al 5 serán los esclavos.

```
docker run -d -v ~/mongodb1/data:/data/db --net mongo-cluster-dev -p 27017:27017 --name mdb1 mongo:4 mongod --replSet mongodb-replicaset --port 27017
docker run -d -v ~/mongodb2/data:/data/db --net mongo-cluster-dev -p 27018:27018 --name mdb2 mongo:4 mongod --replSet mongodb-replicaset --port 27018
docker run -d -v ~/mongodb3/data:/data/db --net mongo-cluster-dev -p 27019:27019 --name mdb3 mongo:4 mongod --replSet mongodb-replicaset --port 27019
docker run -d -v ~/mongodb4/data:/data/db --net mongo-cluster-dev -p 27020:27020 --name mdb4 mongo:4 mongod --replSet mongodb-replicaset --port 27020
docker run -d -v ~/mongodb5/data:/data/db --net mongo-cluster-dev -p 27021:27021 --name mdb5 mongo:4 mongod --replSet mongodb-replicaset --port 27021

```
![5](https://user-images.githubusercontent.com/78786478/140566028-b738ec01-ddef-47e3-8b24-872526fb4496.png)
![6](https://user-images.githubusercontent.com/78786478/140566029-1f998d0b-e08f-43d6-9a29-d4413fa6e3d2.png)

### Se abre el archivo con nano
nano /etc/hosts

### Los agregamos en los hosts de nuestro servidor

Se agrega lo siguiente:
127.0.0.1       mdb1 mdb2 mdb3 mdb4 mdb5

![7](https://user-images.githubusercontent.com/78786478/140566032-f3ade9dd-de12-4461-897d-cda86c1795f3.png)


#### Editamos en replication 
replication:
    replSetName: "red0"

# Relacionarlos para crear el set de réplica, esto solo lo colocamos en el nodo master



### Acceder al mongo maestro directamente a mongo

docker exec -it mdb1 mongo


### Creamos la base de datos y configuramos el cluster añadiendo los esclavos


db = (new Mongo('localhost:27017')).getDB('DBG11')
config={"_id":"mongodb-replicaset","members":[{"_id":0,"host":"mdb1:27017"},{"_id":1,"host":"mdb2:27018"},{"_id":2,"host":"mdb3:27019"},{"_id":3,"host":"mdb4:27020"},{"_id":4,"host":"mdb5:27021"}]}
rs.initiate(config)

![8](https://user-images.githubusercontent.com/78786478/140566034-26591d20-9b49-4379-97c0-7418af7e8de5.png)




### Configuramos la prioridad para darle máxima al maestro y luego al 5to contenedor que es esclavo 
La prioridad es descendente para los esclavos
cfg = rs.conf();
cfg.members[0].priority = 5;
cfg.members[1].priority = 1;
cfg.members[2].priority = 2;
cfg.members[3].priority = 3;
cfg.members[4].priority = 4;
rs.reconfig(cfg);


![9](https://user-images.githubusercontent.com/78786478/140566035-009f5447-f02d-43c7-abb9-cf0ded5445de.png)


### Para conectarnos esta es la cadena 
mongodb://localhost:27017,localhost:27018,localhost:27019,localhost:27020,localhost:27021/DBG11?replicaSet=mongodb-replicaset

![10](https://user-images.githubusercontent.com/78786478/140566037-744891a8-0a1f-4948-825b-cbc3f34bf4b9.png)


### Instalamos compass
https://docs.mongodb.com/compass/current/install/

Agregamos la data desde mongo compass

#### Country
![F3](https://user-images.githubusercontent.com/6895508/140565968-32e027a9-aa75-4caf-9b3d-6909a35ac234.png)


#### Patents
![F2](https://user-images.githubusercontent.com/6895508/140565973-1a240522-2830-4008-8a1f-c88bbdf37930.png)



### Hacer un Fullbackup
![F1](https://user-images.githubusercontent.com/6895508/140565972-832d32c3-09f0-4f82-b06e-a85086b6e7c7.png)



Creando el backup y restaurando dentro del mismo contenedor

Creamos el backup
docker exec -i mdb1 mongodump --out /dumpits

Si es en secundario debemos colocar el puerto

Restauramos el backup
sudo docker exec -i 7f61 mongorestore --uri="mongodb://localhost:27017,localhost:27018,localhost:27019,localhost:27020,localhost:27021/DBG11?replicaSet=mongodb-replicaset" --db DBG11 /dumpits/DBG11


Creando el backup y restaurando desde el archivo en la máquina anfitrión
docker exec -i mdb1 mongodump --out /dumpits

Copiamos el volcado del contenedor a la maquina anfitrión
docker cp 7f61:/dumpits /home/doer/Desktop/LabBD2-Proyecto/dumpits

Copiamos del anfitrión a un contenedor habilitado
docker cp /Desktop/LabBD2-Proyecto/dumpits 4b25:/dumpits 

Ejecutamos el docker obteniendo
sudo docker exec -i 4b25 mongorestore --uri="mongodb://localhost:27017,localhost:27018,localhost:27019,localhost:27020,localhost:27021/DBG11?replicaSet=mongodb-replicaset" --db DBG11 /dumpits/DBG11


### Restaurarlo

docker exec -i 4b25 mongorestore --db="DBG11" ~/Desktop/LabBD2-Proyecto/~/

### MongoImport
mongoimport --uri mongodb://localhost:27017,localhost:27018,localhost:27019,localhost:27020,localhost:27021/DBG11?replicaSet=mongodb-replicaset --type json --db DBG11 --collection patents --file /jsons/patents.json --jsonArray

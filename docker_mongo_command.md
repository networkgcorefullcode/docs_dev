# Comandos útiles para interactuar con MongoDB en Docker

## Listar contenedores activos

```bash
docker ps
```

## Acceder al contenedor de MongoDB

```bash
docker exec -it <container_name> bash
```

Reemplaza `<container_name>` con el nombre o ID del contenedor de MongoDB.

## Abrir el cliente MongoDB dentro del contenedor

```bash
mongo
```

## Conectar a una base de datos específica

```bash
use <database_name>
```

Reemplaza `<database_name>` con el nombre de la base de datos que deseas usar.

## Listar las bases de datos disponibles

```bash
show dbs
```

## Listar las colecciones dentro de una base de datos

```bash
show collections
```

## Insertar un documento en una colección

```bash
db.<collection_name>.insert({key: "value"})
```

Reemplaza `<collection_name>` con el nombre de la colección.

## Buscar documentos en una colección

```bash
db.<collection_name>.find()
```

## Actualizar documentos en una colección

```bash
db.<collection_name>.update({key: "value"}, {$set: {key2: "new_value"}})
```

## Eliminar documentos de una colección

```bash
db.<collection_name>.remove({key: "value"})
```

## Exportar una base de datos desde el contenedor

```bash
docker exec <container_name> mongodump --db <database_name> --out /path/to/backup
```

## Importar una base de datos al contenedor

```bash
docker exec <container_name> mongorestore /path/to/backup
```

## Verificar el estado del contenedor

```bash
docker inspect <container_name>
```

## Reiniciar el contenedor de MongoDB

```bash
docker restart <container_name>
```

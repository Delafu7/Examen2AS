# Examen2AS


## Índice del repositorio

- [Dockerfile](#dockerfile)
  - [Estructura básica de un Dockerfile](#estructura-básica-de-un-dockerfile)
  - [Instrucciones más importantes de Dockerfile](#instrucciones-más-importantes-de-dockerfile)
  - [Comandos básicos relacionados](#comandos-básicos-relacionados)
- [Docker Compose](#docker-compose)
  - [Comandos básicos de Docker Compose](#comandos-básicos-de-docker-compose)
  - [Ejemplo típico de docker-compose.yml](#ejemplo-típico-de-docker-composeyml)
- [Kubernetes](#kubernetes)
  - [Ejemplo típico: Deployment + Service](#ejemplo-típico-deployment--service)
  - [Explicación de los elementos clave](#explicación-de-los-elementos-clave)
  - [Comandos útiles](#comandos-útiles)
  - [Resumen](#resumen)
- [CI/CD](#cicd)
- [Cloud Computing](#cloud-computing)
- [Elasticsearch](#elasticsearch)
- [Pila ELK](#pila-elk)
- [Ejercicios de prueba examen](#ejercicios-de-prueba-examen)
## Dockerfile

### Estructura básica de un Dockerfile
 ```
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]

```

### Instrucciones más importantes de Dockerfile

| Instrucción | Descripción | Ejemplo |
|------------|------------|---------|
| `FROM` | Define la imagen base sobre la que se construye la imagen | `FROM node:18-alpine` |
| `WORKDIR` | Establece el directorio de trabajo dentro del contenedor | `WORKDIR /app` |
| `COPY` | Copia archivos del host al contenedor | `COPY . .` | 
| `ADD` | Copia archivos y permite descomprimir o descargar | `ADD file.tar.gz /app` |
| `RUN` | Ejecuta comandos durante el build | `RUN npm install` | 
| `EXPOSE` | Documenta el puerto usado por la app | `EXPOSE 3000` | 
| `CMD` | Comando por defecto al iniciar el contenedor | `CMD ["npm", "start"]` | 
| `ENTRYPOINT` | Comando fijo al iniciar el contenedor | `ENTRYPOINT ["nginx"]` |
| `ENV` | Define variables de entorno | `ENV NODE_ENV=production` | 
| `ARG` | Variables solo disponibles en build | `ARG VERSION=1.0` |

### Comandos básicos relacionados

- Construir imagen:
```
docker build -t myapp .
```

- Ejecutar en un puerto en concreto, en este caso en el 3000:

```
docker run -p 3000:3000 myapp
```
- Mirar logs de un contenedor concreto:

```
docker logs <container_id>
```

- Entrar al contenedor:

```
docker exec -it <container_id> sh
```

- Ver contendores:
```
docker ps -a
```
- Eliminar contenedor:

```
docker rm <id-o-nombre-contenedor>
```
- Parar contenedor:

```
docker stop <id-o-nombre-contenedor>
```
## Docker compose


### Comandos básicos de Docker Compose

- Levantar servicios:

```
docker compose up
docker compose up -d
```
- Parar y eliminar contenedores:

```
docker compose down
```

- Ver estado de los servicios:

```
docker compose ps
```

- Ver logs:

```
docker compose logs
docker compose logs -f
```
- Reconstruir imágenes:

```
docker compose build
docker compose up --build
```

### Ejemplo típico de `docker-compose.yml`


```yaml

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - db

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: exampledb
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:

```

| Elemento      | Ubicación      | Descripción                              |
| ------------- | -------------- | ---------------------------------------- |
| `version`     | Raíz           | Versión del esquema de Docker Compose    |
| `services`    | Raíz           | Define los contenedores de la aplicación |
| `web`         | `services`     | Servicio de aplicación web               |
| `image`       | `services.web` | Imagen Docker a utilizar                 |
| `ports`       | `services.web` | Mapea puertos host:contenedor            |
| `depends_on`  | `services.web` | Define dependencia con otro servicio     |
| `db`          | `services`     | Servicio de base de datos                |
| `environment` | `services.db`  | Variables de entorno del contenedor      |
| `volumes`     | `services.db`  | Volumen montado en el contenedor         |
| `volumes`     | Raíz           | Definición de volúmenes nombrados        |




## Kubernettes

### Ejemplo típico: Deployment + Service

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:alpine
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080

```

### Explicación de los elementos clave

| Elemento                        | Ubicación  | Descripción                                                |
| ------------------------------- | ---------- | ---------------------------------------------------------- |
| `apiVersion`                    | Raíz       | Versión de la API de Kubernetes                            |
| `kind`                          | Raíz       | Tipo de objeto (`Deployment`, `Service`, etc.)             |
| `metadata.name`                 | Raíz       | Nombre del objeto en el clúster                            |
| `spec.replicas`                 | Deployment | Número de pods que se deben ejecutar                       |
| `spec.selector.matchLabels`     | Deployment | Define qué pods controla el Deployment                     |
| `spec.template.metadata.labels` | Deployment | Etiquetas de los pods creados                              |
| `spec.template.spec.containers` | Deployment | Lista de contenedores dentro del pod                       |
| `image`                         | Container  | Imagen Docker que se desplegará                            |
| `ports.containerPort`           | Container  | Puerto que expone el contenedor                            |
| `Service.spec.type`             | Service    | Tipo de servicio (`ClusterIP`, `NodePort`, `LoadBalancer`) |
| `Service.spec.selector`         | Service    | Indica a qué pods se dirige el servicio                    |
| `Service.spec.ports`            | Service    | Puerto expuesto y mapeo con los pods                       |


### Comandos útiles:

- Ver pods:
```
kubectl get pods
```

- Ver svc entre pod y politica:
```
kubectl get svc

```

- Te muestra el estado del pod:
```
kubectl describe pod <nombre>
```

- Logs de un pod:
```
kubectl logs <pod>
```

- Acceder al pod:
```
kubectl exec -it <pod> -- sh
```
- Eliminar pod:
```
kubectl delete -f archivo.yml

```

### Resumen

- **Deployment:** administra la creación y actualización de pods

- **Service:** permite exponer los pods dentro o fuera del clúster

- **Labels y selectors:** clave para relacionar servicios y pods

- kubectl apply -f archivo.yml para crear o actualizar recursos

- kubectl get pods, kubectl get svc para ver el estado


## CI/CD

## Cloud Computing

## Elasticsearch

## Pila ELK

## Ejercicios de Prueba examen

### Ejercicio1


Crear un fichero Dockerfile para empaquetar el código, sus pasos deben ser:

- Utilizar como base la imagen “python:slim".
- Utilizar como directorio de trabajo /code.
- Copiar los ficheros de código Python en la carpeta /code.


**Mi Dockerfile**

```
FROM python:slim

WORKDIR /code

COPY . /code

```
A continuación, crear un workflow de GitHub Actions con las siguientes características:

- Nombre: examen-workflow
- Se debe disparar al hacer Push en una rama llamada “nueva-caracteristica”.
- Se debe ejecutar en un entorno “ubuntu-latest.

El workflow debe hacer lo siguiente:

- Descargar el código.
- Utiliza la acción “actions/setup-python@v6” (https://github.com/actions/setup-python) para configurar un entorno Python de versión 3.10.
Ejecutar los siguientes comandos:
- a. pip install pytest
- b. pytest test.py
- Utiliza la acción ”docker/build-push-action@v6” (https://github.com/marketplace/actions/build-and-push-docker-images) para construir una imagen usando el Dockerfile. No es necesario subir la imagen a ningún registro.


Creado archivo en **/.github/workflows/examen-workflow.yml** :
```yml
name: examen-workflow

on:
  push:
    branches:
      - nueva-caracteristica

jobs:
  build-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.10
        uses: actions/setup-python@v6
        with:
          python-version: '3.10'

      - name: Install pytest
        run: pip install pytest

      - name: Run tests
        run: pytest test.py

      - name: Build Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: false   # No se sube la imagen a ningún registro

```

### Ejercicio2

Crear un índice “usuarios” con las siguientes características:
- Campo “nombre”, tipo text.
- Campo “apellido”, tipo text.
- Campo “edad”, tipo integer.

TIPO : **PUT**

URI:
```
http://localhost:9200/usuarios 
```

Cuerpo:
```
{
   "mappings": {
       "properties": {
           "nombre": { "type": "text" },
           "apellido": { "type": "text"},
           "edad": {"type": "integer"}
       }
   }
}

```


Añadir elementos al índice:

TIPO : **PUT**

URI:
```
http://localhost:9200/usuarios/_doc/10
```

Cuerpo:
```
{
    "nombre": "Jon",
    "apellido": "Blanco",
    "edad": 27
}


```

TIPO : **PUT**

URI:
```
http://localhost:9200/usuarios/_doc/11
```

Cuerpo:
```
{
    "nombre": "Amaia",
    "apellido": "Lopez",
    "edad": 28
}


```
TIPO : **PUT**

URI:
```
http://localhost:9200/usuarios/_doc/12
```

Cuerpo:
```
{
    "nombre": "Hodei",
    "apellido": "Bilbao",
    "edad": 33
}


```

*Tarea: Obtener los datos de los usuarios cuyo apellido empiece por la letra B.*

TIPO : **GET**

URI:
```
http://localhost:9200/usuarios/_search
```

Cuerpo:
```
{
   "query": {
    "prefix": {
        "apellido": "b"
    }
   }
}



```

*Tarea: Recuperar los datos de “Amaia Lopez”, pero utilizando “Loepz” como término de búsqueda*
TIPO : **GET**

URI:
```
http://localhost:9200/usuarios/_search
```

Cuerpo:
```
{
   "query": {
    "match": {
        "apellido": {
            "query": "Loepz",
            "fuzziness":1
        }
    }
   }
}



```


### Ejercicio3

Configurar un pipeline Logstash con las siguientes características:
- Recibir datos en formato JSON a través de conexiones HTTP en el puerto 9870.
- Eliminar el campo “ciudad” de cada objeto JSON recibido.
- Escribir cada objeto JSON en el índice “usuarios” de Elasticsearch recién creado.

**Añadir archivo de configuración en pipeline/Ej3_logstash.conf:**

```
input {
        http {
                port => 9870
                codec => json
        }
}

filter {
        mutate {
                remove_field => ["ciudad"]
        }
}

output {
        elasticsearch {
                hosts => ["http://localhost:9200"]
                index => "usuarios"
        }

        stdout {
                codec => rubydebug
        }
}
```

Enviar una petición a traves de curl verificando si es correcta:

```
curl -X POST "http://localhost:9870"      -H "Content-Type: application/json"      -d '{
       "nombre": "Amaia",
       "apellido": "Lopez",
       "edad": 28,
       "ciudad": "Donostia"
     }'

```

Comprobar que la petición ha sido correcta:

```
curl http://localhost:9200/usuarios/_search?pretty
```

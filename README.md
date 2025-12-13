# Examen2AS

## Índice del repositorio

- [Dockerfile](#dockerfile)
  - [Estructura básica de un Dockerfile](#estructura-básica-de-un-dockerfile)
  - [Instrucciones más importantes de Dockerfile](#instrucciones-más-importantes-de-dockerfile)
  - [Comandos básicos relacionados](#comandos-básicos-relacionados)

- [Docker Compose](#docker-compose)
  - [Comandos básicos de Docker Compose](#comandos-básicos-de-docker-compose)
  - [Ejemplo típico de docker-compose.yml](#ejemplo-típico-de-docker-composeyml)

- [Kubernettes](#kubernettes)
  - [Ejemplo típico: Deployment + Service](#ejemplo-típico-deployment--service)
  - [Explicación de los elementos clave](#explicación-de-los-elementos-clave)
  - [Comandos útiles](#comandos-útiles)
  - [Resumen](#resumen)

- [CI/CD](#cicd)
  - [Ejemplo básico de workflow](#ejemplo-básico-de-workflow)
  - [Estructura de un workflow](#estructura-de-un-workflow)
  - [Workflows con múltiples jobs](#workflows-con-múltiples-jobs)
  - [Filtrar por ramas](#filtrar-por-ramas)
  - [Ejecutar en múltiples sistemas operativos](#ejecutar-en-múltiples-sistemas-operativos)
  - [GitHub Marketplace](#github-marketplace)
  - [Secretos en GitHub Actions](#secretos-en-github-actions)
  - [Ejercicios](#ejercicios)
    - [Ejercicio 1](#ejercicio1)
    - [Ejercicio 2](#ejercicio2)
    - [Ejercicio 3](#ejercicio3)

- [Cloud Computing](#cloud-computing)
  - [Tipos de auto-escalado](#tipos-de-auto-escalado)
  - [Auto-escalado horizontal (HPA)](#auto-escalado-horizontal-hpa)
  - [Auto-escalado vertical (VPA)](#auto-escalado-vertical-vpa)
  - [Métricas en Kubernetes](#métricas-en-kubernetes)

- [Elasticsearch](#elasticsearch)
  - [Mapping](#mapping)
  - [Operaciones CRUD (API REST)](#operaciones-crud-api-rest)
  - [Ejercicios](#ejercicios-1)
    - [Ejercicio 1](#ejercicio-1)
    - [Ejercicio 2](#ejercicio-2)
    - [Ejercicios de paginación](#ejercicios-paginación)
    - [Ejercicios de ordenación](#ejercicios-de-ordenación)
    - [Ejercicios de filtros](#ejercicios-de-filtros)
    - [Ejercicios de búsquedas difusas](#ejercicios-de-búsquedas-difusas)
    - [Ejercicios de prefijos de búsqueda y comodines](#ejercios-de-prefijos-de-busqueda-y-comodines)
    - [Ejercicios de expresiones regulares](#ejercicios-de-expresiones-regulares)

- [Pila ELK](#pila-elk)

- [Ejercicios de Prueba examen](#ejercicios-de-prueba-examen)
  - [Ejercicio 1](#ejercicio1-1)
  - [Ejercicio 2](#ejercicio2-1)
  - [Ejercicio 3](#ejercicio3-1)

---
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


---

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

---
## CI/CD

### Ejemplo básico de workflow

```
name: Primer workflow

on: push

jobs:
  hola_mundo:
    runs-on: ubuntu-latest
    steps:
      - name: Mostrar mensaje
        run: echo "Hola mundo"

```

### Estructura de un workflow

| Elemento  | Descripción                  |
| --------- | ---------------------------- |
| `name`    | Nombre del workflow          |
| `on`      | Evento que lo dispara        |
| `jobs`    | Conjunto de tareas           |
| `runs-on` | Sistema operativo del runner |
| `steps`   | Pasos que se ejecutan        |


### Workflows con múltiples jobs

Se pueden definir dependencias entre jobs usando *needs*.


```yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: pytest

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - run: echo "Construyendo aplicación"

```

### Filtrar por ramas

Ejecutar un workflow solo en determinadas ramas:

```yml
on:
  push:
    branches:
      - main
      - develop
```

### Ejecutar en múltiples sistemas operativos

```yml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

```

### GitHub Marketplace

GitHub ofrece un Marketplace con acciones reutilizables:

- Construir imágenes Docker

- Subir imágenes a Docker Hub

- Análisis de código

- Seguridad

Ejemplo: construir y subir una imagen Docker

```yml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    push: true
    tags: usuario/miapp
```

### Secretos en GitHub Actions

Para información sensible (tokens, contraseñas):

- Se usan secrets

- Se configuran en el repositorio

- Se accede con ${{ secrets.NOMBRE }}

Ejemplo:

```yml
- name: Login Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Ejercicios

#### Ejercicio1

Crear un workflow con los siguientes pasos:
- Descargar el código.
- Instalar pytest.
- Utilizar pytest para ejecutar los Tests del código.
- Verificar que el workflow finaliza correctamente.
- Introducir un fallo en el código y comprobar el resultado
del workflow

```yml
name: Python Tests
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'

    - name: Install depencies
      run: |
        python -m pip install --upgrade pip
        pip install pytest

    - name: Run tests
      run: pytest test.py

```

#### Ejercicio2

Crear un workflow que analice la calidad del código con
la siguiente acción:
- Nombre: advanced-security/python-lint-code-scanning-action@v1
- Parámetros:
  - linter: pylint
  - Es necesario añadir el siguiente parámetro al workflow:

      runs-on: …
        permissions:
          security-events: write
      steps:
- Comprobar el resultado.
- Repositorio, sección “Seguridad”, apartado “Análisis de código”.


**IMPORTANTE: El repositorio debe de ser público sino no se puede usar code scanning.**

```yml
name: Python Lint Code Scanning

on:
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read # Añade permisos de lectura de contenido para el checkout
  security-events: write # Mantiene permisos de escritura para la acción de linting
  actions: read # A veces es necesario para que las integraciones lean datos del workflow


jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.MY_PAT}}
          
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          pip install pylint

      - name: Run Python Lint Code Scanning
        uses: advanced-security/python-lint-code-scanning-action@v1
        with:
          linter: pylint

```

#### Ejercicio3

Crear 2 workflows:
- 1) Se dispara al hacer Push en main, ejecuta los Tests como se describe en el ejercicio 1.
- 2) Se dispara al hacer Pull Request en cualquier rama,ejecuta la acción Linter presentada en el ejercicio 2.

Crear una nueva rama y añadir en el código:
- Una función que incremente el saldo en 1000.
- El Test correspondiente.
- Hacer una Pull Request para juntar la rama con main.
- Verificar que cada workflow se ha ejecutado correctamente.

**pythonTest.yml:**

```yml
name: Python Tests
on:
  push:
    branches: [ main ]
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'

    - name: Install depencies
      run: |
        python -m pip install --upgrade pip
        pip install pytest

    - name: Run tests
      run: pytest test.py

```

**pylint.yml:**

```yml
name: Python Lint Code Scanning

on:
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read # Añade permisos de lectura de contenido para el checkout
  security-events: write # Mantiene permisos de escritura para la acción de linting
  actions: read # A veces es necesario para que las integraciones lean datos del workflow


jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.MY_PAT}}
          
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          pip install pylint

      - name: Run Python Lint Code Scanning
        uses: advanced-security/python-lint-code-scanning-action@v1
        with:
          linter: pylint

```
---
## Cloud Computing

### Tipos de auto-escalado

| Tipo | Descripción |
|----|-------------|
| **Horizontal** | Añade o elimina recursos del mismo tipo |
| **Vertical** | Aumenta o reduce la capacidad del recurso |

### Auto-escalado horizontal (HPA)

Controla automáticamente el número de pods de un Deployment.

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: mi-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```
Comandos útiles:

```
kubectl apply -f hpa.yaml
kubectl get hpa
kubectl describe hpa
```

### Auto-escalado vertical (VPA)

Ajusta automáticamente CPU y memoria de los pods.

```yml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: mi-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: Auto
```

Comandos útiles:

```bash
kubectl apply -f vpa.yaml
kubectl get vpa
kubectl describe vpa
```

### Métricas en Kubernetes

```bash
kubectl top pod
kubectl top node
```

---

## Elasticsearch

### Mapping

- Define el esquema de los documentos de un índice.

- Tipos de datos: text, integer, float, boolean, date, etc.

- Elasticsearch puede inferirlos automáticamente

- Se pueden definir manualmente

Ejemplo:

```json
{
  "mappings": {
    "properties": {
      "titulo": { "type": "text" },
      "autor": { "type": "text" },
      "isbn": { "type": "integer" }
    }
  }
}

```
### Operaciones CRUD (API REST)

| Operación  | Método REST |
| ---------- | ----------- |
| Crear      | POST / PUT  |
| Leer       | GET         |
| Actualizar | POST / PUT  |
| Borrar     | DELETE      |

### Ejercicios


#### Ejercicio1
Crear un índice “peliculas” con las siguientes características:
- Campo “titulo”, tipo text.
- Campo “director”, tipo text.
- Campo “año”, tipo integer.

TIPO : **PUT**

URI:
```
http://localhost:9200/peliculas/
```

Cuerpo:
```json
{
   "mappings": {
    "properties": {
      "titulo":   { "type": "text" },
      "director": { "type": "text" },
      "año":      { "type": "integer" }

        }
    }
   }
}
```
Modificar el campo “año” de Gladiator (ID = 2)

TIPO : **POST**

URI:
```
http://localhost:9200/peliculas/_update/2
```

Cuerpo:
```json
{
  "doc": {
    "año": 2000
  }
}

```
Borrar la película “Inception” (ID = 3)

TIPO : **DELETE**

URI:
```
http://localhost:9200/peliculas/_doc/2
```

Cuerpo: Nada

#### Ejercicio 2

- Crear un índice llamado “shakespeare”
- Seguir el esquema de la diapositiva anterior.
- Cargar el dataset de las obras de Shakespeare en el
índice “shakespeare”.
Encontrar:
- A qué obra de Shakespeare pertenece la frase “To be or not to be” y en qué línea de la obra se encuentra.
- Cuántas frases tiene el personaje "OCTAVIUS CAESAR" en la obra "Antony and Cleopatra”.


Indice:

TIPO : **PUT**

URI:
```
http://localhost:9200/shakespeare
```

Cuerpo:
```json
{
  "mappings": {
    "properties": {
      "speaker":        { "type": "keyword" },
      "play_name":      { "type": "keyword" },
      "line_id":        { "type": "integer" },
      "speech_number":  { "type": "integer" }
    }
  }
}


```
Cargar datos desde bin:


TIPO : **POST**

URI:
```
http://localhost:9200/shakespeare/_bulk
```

Buscar la frase “To be or not to be”:

TIPO : **GET**

URI:
```
http://localhost:9200/shakespeare/_search?pretty
```

Cuerpo:
```json
{
  "query": {
    "match_phrase": {
      "text_entry": "To be or not to be"
    }
  }
}

```

Contar cuántas frases dice “OCTAVIUS CAESAR” en “Antony and Cleopatra”

TIPO : **GET**

URI:
```
http://localhost:9200/shakespeare/_search?pretty
```

Cuerpo:
```json
{
  "query": {
    "bool": {
      "must": [
        { "term": { "speaker": "OCTAVIUS CAESAR" }},
        { "term": { "play_name": "Antony and Cleopatra" }}
      ]
    }
  }
}


```

#### Ejercicios paginación

- size → número de resultados que quieres devolver.
- from → desplazamiento (offset), es decir, cuántos documentos saltarte antes de empezar a devolver resultados.

**Recuperar los primeros 20 documentos del índice:**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
  "size":20
}

```
**Recuperar los segundos 20 documentos del índice**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
  "from":20,
  "size":20
}

```
**Buscar las personas que residen en Texas (state = "TX") y devolver los primeros 15 resultados:**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

```json
{
    "query":{
        "term":{
            "state":"TX"
        }
    },
    "size": 15
}
```

#### Ejercicios de Ordenación

**Residentes en Los Ángeles (state = "LA") ordenados por edad ascendente:**


TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
    "query":{
        "term":{
            "state":"LA"
        }
    },
    "sort": [
        {"age":{ "order" : "asc"}}
    ]

```

#### Ejercicios de filtros


** Recuperar los datos de usuarios cuyo estado de residencia sea Los Ángeles (código de estado “LA”) pero que su ciudad de residencia no sea Loretto, y que además su edad sea superior a los 33 años**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
    "query":{
        "bool": {
            "filter":[
                 {"term": {"state": "LA"}},
                 { "range": {"age": {"gt":33}}}
            ],
            "must_not": [
                {"term": {"city":"Loretto"}}
            ]
        }
    }
}

```

#### Ejercicios de búsquedas difusas

** Utilizar una búsqueda difusa para recuperar los datos de la persona residente en Wyoming, pero utilizando ”Woyming” como término de búsqueda**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
    "query":{
        "fuzzy": {
            "city":{
                "value": "Woyming",
                "fuzziness":1
            }
        }
    }
}

```

#### Ejercios de prefijos de busqueda y comodines

- prefix query → para buscar términos que empiezan por un prefijo.
- wildcard query → para buscar términos que cumplen un patrón con comodines (* = cualquier número de caracteres, ? = un solo carácter).

**Recuperar los datos de las personas cuyos apellidos comiencen por “Mc”**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
    "query":{
        "prefix": {
            "lastname": "Mc"
        }
    }
}

```


**Recuperar los datos de las personas cuya ciudad de residencia comience por la letra “G” y acabe con “field”**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
    "query":{
        "wildcard": {
            "city": "G*field"
        }
    }
}

```
#### Ejercicios de expresiones regulares

En Elasticsearch, las búsquedas con expresiones regulares se hacen con el operador regexp. Este permite definir patrones más complejos que los prefijos o comodines.


**Utilizando expresiones regulares, recuperar los datos de las personas cuyo nombre de empleador comience por la letra "A" y esté compuesto por 4 o 5 letras, p.e. los empleadores "Avit" o "Amtap".**

TIPO : **GET**

URI:
```
http://localhost:9200/accounts/_search
```

Cuerpo:
```json
{
    "query":{
        "regexp": {
            "employer": {
                "value": "A.{3,4}"
            }
        }
    }
}


```
---

## Pila ELK

---
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

# Taller Hands-On: MLflow + Docker
## Del dato al modelo desplegado

**Autora:** Carolina Mantilla  
**Fecha:** Mayo 25, 2026

---

## 1. Objetivo del taller

El objetivo de este taller fue recorrer de forma práctica el ciclo de vida de un modelo de Machine Learning utilizando **MLflow** y **Docker**.

El ejercicio permitió comprender cómo un conjunto de datos puede convertirse en un modelo entrenado, registrado, versionado y finalmente desplegado como una API capaz de recibir datos y devolver predicciones.

El enfoque principal no fue profundizar en el dominio del dataset ni en el algoritmo de Machine Learning, sino entender el flujo completo:

```text
datos → entrenamiento → tracking → registro → despliegue → inferencia
```

---

## 2. Herramientas utilizadas

Durante el taller se utilizaron las siguientes herramientas:

- **Docker**: para ejecutar el entorno de forma reproducible.
- **Docker Compose**: para levantar varios servicios relacionados entre sí.
- **MLflow**: para tracking, registro del modelo y serving.
- **Scikit-learn**: para entrenar un modelo clásico de clasificación.
- **Python**: como lenguaje base del proyecto.
- **Git y GitHub**: para versionar y entregar el material del taller.

---

## 3. Proyecto base utilizado

El taller se basó en el repositorio:

```text
https://github.com/mtpatter/mlflow-tutorial
```

Este repositorio contiene un ejemplo completo de entrenamiento, registro y despliegue de un modelo de clasificación usando MLflow y Docker.

La estructura principal del proyecto fue:

```text
mlflow-tutorial/
├── Dockerfile
├── README.md
├── assets/
├── clf-train-registry.py
├── clf-train.py
├── compose-server.yml
├── docker-compose-no-registry.yml
├── docker-compose.yml
├── predict.sh
├── requirements.txt
├── runServer.sh
└── serveModel.sh
```

Después de ejecutar el taller, también se generaron archivos como:

```text
mlflow-artifact-root/
mlflow.db
test.csv
```

---

## 4. Relación con el Data Lifecycle

En el ciclo de vida tradicional de datos se suele hablar de:

```text
ingestión → transformación → consumo
```

En Machine Learning, ese flujo se amplía porque el resultado no es solo un dato procesado, sino un modelo que puede ser usado para hacer predicciones.

En este taller, el flujo fue:

```text
datos → entrenamiento → validación → tracking → registro → serving → inferencia
```

Relación entre ambos ciclos:

| Data Lifecycle | ML Lifecycle |
|---|---|
| Ingestión de datos | Carga del dataset de Scikit-learn |
| Transformación | Separación de variables y división train/test |
| Consumo | Entrenamiento del modelo |
| Producto resultante | Modelo entrenado |
| Publicación | Modelo servido como API |
| Seguimiento | Tracking, métricas, registry y versiones en MLflow |

---

## 5. Arquitectura del taller

El archivo principal de orquestación fue:

```text
docker-compose.yml
```

Este archivo define tres servicios:

```text
server
trainmodel
servemodel
```

### 5.1 Servicio `server`

Este servicio levanta el servidor de MLflow.

Su función es permitir el tracking de experimentos, almacenar métricas, registrar modelos y mostrar la interfaz web de MLflow.

Se accede desde el navegador en:

```text
http://localhost:8000
```

Dentro del contenedor, MLflow corre en el puerto `5000`, pero Docker lo expone en el puerto `8000` de la máquina local.

---

### 5.2 Servicio `trainmodel`

Este servicio ejecuta el entrenamiento del modelo.

El comando principal es:

```bash
python clf-train-registry.py clf-model http://server:5000 --outputTestData test.csv
```

Este proceso realiza lo siguiente:

1. Carga un dataset de clasificación desde Scikit-learn.
2. Separa los datos en entrenamiento y prueba.
3. Entrena un modelo Random Forest.
4. Registra el experimento en MLflow.
5. Guarda el modelo como `clf-model`.
6. Crea una versión del modelo.
7. Asigna el alias `Staging`.
8. Genera el archivo `test.csv` para probar inferencia.

Este contenedor termina después de entrenar el modelo, por eso aparece con estado `Exited (0)`.

---

### 5.3 Servicio `servemodel`

Este servicio se encarga de servir el modelo como API.

El modelo servido fue:

```text
models:/clf-model@Staging
```

Esto significa que MLflow busca el modelo llamado `clf-model` que tenga el alias `Staging`.

El endpoint de inferencia quedó disponible en:

```text
http://localhost:1234/invocations
```

---

## 6. Ejecución del taller

### 6.1 Clonar el repositorio base

Se clonó el repositorio del tutorial:

```bash
git clone https://github.com/mtpatter/mlflow-tutorial.git
cd mlflow-tutorial
```

Se verificó la estructura con:

```bash
tree -L 2
```

Resultado esperado:

```text
.
├── Dockerfile
├── README.md
├── assets
│   └── cartoon-serve-api.png
├── clf-train-registry.py
├── clf-train.py
├── compose-server.yml
├── docker-compose-no-registry.yml
├── docker-compose.yml
├── predict.sh
├── requirements.txt
├── runServer.sh
└── serveModel.sh
```

---

### 6.2 Levantar los servicios con Docker Compose

Se ejecutó:

```bash
docker compose -f docker-compose.yml up --build
```

Este comando construyó las imágenes necesarias y levantó los servicios definidos en Docker Compose.

Durante la ejecución se observó que el modelo fue entrenado y registrado correctamente:

```text
Test data written to 'test.csv'
Model status: READY
Set 'Staging' alias for model 'clf-model' version 1
Successfully verified 'Staging' alias for model 'clf-model' version 1
```

También se confirmó que el modelo quedó servido como API:

```text
Listening at: http://0.0.0.0:1234
```

---

### 6.3 Verificar MLflow UI

Se abrió el navegador en:

```text
http://localhost:8000
```

En la interfaz de MLflow se pudo observar:

- El experimento creado.
- El run ejecutado.
- El modelo registrado como `clf-model`.
- La versión `v1`.
- El alias `Staging`.

Esto confirma que MLflow registró correctamente el entrenamiento y el modelo.

---

### 6.4 Hacer una predicción usando la API

En una segunda terminal, se ejecutó:

```bash
./predict.sh test.csv
```

El modelo respondió con predicciones en formato JSON:

```json
{
  "predictions": [
    0.5074,
    0.0,
    0.9837,
    0.986,
    0.9913
  ]
}
```

Cada valor representa una predicción o probabilidad generada por el modelo para una fila del archivo `test.csv`.

Esto confirma que el modelo fue desplegado correctamente y que puede recibir datos para realizar inferencia.

---

## 7. Verificación de contenedores

Se revisaron los servicios activos con:

```bash
docker compose ps
```

Resultado observado:

```text
servemodel   Up
server       Up
```

Después se revisaron todos los contenedores, incluyendo los terminados:

```bash
docker compose ps -a
```

Resultado observado:

```text
servemodel   Up
server       Up
trainmodel   Exited (0)
```

Esto es correcto porque:

- `server` debe seguir activo para mostrar MLflow UI.
- `servemodel` debe seguir activo para atender predicciones.
- `trainmodel` termina después de entrenar y registrar el modelo.

---

## 8. Archivos generados

Después de ejecutar el taller, aparecieron archivos adicionales:

```text
mlflow-artifact-root/
mlflow.db
test.csv
```

### `mlflow.db`

Es una base de datos SQLite donde MLflow guarda información del tracking y registry, como:

- Experimentos.
- Runs.
- Métricas.
- Parámetros.
- Modelos registrados.
- Versiones.
- Aliases.

### `mlflow-artifact-root/`

Es la carpeta donde MLflow guarda los artefactos del modelo, es decir, los archivos necesarios para cargarlo, reproducirlo o servirlo.

### `test.csv`

Es el archivo de prueba generado durante el entrenamiento. Se utiliza para enviar datos al modelo desplegado y validar que la API responde correctamente.

---

## 9. Comandos principales utilizados

### Levantar el proyecto

```bash
docker compose -f docker-compose.yml up --build
```

### Ver contenedores activos

```bash
docker compose ps
```

### Ver todos los contenedores

```bash
docker compose ps -a
```

### Ejecutar predicción

```bash
./predict.sh test.csv
```

### Ver la estructura del proyecto

```bash
tree -L 2
```

### Apagar los servicios

```bash
docker compose down
```

---

## 10. Interpretación del flujo completo

El taller permitió ejecutar el siguiente flujo:

```text
1. Se cargan datos desde Scikit-learn.
2. Se preparan variables de entrada y variable objetivo.
3. Se divide el dataset en entrenamiento y prueba.
4. Se entrena un modelo Random Forest.
5. Se registra el entrenamiento en MLflow.
6. Se guarda el modelo como artefacto.
7. Se registra el modelo como clf-model.
8. Se asigna el alias Staging.
9. Se despliega el modelo como API.
10. Se envía un archivo CSV al endpoint.
11. El modelo responde con predicciones.
```

Representación resumida:

```text
dataset → train/test split → entrenamiento → MLflow tracking → model registry → serving → predicción
```

---

## 11. Conclusión

Este taller permitió comprender cómo un modelo de Machine Learning puede pasar de ser un resultado local de entrenamiento a convertirse en un servicio desplegado y consumible mediante una API.

La práctica mostró la importancia de herramientas como MLflow y Docker dentro de un flujo MLOps básico:

- MLflow permite registrar experimentos, modelos, métricas y versiones.
- Docker permite ejecutar el entorno de forma reproducible.
- Docker Compose permite coordinar varios servicios.
- El serving permite que el modelo sea consumido por otros sistemas.

En términos del ciclo de vida de datos, este taller muestra cómo el consumo de datos puede evolucionar hacia la creación de un modelo desplegable, agregando nuevas etapas como entrenamiento, validación, registro, versionamiento e inferencia.

Este ejercicio funciona como una introducción práctica al mundo de MLOps y como puente hacia herramientas más avanzadas como pipelines automatizados, integración continua para modelos y servicios administrados como Amazon SageMaker.
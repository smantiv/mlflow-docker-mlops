# MLflow Docker MLOps Tutorial

Complete example of training a classifier model with scikit-learn and serving it with MLflow using Docker and Docker Compose.

![img](assets/cartoon-serve-api.png)

## Overview

This repository walks through an example of:
- Training a classifier model with scikit-learn
- Serving the model with MLflow
- Using MLflow registry for model tracking and versioning
- Containerizing everything with Docker

### Quick Links
- [Medium Blog](https://maria-patterson.medium.com/) - Walkthrough and tutorials
- [Towards Data Science Article](https://towardsdatascience.com) - Detailed explanation
- [Coffee Fund](https://github.com/sponsors/mtpatter) - Support the author

**Note:** Updated September 2024 to MLflow 2.16.2

## Quick Start (TLDR)

### With Registry
```bash
docker compose -f docker-compose.yml up --build
```
Access MLflow UI at `http://localhost:8000`

### Without Registry
```bash
docker compose -f docker-compose-no-registry.yml up --build
```
Model served on port 1234

### MLflow Server Only
```bash
docker compose -f compose-server.yml up --build
```

## Serving Models Without Registry

### Train a Model
```bash
python clf-train.py clf-model --outputTestData test.csv
```

Trains a Random Forest classifier on the sklearn breast cancer dataset and saves it locally.

### Serve Model
```bash
mlflow models serve -m clf-model -p 1234 -h 0.0.0.0 --env-manager local
```

## Serving Models With Registry

### Start MLflow Server
```bash
mlflow server \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root ./mlflow-artifact-root \
  --host 0.0.0.0 \
  --port 8000
```

Uses SQLite backend and stores artifacts locally.

### Train and Register Model
```bash
python clf-train-registry.py clf-model "http://localhost:8000" --outputTestData test.csv
```

Trains a Random Forest classifier, modifies the predict method to return probabilities, and registers the model. The newest version is automatically moved to the `Staging` alias.

### Serve Registered Model
```bash
export MLFLOW_TRACKING_URI=http://localhost:8000
mlflow models serve -m models:/clf-model@Staging -p 1234 -h 0.0.0.0 --env-manager local
```

## Making Predictions

### Via cURL
```bash
curl http://localhost:1234/invocations \
  -H 'Content-Type: text/csv' \
  --data-binary @test.csv
```

### Via Script
```bash
./predict.sh test.csv
```

Both return an array of predicted probabilities.

## Cleanup

```bash
docker compose down
```

**Note:** Compose files mount volumes and write to the local directory. Clean up manually if needed.

## Project Structure

- `clf-train.py` - Train model without registry
- `clf-train-registry.py` - Train and register model with MLflow
- `predict.sh` - Make predictions against served model
- `docker-compose.yml` - Full stack with registry
- `docker-compose-no-registry.yml` - Stack without registry
- `compose-server.yml` - MLflow server only
- `assets/` - Images and resources

## Technologies

- **MLflow 2.16.2** - Model tracking, registry, and serving
- **Scikit-learn** - Machine learning classifier
- **Docker & Compose** - Containerization and orchestration
- **Python** - Main language
- **SQLite** - Model registry backend

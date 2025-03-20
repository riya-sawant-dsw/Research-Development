# MLflow Setup and Usage Guide

## Prerequisites
Ensure you have Python and pip installed. You can verify by running:

```sh
python --version
pip --version
```

## Installation
To install Jupyter and MLflow, run:

```sh
python -m pip install jupyter
```

## Running Jupyter Notebook
Start Jupyter Notebook with:

```sh
python -m notebook
```

## Setting Up MLflow Server
### Creating a Directory for MLflow
```sh
mkdir MLFlow
cd MLFlow
```

### Starting MLflow Server (Basic Setup)
```sh
mlflow server --host 127.0.0.1 --port 5000
```

### Checking Available MLflow Server Options
```sh
mlflow server --help
```

### Setting MLflow Tracking URI
```sh
set MLFLOW_TRACKING_URI=http://localhost:5000
```

## Serving a Model with MLflow
```sh
mlflow models serve -m runs:/<run_id>/model -p 5000
```
Replace `<run_id>` with the actual run ID of the trained model.

## Advanced MLflow Server Configurations

### Using File-based Backend Store
```sh
mlflow server --backend-store-uri file:///C:/Users/Admin/mlruns --default-artifact-root file:///C:/Users/Admin/mlruns --host 127.0.0.1 --port 5000
```

### Using SQLite Backend Store
```sh
mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri sqlite:///mlruns.db --default-artifact-root ./mlartifacts --serve-artifacts
```

### Alternative SQLite Configuration with Explicit Artifact Destination
```sh
mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri sqlite:///mlruns.db --default-artifact-root ./mlartifacts --artifacts-destination ./mlartifacts --serve-artifacts
```

## Building a Docker Image for the MLflow Model
```sh
mlflow models build-docker -m runs:/<run_id>/<artifact_path> -n <image_name> --enable-mlserver
```
Replace `<run_id>` and `<artifact_path>` with the actual run ID and path to the model artifacts.

## Notes
- Ensure the necessary dependencies for MLflow models are installed.
- Replace placeholders (`<run_id>`, `<artifact_path>`, `<image_name>`) with actual values from your MLflow runs.
- If running in a production environment, configure security settings accordingly.

---
This documentation provides a step-by-step guide for setting up and using MLflow to track, serve, and containerize models.


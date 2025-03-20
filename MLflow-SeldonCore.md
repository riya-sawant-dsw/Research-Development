# MLflow Model Training and Deployment with Seldon Core

## Overview
This guide provides a step-by-step process for training an ML model with MLflow, packaging it, and deploying it using Seldon Core on Kubernetes.

## Prerequisites
- Minikube installed and running
- Kubernetes CLI (`kubectl`) installed
- Helm installed
- Google Cloud SDK installed (for gsutil)
- Chocolatey installed (for Helm on Windows)

## Setup

### 1. Create and Activate a Virtual Environment
```bash
conda create -n python3.8-mlflow-example python=3.8
conda activate python3.8-mlflow-example
```

### 2. Install Required Packages
```bash
pip install mlflow
mlflow
```

### 3. Clone the MLflow Repository and Train a Model
```bash
git clone https://github.com/mlflow/mlflow
cd mlflow/examples
python sklearn_elasticnet_wine/train.py
```

### 4. Start the MLflow UI
```bash
mlflow ui
```

### 5. Package the Model
```bash
pip install conda-pack mlserver mlserver-mlflow
cd /MLflow-SeldonCore/mlflow/examples/mlruns/0/29db70bef91d40f389c71aae02b5c66d/artifacts/model
conda pack -o environment.tar.gz -f
```

### 6. Upload Model to Google Cloud Storage
```bash
pip install gsutil
gsutil --version
gsutil cp -r ../model gs://seldon-models/test/elasticnet_wine_29db70bef91d40f389c71aae02b5c66d
```

### 7. Authenticate Google Cloud
```bash
gcloud auth login
```

### 8. Copy Model Files on Windows
```bash
xcopy /E /I "D:\MLflow-SeldonCore\mlflow\examples\mlruns\0\29db70bef91d40f389c71aae02b5c66d\artifacts\model" "D:\seldon-models\elasticnet_wine_29db70bef91d40f389c71aae02b5c66d"
cd D:\MLflow-SeldonCore\mlflow\examples
```

### 9. Build and Run MLflow Model in Docker
```bash
mlflow models build-docker -m runs:/29db70bef91d40f389c71aae02b5c66d/model -n mlflow-wine-model --enable-mlserver
docker run -p 5000:5000 mlflow-wine-model
```

## Deploying on Kubernetes with Seldon Core

### 10. Start Minikube and Setup Kubernetes CLI
```bash
minikube start
curl.exe -LO "https://dl.k8s.io/release/v1.32.0/bin/windows/amd64/kubectl.exe"
kubectl version --client
kubectl cluster-info
```

### 11. Install Seldon Core Operator
```bash
kubectl get crd | findstr seldon
kubectl create namespace seldon-system
kubectl apply -f https://github.com/SeldonIO/seldon-core/releases/download/v1.17.0/seldon-core-operator.yaml -n seldon-system
```

### 12. Install Helm and Seldon Core via Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

#### Install Chocolatey for Helm (Run as Administrator on Windows)
```bash
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
choco install kubernetes-helm
```

#### Install Seldon Core Operator
```bash
helm install seldon-core seldon-core-operator --repo https://storage.googleapis.com/seldon-charts --set usageMetrics.enabled=true --set istio.enabled=false --namespace seldon-system
```

### 13. Mount Model in Minikube
```bash
minikube mount D:\MLflow-SeldonCore\mlflow\examples\mlruns\0\29db70bef91d40f389c71aae02b5c66d\artifacts\model:/mnt/models
```

### 14. Deploy Model with Kubernetes
```bash
kubectl apply -f wine-deployment.yaml -n default
```

### 15. Verify Deployment
```bash
kubectl get pods -n seldon-system
kubectl get svc -n seldon-system
kubectl get endpoints -n seldon-system
kubectl describe pod seldon-controller-manager-75f4f48c9f-2m6rv -n seldon-system
```

## References
- [Seldon MLflow Server](https://docs.seldon.io/projects/seldon-core/en/latest/servers/mlflow.html)
- [MLflow CLI Documentation](https://mlflow.org/docs/latest/api_reference/cli.html#mlflow-models-build-docker)
- [Deploy MLflow Model on Kubernetes](https://mlflow.org/docs/latest/deployment/deploy-model-to-kubernetes#build-docker-for-deployment)
- [Seldon Core Installation Guide](https://docs.seldon.io/projects/seldon-core/en/latest/workflow/install.html)
- [Seldon Core Releases](https://github.com/SeldonIO/seldon-core/releases)

This guide ensures a fully automated pipeline for training, packaging, and deploying ML models with MLflow and Seldon Core. 🚀


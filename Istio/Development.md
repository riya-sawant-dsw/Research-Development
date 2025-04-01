

# Istio Hands-on Demo

This guide walks you through the process of setting up Istio on Minikube and deploying a sample application. You will learn about Istio's capabilities such as traffic management, observability, and security.

## Step 1: Setting Up Minikube

### 1.1 Install Minikube

Install Minikube on your local machine.


- **On Windows (via Chocolatey)**:
  ```bash
  choco install minikube
  ```

- **On Linux (via apt)**:
  ```bash
  sudo apt-get install minikube
  ```

### 1.2 Start Minikube

Start a Minikube cluster:
```bash
minikube start
```

### 1.3 Install kubectl

Make sure `kubectl` is installed to interact with the Kubernetes cluster.

- **On Linux**:
  ```bash
  sudo apt-get install kubectl
  ```

- **On Windows (via Chocolatey)**:
  ```bash
  choco install kubernetes-cli
  ```

## Step 2: Installing Istio

### 2.1 Download Istio

Download the latest Istio release:
```bash
curl -L https://istio.io/downloadIstio | sh -
```

This installs Istio and places the `istioctl` command in your system path.

### 2.2 Install Istio with istioctl

To install Istio using the **demo profile** (which includes basic components like Istiod, Prometheus, Grafana, etc.):
```bash
cd istio-*/  # Replace with your Istio version directory
bin/istioctl install --set profile=demo -y
```

### 2.3 Verify Istio Installation

Check if Istio components are installed correctly:
```bash
kubectl get pods -n istio-system
```
You should see multiple Istio-related pods running (e.g., `istiod`, `istio-ingressgateway`).

## Step 3: Deploy a Sample Application

### 3.1 Deploy Sample Apps (Bookinfo App)

Deploy the default **Bookinfo** application using the following command:
```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

The **Bookinfo** app consists of four microservices:
- `productpage`: Displays product info
- `details`: Provides detailed product descriptions
- `reviews`: Shows reviews for products
- `ratings`: Manages rating data for products

### 3.2 Expose the Application via Istio Gateway

Expose the Bookinfo app using Istio's **Ingress Gateway**:
```bash
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

Access the Bookinfo app using the following command to get the URL:
```bash
minikube service istio-ingressgateway -n istio-system --url
```
Open the URL in your browser to access the app.

## Step 4: Configuring Traffic Management

### 4.1 Create VirtualService for Routing

Create a **VirtualService** to control traffic routing between different versions of the **reviews** service. For example, route 80% of traffic to version `v1` and 20% to version `v2` of the `reviews` service.

Create the file `reviews-virtualservice.yaml` with the following content:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v1
          weight: 80
        - destination:
            host: reviews
            subset: v2
          weight: 20
```

### 4.2 Apply the Configuration

Apply this **VirtualService** configuration:
```bash
kubectl apply -f reviews-virtualservice.yaml
```

Now 80% of traffic will go to version `v1` and 20% to version `v2` of the **reviews** service.

## Step 5: Observability and Metrics

### 5.1 Access Prometheus and Grafana Dashboards

Istio comes with **Prometheus** and **Grafana** for monitoring.

- **Prometheus**:
  ```bash
  kubectl port-forward -n istio-system svc/prometheus 9090:9090
  ```
  Access Prometheus at `http://localhost:9090`.

- **Grafana**:
  ```bash
  kubectl port-forward -n istio-system svc/grafana 3000:3000
  ```
  Access Grafana at `http://localhost:3000`. The default login is **admin** / **admin**.

### 5.2 View Traces with Jaeger

Istio integrates with **Jaeger** for distributed tracing.

Run the following to forward Jaeger's port:
```bash
kubectl port-forward -n istio-system svc/jaeger-query 5775:5775
```
Access Jaeger at `http://localhost:5775`.

## Step 6: Clean Up

Once you're done, you can delete the resources:
```bash
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl delete -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

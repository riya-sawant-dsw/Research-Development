

#  Running an External File with Telepresence in a Cluster

This document outlines the process of setting up a Minikube Kubernetes cluster, deploying a PostgreSQL database, backend, and frontend components, and using Telepresence to run an external Python script (`external_file.py`) that interacts with all three components simultaneously. The goal is to demonstrate seamless communication and early error detection without deploying the script to the cluster.

- **Environment**: Fedora 41
- **Tools**: Minikube, kubectl, Telepresence, Python 3

---

## Prerequisites

- Fedora system with internet access.
- Root or sudo privileges for installing packages and managing system files.
- Basic familiarity with terminal commands and Kubernetes concepts.

---

## Setup Instructions

### Step 1: Install Telepresence on Fedora

Telepresence enables local development by proxying traffic between your machine and the Kubernetes cluster.

1. **Update Fedora**:
   ```bash
   sudo dnf update -y
   ```

2. **Install `conntrack-tools`** (required for Telepresence networking):
   ```bash
   sudo dnf install -y conntrack-tools
   ```

3. **Download Telepresence Binary**:
   - Fetch the latest Telepresence binary for Linux (AMD64):
     ```bash
     sudo curl -fL https://github.com/telepresenceio/telepresence/releases/latest/download/telepresence-linux-amd64 -o /usr/local/bin/telepresence
     ```
   - Note: The command was run twice in your sequence, but once is sufficient.

4. **Make Telepresence Executable**:
   ```bash
   sudo chmod a+x /usr/local/bin/telepresence
   ```

5. **Verify Installation**:
   ```bash
   telepresence version
   ```
   - Expected output: Version number (e.g., `v2.x.x`).

---

### Step 2: Set Up Minikube Cluster

Minikube provides a local Kubernetes cluster for testing.

1. **Start Minikube**:
   ```bash
   minikube start
   ```

2. **Check Minikube Status**:
   ```bash
   minikube status
   ```
   - Ensure `host`, `kubelet`, and `apiserver` are `Running`.

3. **Verify Cluster**:
   - Check nodes:
     ```bash
     kubectl get nodes
     ```
     - Expected: One node (e.g., `minikube`).
   - Check pods (initially empty):
     ```bash
     kubectl get pods
     ```

4. **Clean Up (Optional)**:
   - If needed, stop and delete the cluster to start fresh:
     ```bash
     minikube stop
     minikube delete
     minikube start
     ```
   - Re-verify:
     ```bash
     kubectl get nodes
     kubectl get pods
     kubectl get services
     ```

---

### Step 3: Deploy PostgreSQL to Minikube

Add a PostgreSQL database as the first cluster component.

1. **Create PostgreSQL Pod and Service**:
   - You applied two slightly different configurations; the second one (Postgres 14) overwrote the first. Here’s the final version:
     ```bash
     kubectl apply -f - <<EOF
     apiVersion: v1
     kind: Pod
     metadata:
       name: postgres-pod
       labels:
         app: postgres
     spec:
       containers:
       - name: postgres
         image: postgres:14
         env:
         - name: POSTGRES_DB
           value: mydb
         - name: POSTGRES_USER
           value: postgres
         - name: POSTGRES_PASSWORD
           value: mypassword
         ports:
         - containerPort: 5432
     ---
     apiVersion: v1
     kind: Service
     metadata:
       name: postgres-service
     spec:
       ports:
       - port: 5432
         targetPort: 5432
       selector:
         app: postgres
     EOF
     ```

2. **Verify Deployment**:
   - Check pod status:
     ```bash
     kubectl get pods
     ```
     - Expected: `postgres-pod` in `Running` state.
   - Inspect pod details (if needed):
     ```bash
     kubectl describe po postgres-pod
     ```
   - Check service:
     ```bash
     kubectl get services
     ```
     - Expected: `postgres-service` with a ClusterIP.

---

### Step 4: Configure Telepresence

Set up Telepresence to connect to the cluster and manage timeouts.

1. **Create Telepresence Config Directory**:
   ```bash
   mkdir -p /home/riyasawant/.config/telepresence
   ```

2. **Set Helm Timeout** (to avoid installation delays):
   ```bash
   echo 'timeouts:
     helm: 120s' > /home/riyasawant/.config/telepresence/config.yml
   ```

3. **Install Telepresence Traffic Manager**:
   ```bash
   telepresence helm install
   ```
   - This installs the Traffic Manager in the `ambassador` namespace.

4. **Upgrade Telepresence (if needed)**:
   ```bash
   telepresence helm upgrade
   ```

5. **Troubleshooting Telepresence** (your steps to resolve issues):
   - Gather logs (if errors occur):
     ```bash
     telepresence gather-logs
     ```
   - Quit Telepresence with system cleanup:
     ```bash
     telepresence quit -s
     ```
   - Check for running processes:
     ```bash
     ps aux | grep telepresence
     ```
   - Kill lingering processes:
     ```bash
     sudo pkill -f telepresence
     ```
   - Remove stale socket:
     ```bash
     sudo rm -f /var/run/telepresence-daemon.socket
     ls -l /var/run/telepresence-daemon.socket
     ```

6. **Connect to Cluster**:
   ```bash
   telepresence connect
   ```
   - Sets up the Virtual Network Interface (VIF) and connects to the Traffic Manager.

7. **Check Status**:
   ```bash
   telepresence status
   ```
   - Expected: `Connected: true`.

---

### Step 5: Prepare the External File

Create a Python script to interact with multiple cluster components.

1. **Install Python and Dependencies**:
   ```bash
   sudo dnf install -y python3 python3-pip
   pip3 install psycopg2-binary
   pip3 install requests
   ```

2. **Create `external_file.py`**:
   ```bash
   nano external_file.py
   ```
   Add the following content:
   ```python
   import psycopg2
   import requests
   import time

   while True:
       try:
           # 1. Connect to PostgreSQL
           conn = psycopg2.connect(
               dbname="mydb",
               user="postgres",
               password="mypassword",
               host="postgres-service",
               port="5432"
           )
           print("Connected to postgres-service successfully!")
           conn.close()

           # 2. Interact with backend-service
           backend_response = requests.get("http://backend-service")
           if backend_response.status_code == 200:
               print(f"Backend-service responded: {backend_response.text[:50]}...")
           else:
               print(f"Backend-service failed with status: {backend_response.status_code}")

           # 3. Interact with frontend-service
           frontend_response = requests.get("http://frontend-service")
           if frontend_response.status_code == 200:
               print(f"Frontend-service responded: {frontend_response.text[:50]}...")
           else:
               print(f"Frontend-service failed with status: {frontend_response.status_code}")

       except psycopg2.Error as e:
           print(f"Error connecting to postgres-service: {e}")
       except requests.RequestException as e:
           print(f"Error with HTTP requests: {e}")
       except Exception as e:
           print(f"Unexpected error: {e}")

       time.sleep(5)  # Retry every 5 seconds
   ```
   Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

   - **Note**: Your original file had syntax errors (e.g., `print` indentation, truncated lines). The corrected version above fixes these.

---

### Step 6: Deploy Backend and Frontend Components

Add backend and frontend services to complete the cluster setup.

1. **Deploy Backend**:
   ```bash
   kubectl apply -f - <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: backend
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: backend
     template:
       metadata:
         labels:
           app: backend
       spec:
         containers:
         - name: backend
           image: nginx
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: backend-service
   spec:
     ports:
     - port: 80
       targetPort: 80
     selector:
       app: backend
   EOF
   ```

2. **Deploy Frontend**:
   ```bash
   kubectl apply -f - <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: frontend
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: frontend
     template:
       metadata:
         labels:
           app: frontend
       spec:
         containers:
         - name: frontend
           image: nginx
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: frontend-service
   spec:
     ports:
     - port: 80
       targetPort: 80
     selector:
       app: frontend
   EOF
   ```

3. **Verify Deployment**:
   - Check pods:
     ```bash
     kubectl get pods
     ```
     - Expected: `postgres-pod`, `backend-xxx`, `frontend-xxx` in `Running` state.
   - Check services:
     ```bash
     kubectl get services
     kubectl get services -o wide
     ```
     - Expected: `postgres-service`, `backend-service`, `frontend-service` with ClusterIPs.

---

### Step 7: Test Connectivity with Telepresence

Ensure Telepresence proxies traffic to all components.

1. **Reconnect Telepresence** (if disconnected):
   ```bash
   telepresence connect
   telepresence status
   ```

2. **Test Services Locally**:
   - Backend:
     ```bash
     curl http://backend-service
     ```
     - Expected: NGINX welcome page.
   - Frontend:
     ```bash
     curl http://frontend-service
     ```
     - Expected: NGINX welcome page.

---

### Step 8: Run the External File

Execute the script to interact with all components simultaneously.

1. **Run the Script**:
   ```bash
   python3 external_file.py
   ```
   - **Expected Output** (if successful):
     ```
     Connected to postgres-service successfully!
     Backend-service responded: <!DOCTYPE html><html><head><title>Welcome to nginx!...
     Frontend-service responded: <!DOCTYPE html><html><head><title>Welcome to nginx!...
     ```
     - Repeats every 5 seconds.
   - **Possible Errors**:
     - `Error connecting to postgres-service`: Check pod status or credentials.
     - `Error with HTTP requests`: Ensure backend/frontend pods are running.

2. **Verify Cluster State**:
   ```bash
   kubectl get pods
   kubectl get ns
   kubectl describe node
   ```

---

## How It Works

- **Telepresence Proxying**: 
  - `telepresence connect` sets up a Virtual Network Interface (VIF), allowing your local machine to access cluster services (`postgres-service`, `backend-service`, `frontend-service`) via their ClusterIPs.
  - The script’s requests are proxied to the cluster, enabling simultaneous interaction.

- **Components**:
  - **PostgreSQL**: Runs in `postgres-pod`, exposed as `postgres-service:5432`.
  - **Backend**: NGINX in `backend` deployment, exposed as `backend-service:80`.
  - **Frontend**: NGINX in `frontend` deployment, exposed as `frontend-service:80`.

- **External File**: 
  - Connects to PostgreSQL, sends HTTP requests to backend and frontend, and reports results every 5 seconds.
  - Runs locally, with Telepresence handling network communication.

---

## Achievements

- **Seamless Communication**: The script interacts with all three components without deployment.
- **Early Error Detection**: Errors (e.g., service downtime, wrong credentials) appear immediately in the terminal.
- **Multi-Component Interaction**: Simulates your original scenario with a database, backend, and frontend.

---

## Troubleshooting

- **Telepresence Issues**:
  - Re-run `telepresence connect` if disconnected.
  - Check logs: `telepresence gather-logs`.
  - Clean up stale processes: `sudo pkill -f telepresence; sudo rm -f /var/run/telepresence-daemon.socket`.

- **Cluster Issues**:
  - Verify pods: `kubectl get pods`.
  - Check services: `kubectl get services -o wide`.
  - Restart Minikube: `minikube stop; minikube start`.

- **Script Errors**:
  - PostgreSQL: Ensure credentials match (`postgres`, `mypassword`, `mydb`).
  - HTTP: Confirm `backend-service` and `frontend-service` are running.

---

## Cleanup

1. **Disconnect Telepresence**:
   ```bash
   telepresence quit
   ```

2. **Remove Cluster Components**:
   ```bash
   kubectl delete pod postgres-pod
   kubectl delete service postgres-service
   kubectl delete deployment backend
   kubectl delete service backend-service
   kubectl delete deployment frontend
   kubectl delete service frontend-service
   ```

3. **Stop Minikube**:
   ```bash
   minikube stop
   minikube delete
   ```

---

## References
- [Telepresence Architecture](https://telepresence.io/docs/reference/architecture)

# Knowledge Transfer: Securing Python Applications on Client Servers with TEEs in Kubernetes/Docker

## Objective
To deploy a Python application on a client’s server using Kubernetes/Docker, ensuring the client cannot access or reverse-engineer the source code—even with full server control—while allowing developers to manage and verify it.

---

## Proposed Solution: Trusted Execution Environments (TEEs) with Kubernetes/Docker

### Intuitive Overview: The Locked Safe Analogy
Imagine sending a secret recipe (your Python code) to a chef (the client) in a magical, locked safe (the TEE):
- **The Safe**: Built into the client’s kitchen (server CPU), it only opens with a hardware key the chef can’t touch. Your recipe stays inside, encrypted.
- **Cooking**: The chef gives the safe ingredients (data), and it cooks the dish (runs the app) inside, delivering the food (results) through a slot. The chef never sees the recipe.
- **Your Control**: You have a remote control (attestation) to check if the safe is using your exact recipe, without visiting the kitchen.

**What This Means**:
- **Client**: Can use the app but can’t peek inside the safe, even if they own the kitchen.
- **Developer**: You lock the recipe in the safe, send it via a delivery crate (Docker), and manage it with a delivery service (Kubernetes), checking it remotely.

This approach uses **TEEs**—secure zones in the CPU—to protect your app, delivered through Kubernetes/Docker.

---

## Why This Approach?
- **Client Lockout**: The safe (TEE) is hardware-locked, beyond the client’s control, even with root access.
- **Developer Access**: You control the recipe and verify it’s intact from afar.
- **Container Fit**: Builds on our Kubernetes/Docker skills, making deployment seamless.

---

## How It Works

### Simple Workflow
1. **Lock Your App**: Turn your Python code (`app.py`) into a sealed file (`app.bin`) and put it in a TEE safe (SGX enclave or SEV container).
2. **Pack It**: Place the safe in a Docker crate (container).
3. **Deliver It**: Use Kubernetes to ship and run the crate on the client’s server.
4. **Client Uses It**: They interact with the app’s outputs, but the safe stays locked.
5. **You Check It**: Use a remote signal (attestation) to confirm the app is yours, untouched.

### Client vs. Developer Access
- **Client Cannot Access**:
  - The safe is locked by the CPU. The app runs inside, and the client only sees results—not the code.
  - Even with server control (e.g., `kubectl exec`), they hit a hardware wall.
- **Developer Can Access**:
  - You start with the code, lock it in the safe, and send it over.
  - You use Kubernetes to manage it and a special check (attestation) to ensure it’s running correctly, all remotely.

---

## Technical Details

### Hardware Needed
- **Intel SGX**: CPUs like Xeon Scalable or Core i5/i7/i9 (Skylake+). Check with `lscpu | grep sgx`.
- **AMD SEV**: AMD EPYC (7001/7002/7003). Check BIOS for SEV support.
- **Requirement**: Client server must have a TEE-capable CPU, enabled in BIOS.

### Software Tools
- **Nuitka**: Turns Python into a binary (`nuitka --standalone --onefile app.py -o app.bin`).
- **Intel SGX**:
  - **Graphene-SGX**: Runs the binary in an SGX enclave.
  - **SGX SDK**: Builds the enclave (needs Linux kernel ≥ 5.11).
- **AMD SEV**:
  - **Kata Containers**: Runs the binary in an SEV-encrypted container.
  - **QEMU/KVM**: Supports SEV encryption.
- **Docker**: Packages the TEE app.
- **Kubernetes**: Deploys and manages it.

### Step-by-Step Setup
1. **Compile the App**:
   - `nuitka --standalone --onefile app.py -o app.bin`
2. **Lock It in a TEE**:
   - **SGX**: Use Graphene-SGX:
     - Configure `manifest` for `app.bin`.
     - Build: `make SGX=1` → `app.sgx`.
   - **SEV**: Use Kata Containers:
     - Install Kata: `curl -sSL <kata-install-script> | bash`.
     - Enable SEV in `configuration.toml`.
3. **Build Docker Image** (SGX Example):
   ```dockerfile
   FROM ubuntu:20.04
   RUN apt update && apt install -y libsgx-launcher
   COPY app.sgx /app/app.sgx
   COPY graphene-runtime /app/graphene-runtime
   CMD ["/app/graphene-runtime", "/app/app.sgx"]

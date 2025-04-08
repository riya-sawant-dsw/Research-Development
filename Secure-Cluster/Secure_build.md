
---

# 🔒 Secure Python App with PyArmor + Docker Multi-Stage Build

## 🎯 Objective

Protect Python source code from reverse engineering by:
- Obfuscating Python files using **PyArmor**
- Packaging only the **runtime-encrypted version** inside a **Docker multi-stage build**
- Producing a **production-ready container** that executes without exposing original code

---

## 🧱 File Structure

```
project-root/
├── app.py                 #Python script
└── Dockerfile             # Multi-stage Dockerfile
```

---

## 🐳 Dockerfile Overview

```Dockerfile
# Stage 1: Builder
FROM python:3.9-slim AS builder
WORKDIR /app

# Copy original Python script
COPY app.py .

# Install PyArmor and generate obfuscated code
RUN pip install pyarmor && \
    pyarmor gen app.py

# Stage 2: Runtime Image
FROM python:3.9-slim
WORKDIR /app

# Copy only runtime-protected files from builder
COPY --from=builder /app/dist /app

# Optional: Drop to non-root user
RUN useradd -m appuser
USER appuser

# Command to run your protected app
CMD ["python3", "app.py"]
```

---


## 🛠️ Build the Docker Image

```bash
docker build -t secure-app .
```

---

## 🚀 Run and Validate

```bash
docker run -it --rm secure-app /bin/bash
```

Inside container:

```bash
ls -la /app/
cat /app/app.py         # Should display encrypted bytecode
python3 app.py          # Should still run and produce expected output
```

---

## ✅ Result (Inside Container)

```
/app/
├── app.py                    # Encrypted bytecode with PyArmor loader
└── pyarmor_runtime_000000/   # Required runtime for execution

> Output: Processed: test_input with secret sauce
```

---

## 🔐 Outcome

- **Source code is never present** in final image
- **Obfuscated app** runs normally using PyArmor runtime
- **Secured runtime-only container** can be safely deployed in any production system

---

## 📌 Notes

- This setup uses `pyarmor gen` (not `pyarmor obfuscate`) — required for PyArmor v9+
- Trial watermark is present in output until licensed version is used
- Final image can be deployed via Docker, Kubernetes, or cloud-native services

---

## 🧰 Dependencies

- Python 3.9
- PyArmor ≥ 9.0
- Docker

---

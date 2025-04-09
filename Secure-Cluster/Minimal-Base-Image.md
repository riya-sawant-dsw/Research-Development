

# 🔐 Ultra-Secure Python App Deployment with PyArmor & Distroless Docker

## 🎯 Objective

Deploy a **Flask app** in a **production-grade container** with the following goals:

- Hide original source code using **PyArmor obfuscation**
- Use **multi-stage Docker builds** to keep the image clean
- Eliminate shell/package access via **Distroless base image**
- Produce a **hardened runtime-only container**

---

## 🧱 File Structure

```
demo-app/
├── app.py              # Flask application
├── requirements.txt    # Python dependencies
└── Dockerfile          # Multi-stage, distroless-based
```

---

## 📄 app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, this is a secure app!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

## 📦 requirements.txt

```
flask==2.3.3
```

---

## 🐳 Dockerfile

```Dockerfile
# Stage 1: Builder
FROM python:3.9-slim AS builder
WORKDIR /app

COPY app.py requirements.txt .

RUN pip install pyarmor && \
    pip install -r requirements.txt && \
    pyarmor gen app.py

# Stage 2: Runtime (Distroless)
FROM gcr.io/distroless/static-debian12

WORKDIR /app
COPY --from=builder /app/dist /app
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages

CMD ["python3", "app.py"]
```

---

## 🧪 Commands to Build and Run

### 1. Activate Conda Environment

```bash
conda activate unify-ai
```

### 2. Move to App Directory

```bash
cd /home/jrspy/dev-v1.1/test/demo-app
```

### 3. Install PyArmor & Obfuscate

```bash
pip install pyarmor
pyarmor gen app.py
```

### 4. Build the Secure Docker Image

```bash
docker build -t client-registry.example.com/secure-app:prod .
```

### 5. Test Distroless Container Shell Access (Expected to fail)

```bash
docker run -it gcr.io/distroless/python3-debian11 sh
docker run -it gcr.io/distroless/python3-debian11 bash
```

> 🔒 Both commands should fail: distroless images have **no shell**, **no exec**, **no package manager**.

---

## ✅ Expected Result

- The `app.py` file inside the container is **obfuscated bytecode**, not readable Python.
- No shell access or package installation is possible post-build.
- Flask app runs and responds on port `5000`.

---

## 🔐 Security Wins

- ✅ Source code never enters final image.
- ✅ Obfuscated Python bytecode with PyArmor.
- ✅ Minimal attack surface via Distroless image.
- ✅ Hardened production-ready image suitable for internal/private registries.

---

## 📌 Notes

- Ensure PyArmor version is compatible with Python 3.9 and Flask bytecode.
- PyArmor trial adds a watermark; use a licensed version for clean outputs.
- You **must** copy `site-packages` if dependencies are needed at runtime.

---

# NVIDIA Triton Inference Server – Beginner Notes (3-Month Learning Plan, Day 3)

This document explains the **end-to-end process of model inference deployment using Triton Inference Server**, with answers to doubts I had while learning. I use the example of a **simple web application** connecting to Triton.

---

## 🔹 1. Get the Model Artifact

* You start with a **pre-trained model** in a supported format: ONNX, TensorFlow SavedModel, PyTorch TorchScript, or TensorRT plan.
* Example: `resnet50.onnx`
* Models can come from:

  * Your ML team (trained elsewhere).
  * Public sources like ONNX Model Zoo or Hugging Face Hub.

👉 **Key point:** Inference always starts with a **trained model artifact**, not raw training code.

---

## 🔹 2. Deploy Triton Server (via Docker)

Pull the Triton container image:

```bash
docker pull nvcr.io/nvidia/tritonserver:23.10-py3
```

Run Triton with GPU access and expose ports:

```bash
docker run --gpus=all -p 8000:8000 -p 8001:8001 -p 8002:8002 \
  -v $(pwd)/models:/models \
  nvcr.io/nvidia/tritonserver:23.10-py3 tritonserver --model-repository=/models
```

👉 **Why Docker?**

* Ensures the model + Triton + dependencies run the same on any machine.
* Avoids “it works on my machine” issues.
* Provides **GPU support** via NVIDIA Container Runtime.
* Makes the deployment **portable** (local, cloud, edge, Kubernetes).

---

## 🔹 3. Organize the Model Repository

Triton expects a specific folder structure:

```
models/
└── resnet50/
    ├── 1/
    │   └── model.onnx
    └── config.pbtxt
```

* `1/` = model version folder.
* `config.pbtxt` = tells Triton model name, inputs/outputs, batch size, datatypes, etc.

👉 **Permissions & Ownership:**

* Use `ls -l` to check file access.
* If Triton fails with *permission denied* errors:

  * `chmod 644 config.pbtxt` → make file readable.
  * `sudo chown -R $USER:$USER models/` → fix ownership if files belong to `root`.

---

## 🔹 4. Configure & Optimize

* Edit `config.pbtxt` to enable batching, set input shapes, outputs, precision, etc.
* Convert models to **TensorRT (.plan)** for faster GPU inference.
* Add **ensembles** → chain preprocessing → model → postprocessing inside Triton.

👉 **Why Triton vs simple web app?**

* A simple web app can serve one model directly (CPU or GPU) using framework libraries.
* Triton is required when you need multiple models, batching, GPU efficiency, or scaling.

---

## 🔹 5. Serve the Model

Triton opens these ports:

* **8000** → HTTP endpoint
* **8001** → gRPC endpoint
* **8002** → Metrics (Prometheus)

Example inference request:

```bash
curl -s http://localhost:8000/v2/models/resnet50/versions/1/infer \
  -d @input.json -H "Content-Type: application/json"
```

👉 **Networking tools:**

* `ss -tulnp | grep 8000` → check if port 8000 is open.
* `curl` → test Triton health (`/v2/health/ready`) and inference.

---

## 🔹 6. Monitor & Scale

* **Monitor:** Use Prometheus/Grafana with metrics from port 8002.
* **Profile:** Use `perf_analyzer` to test throughput and latency.
* **Scale:**

  * Run multiple Triton instances.
  * Use Kubernetes/Load Balancers for large deployments.

👉 **Cloud vs Triton:**

* **Simple case:** inference can happen directly in cloud endpoints (AWS, Azure, GCP) — enough for basic web apps.
* **Advanced case:** use Triton when you need multi-model, multi-framework, GPU optimization, or on-prem/edge deployments.

---

## 🔹 Why I am Using RunPod

* **What it is:** RunPod is a cloud GPU provider that offers access to powerful NVIDIA GPUs (A100, H100, 3090, 4090, etc.) on demand.
* **Why I use it:**

  * My local machine may not have a strong GPU.
  * Training and inference (especially with Triton) need CUDA-enabled GPUs.
  * RunPod gives me affordable, instant GPU access without buying expensive hardware.
  * It provides pre-built environments (PyTorch, TensorFlow, Jupyter, Triton-ready).
* **When I use it:**

  * For practice and learning with Triton Inference Server.
  * For workloads that require GPU acceleration but don’t justify owning a dedicated GPU.
  * To run experiments and benchmarking (e.g., with `perf_analyzer`) in a production-like setup.

👉 **Simple analogy:** RunPod is like renting a supercar 🏎️ instead of buying one. I get GPU power only when I need it, pay as I go, and can run Triton experiments in the cloud.

---

## ✅ Summary

1. Get pre-trained model artifact (ONNX, TF, PyTorch, TensorRT).
2. Deploy Triton in Docker → ensures consistency, portability, and GPU support.
3. Organize model repo with version folders + `config.pbtxt`.
4. Configure & optimize → batching, TensorRT, ensembles.
5. Serve model via HTTP/gRPC on ports 8000/8001.
6. Monitor & scale → Prometheus, perf\_analyzer, Kubernetes.

👉 **Inference flow:** Simple web app → sends request → Triton serves model → response returned.

# Linux + Triton Inference Server Command Cheatsheet

A practical, GitHub‑ready guide to essential Linux commands you’ll actually use when working with **NVIDIA Triton Inference Server** — with copy‑paste examples.

> **Tip:** Most paths in Triton containers live under `/opt/tritonserver`.

---

## Table of Contents

* [File & Directory Management](#file--directory-management)
* [File Viewing](#file-viewing)
* [Minimal File Editing (optional but useful)](#minimal-file-editing-optional-but-useful)
* [System Information](#system-information)
* [Process Management](#process-management)
* [Networking (Ports 8000/8001/8002)](#networking-ports-800080018002)
* [Search & Filters](#search--filters)
* [Permissions & Ownership](#permissions--ownership)
* [Disk Usage & Storage](#disk-usage--storage)
* [Docker Basics for Triton](#docker-basics-for-triton)
* [Logs & Debugging](#logs--debugging)
* [Model Repository Structure](#model-repository-structure)
* [Quick Reference](#quick-reference)

---

## File & Directory Management

Common tasks when navigating Triton’s model repository and binaries.

```bash
# Show absolute path (useful to confirm where you are)
pwd

# List files (non-hidden)
ls

# List everything, long format, human sizes
ls -alh

# Change directory to Triton install path (inside container)
cd /opt/tritonserver

# Jump to model repo
cd /opt/tritonserver/models

# Create a new model directory and a version folder
mkdir -p resnet50/1

# Copy a model file into the version folder (example)
cp ~/Downloads/resnet50_fp32.plan resnet50/1/model.plan

# Move or rename a model
o mv resnet50 my_resnet50

# Remove a file / folder (careful!)
rm config.pbtxt
rm -r my_resnet50
```

**Triton context:** Model repos follow `model_name/<version>/model.*` with an optional `config.pbtxt` in `model_name/`.

---

## File Viewing

Fast ways to peek at configs and model metadata.

```bash
# Print entire config
cat config.pbtxt

# First / last lines (quick sanity checks)
head -n 20 config.pbtxt
tail -n 20 config.pbtxt

# Page through long files (q to quit)
less config.pbtxt

# Show directory tree of models (if tree is installed)
tree -L 2 /opt/tritonserver/models
```

**Triton context:** Use these to verify inputs/outputs, `max_batch_size`, and `backend`/`platform` in `config.pbtxt`.

---

## Minimal File Editing (optional but useful)

Often you’ll tweak `config.pbtxt` or compose files.

```bash
# Open and edit config with nano (simple)
nano config.pbtxt

# Or with Vim (advanced)
vi config.pbtxt
```

> If you prefer not to edit inside containers, mount configs from the host via Docker bind mounts (see Docker section).

---

## System Information

Check that the environment (especially GPUs) is healthy.

```bash
# Kernel & distro basics
uname -a

# CPU & memory snapshot
top      # or: htop
free -h

# Disk space (host or container filesystem)
df -h

# GPU status and drivers (requires NVIDIA container runtime)
nvidia-smi
```

**Triton context:** Confirm GPUs are visible (`nvidia-smi`) before starting Triton with GPU backends.

---

## Process Management

Find, inspect, and manage processes.

```bash
# List processes containing the word 'triton'
ps aux | grep -i triton

# Kill a process by PID (careful)
kill -9 <PID>
```

**Docker variant** (preferred for Triton): see [Docker Basics](#docker-basics-for-triton).

---

## Networking (Ports 8000/8001/8002)

Triton defaults: **HTTP 8000**, **gRPC 8001**, **Metrics 8002**.

```bash
# Show listening sockets (modern)
ss -tulnp | grep -E "8000|8001|8002"

# Legacy alternative
netstat -tulnp | grep -E "8000|8001|8002"

# Quick health probe (HTTP)
curl -s http://localhost:8000/v2/health/ready && echo "\nREADY"

# List loaded models via HTTP API
curl -s http://localhost:8000/v2/models | jq .
```

**Triton context:** Use these to verify the server is listening and responding.

---

## Search & Filters

Hunt for errors in logs and validate configs.

```bash
# Case-insensitive search for errors in log
grep -i "error" triton.log

# Only show lines with 'input' or 'output' in config
grep -nE "input|output" config.pbtxt

# Find all config.pbtxt files under models/ (host or container)
find /opt/tritonserver/models -name config.pbtxt
```

---

## Permissions & Ownership

Fix common permission issues when mounting host directories.

```bash
# Show long listing with ownership
ls -l

# Change owner recursively to current user (host side)
sudo chown -R "$USER":"$USER" ~/model-repo

# Make a file executable (example: custom backend binary)
chmod +x my_backend.so
```

**Triton context:** When bind-mounting model repos, ensure the container user can read the files.

---

## Disk Usage & Storage

Know what’s taking space.

```bash
# Size of entire model repo (human readable)
du -sh /opt/tritonserver/models

# Size per model (sorted)
du -sh /opt/tritonserver/models/* | sort -h
```

---

## Docker Basics for Triton

The most common commands to run and inspect Triton containers.

```bash
# Pull a Triton image (replace with a specific tag if needed)
docker pull nvcr.io/nvidia/tritonserver:23.10-py3

# Run Triton with a host model repo (read-only), expose ports
# NOTE: requires NVIDIA Container Toolkit for GPU access

docker run --gpus=all \
  -p 8000:8000 -p 8001:8001 -p 8002:8002 \
  -v $(pwd)/models:/models:ro \
  --name triton \
  nvcr.io/nvidia/tritonserver:23.10-py3 \
  tritonserver --model-repository=/models

# List running containers
docker ps

# Stream logs from the Triton container
docker logs -f triton

# Exec into the container shell
docker exec -it triton /bin/bash

# Stop & remove the container
docker stop triton && docker rm triton
```

**Triton context:** Bind-mount your model repo to iterate quickly on `config.pbtxt` without rebuilding images.

---

## Logs & Debugging

See what Triton is doing during startup and model loading.

```bash
# Inside container: write logs to a file (example run)
tritonserver --model-repository=/models > triton.log 2>&1

# Tail the last 50 lines
tail -n 50 triton.log

# Follow logs in real time
tail -f triton.log

# With Docker (container named 'triton')
docker logs -f triton
```

Common things to look for: port binds, backend initialization, model load success/failure, and shape/type mismatches.

---

## Model Repository Structure

Typical layout you’ll see and create.

```
/opt/tritonserver/models/
├── resnet50/
│   ├── 1/
│   │   └── model.plan
│   └── config.pbtxt
└── bert/
    ├── 1/
    │   └── model.onnx
    ├── 2/
    │   └── model.onnx
    └── config.pbtxt
```

**Quick checks:**

```bash
# Verify a model has a config
ds ls -l */config.pbtxt

# Recursively list models & versions
ls -R /opt/tritonserver/models
```

---

## Quick Reference

A compact set of commands you’ll use daily.

```bash
# Where am I? (absolute path)
pwd

# What’s here?
ls -alh

# Peek at config
head -n 20 config.pbtxt

# Is Triton listening?
ss -tulnp | grep -E "8000|8001|8002"

# Is GPU visible?
nvidia-smi

# Disk space & model sizes
df -h
du -sh /opt/tritonserver/models/* | sort -h

# Container lifecycle
docker ps
docker logs -f triton
docker exec -it triton /bin/bash
docker stop triton && docker rm triton
```

---

### License

MIT (feel free to copy/paste this into your GitHub repo README)

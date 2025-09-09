# Week 1 — Day 1 Setup
# Week 1 — Day 1: RunPod Setup and GitHub Connection

## ✅ Tasks Completed Today

### 1. Created RunPod Account
- Registered on [RunPod](https://www.runpod.io/) to rent cloud GPUs because my local laptop only has Intel UHD graphics.
- NVIDIA Triton Inference Server requires an NVIDIA GPU, so a cloud GPU is essential.

---

### 2. Launched First Pod (RTX 3090)
- Selected template: **PyTorch 2.8.0 + CUDA 12.8 + Ubuntu 22.04**  
- Hardware:  
  - **1x RTX 3090 (24GB VRAM)**  
  - **125GB RAM, 16 vCPUs, 80GB Disk**

The pod works as my personal Linux-based development environment.

---

### 3. Configured HTTP vs TCP Ports
- **HTTP Ports (via Cloudflare proxy):**  
  - Port `8000` → Triton REST API  
  - Port `8888` → Jupyter Notebook  
- **TCP Ports:**  
  - Port `22` → SSH access  

🧠 Learned that:
- **HTTP Proxy** gives secure HTTPS links (easy to test from browser/Colab).  
- **TCP Ports** give direct raw access (useful for advanced clients like gRPC or metrics).  

---

### 4. Understood Cloudflare Proxy and Load Balancer
- **Cloudflare proxy** is in the middle for:  
  - Automatic HTTPS encryption  
  - Global routing + DDoS protection  
- **Load balancer** distributes requests across pods and keeps services available if one pod fails.  

---

### 5. Checked Preinstalled Tools
I confirmed what software was already available in my RunPod pod.

**Commands used:**
```bash
git --version          # Git
python3 --version      # Python
pip --version          # Pip
nvcc --version         # CUDA compiler
nvidia-smi             # NVIDIA GPU info
lsb_release -a         # Ubuntu version

or

dpkg -l | less

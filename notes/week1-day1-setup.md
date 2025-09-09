# Week 1 — Day 1: RunPod Setup and GitHub Connection

## ✅ Tasks Completed Today

---

### 1. Created RunPod Account
I registered on RunPod to rent cloud GPUs because my local laptop only has Intel UHD graphics.  
NVIDIA Triton Inference Server requires an NVIDIA GPU, so a cloud GPU is essential.

---

### 2. Launched First Pod (RTX 3090)
I launched my first pod with the following configuration:  
- Template: PyTorch 2.8.0 + CUDA 12.8 + Ubuntu 22.04  
- Hardware: 1x RTX 3090 (24GB VRAM), 125GB RAM, 16 vCPUs, 80GB Disk  

This pod serves as my personal Linux-based development environment with GPU acceleration.

---

### 3. Configured HTTP vs TCP Ports
I set up the networking for the pod.  
- HTTP Ports: 8000 for Triton REST API, 8888 for Jupyter Notebook  
- TCP Ports: 22 for SSH access  

I learned that HTTP Proxy provides secure HTTPS links which are easy to test from the browser or Colab, while TCP Ports give direct raw access useful for advanced clients like gRPC or metrics.

---

### 4. Understood Cloudflare Proxy and Load Balancer
I studied how networking works inside RunPod. Two important components are **Cloudflare Proxy** and the **Load Balancer**.

#### Cloudflare Proxy
- Acts as a middle layer between my pod and the outside world.  
- Provides **automatic HTTPS encryption**, so even if my service runs on plain HTTP, the outside world sees a secure HTTPS connection.  
- Handles **global routing** through Cloudflare’s worldwide network, which helps reduce latency for users connecting from different regions.  
- Adds **DDoS protection**, filtering out malicious traffic before it reaches my pod.  
- Has a **100-second timeout limit** for HTTP connections. This means if my service does not respond within 100 seconds, the connection is closed automatically.

#### Load Balancer
- Distributes **incoming traffic** across multiple pods (if I scale up to more than one).  
- Ensures **high availability**: if one pod goes down, the load balancer redirects requests to a healthy pod.  
- Helps with **scalability**: when demand increases, more pods can be launched and traffic spread across them.  
- Improves **performance** by avoiding bottlenecks, since no single pod is overloaded.  

🧠 From this, I learned that Cloudflare Proxy focuses on **security and global access**, while the Load Balancer ensures **reliability and scalability** of services running inside RunPod.  

### 🌐 Connection Flow Diagram (Horizontal)

User (Browser/API Client) 
   ──▶ 🌍 Cloudflare Proxy (HTTPS + Security + Global Routing) 
   ──▶ ⚖️ RunPod Load Balancer (Distributes Traffic + Reliability) 
   ──▶ 🚀 My Pod (AI Service on Ports: 8000 REST | 8001 gRPC | 8002 Metrics | 8888 Jupyter)

---

### 5. Checked Preinstalled Tools
I verified what software was already installed in my RunPod pod.  
These included Git, Python, Pip, CUDA, NVIDIA GPU drivers, and Ubuntu packages.  
This step helped me confirm the environment was ready for running Triton Inference Server.

---

### 6. GitHub SSH Setup
I connected my RunPod pod to GitHub using SSH so I don’t need to use a Personal Access Token (PAT) every time.  
The steps included cloning or creating the repository folder, moving into it, switching the remote URL from HTTPS to SSH, verifying the remote connection, testing the SSH connection to GitHub, and finally making a test commit to confirm that everything worked successfully.  
After this setup, pushing changes to GitHub worked without asking for a PAT.

---

## ⚠️ Difficulties I Faced on Day 1

### 1. No NVIDIA GPU in my Laptop
My local machine only has Intel UHD graphics, so I could not run NVIDIA Triton Inference Server directly.  
I solved this by using a RunPod cloud GPU (RTX 3090).

---

### 2. AVX Instruction Set Not Supported on My CPU
When I tried pulling NVIDIA Triton locally, I got an error because my CPU (Intel Celeron) does not support AVX instructions.  
I solved this by moving everything to the cloud (RunPod).

---

### 3. Confusion with HTTP vs TCP Ports
At first, I was not sure why Triton needs ports 8000, 8001, and 8002, and how Cloudflare proxy and TCP mapping work.  
I solved this by learning that 8000 is for REST API, 8001 is for gRPC, and 8002 is for metrics, while Cloudflare proxy provides HTTPS but has a timeout.

---

### 4. GitHub Authentication Issue (PAT Prompt)
My repository was originally set to HTTPS remote, so every push asked for a Personal Access Token (PAT).  
I solved this by switching to SSH setup with keys, so pushes now work without asking every time.

## 📜 Commands Used

```bash
# Step 5: Check preinstalled tools
git --version
python3 --version
pip --version
nvcc --version
nvidia-smi
lsb_release -a
dpkg -l

# Step 6: GitHub SSH setup
git clone git@github.com:Tankasala25/triton-runpod-learning.git
cd triton-runpod-learning
git remote set-url origin git@github.com:Tankasala25/triton-runpod-learning.git
git remote -v
ssh -T git@github.com
git add .
git commit -m "Connected through SSH setup"
git push origin main

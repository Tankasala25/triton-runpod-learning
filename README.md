# triton-runpod-learning
Learning NVIDIA Triton Inference Server on RunPod GPUs — 3 month AI Inference Engineer roadmap

Perfect 👍 — I’ll insert your Step 2 (Go into the repo) right after Step 1 in the same file.
Here’s the updated, clean, single-file version:

# GitHub SSH Setup from RunPod

This file documents the complete process I followed to connect my RunPod pod to GitHub using SSH.  
With SSH setup, I don’t need to use a Personal Access Token (PAT) when pushing code.

---

## Steps I Followed

### 1. Clone or create the repo folder
I cloned my GitHub repository directly with SSH.
```bash
git clone git@github.com:Tankasala25/triton-runpod-learning.git

2. Go into the repo

I moved into the repository folder.

cd triton-runpod-learning

3. Switch the remote URL from HTTPS to SSH

I updated the remote URL so GitHub uses SSH instead of HTTPS.

git remote set-url origin git@github.com:Tankasala25/triton-runpod-learning.git

4. Verify the remote is SSH

I checked the remote configuration.

git remote -v


Expected output:

git@github.com:Tankasala25/... (fetch)
git@github.com:Tankasala25/... (push)

5. Test SSH connection

I tested my SSH connection to GitHub.

ssh -T git@github.com


On first run it asked:

Are you sure you want to continue connecting (yes/no/[fingerprint])?


I typed yes.

Result:

Hi Tankasala25! You've successfully authenticated, but GitHub does not provide shell access.

6. Make a test commit to confirm SSH works

Finally, I made a test commit and pushed it to confirm SSH was working.

git add .
git commit -m "Connected through SSH setup"
git push origin main


✅ The commit was pushed successfully without asking for a PAT or password.

Key Takeaway

Using SSH keys makes GitHub access easier and more secure in RunPod.
Now I can run git push and git pull without entering tokens or passwords.

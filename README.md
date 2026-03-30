# Fedora issue #124: assignment submission document for the issue on learning how RamaLama makes working with AI boring.

As an Outreachy contributor for the **Fedora Project**, this task focused on setting up **RamaLama**, an open-source tool designed to make local AI development simple and "boring." 

The task was to install RamaLama on a Fedora environment, manage AI models across multiple transports (Ollama, HuggingFace, OCI), and verify model performance with Fedora-specific prompts. This serves as the foundation for building a **Retrieval-Augmented Generation (RAG)** system based on Fedora RPM Packaging Guidelines.

---

## Environment Setup

To maintain a clean and reproducible environment, I utilized a Fedora-powered Virtual Machine isolated from my local system.

### 1. Multipass VM Initialization
I used the `multipass` CLI to relaunch my previously installed Fedora 43 Cloud instance. 

```bash
# Check existing VMs
multipass list

# Login to the dedicated Fedora environment
multipass shell fedora-om26
```
Once inside the VM, I cloned the project repository and initialized a dedicated Python virtual environment to manage dependencies safely.

```bash
# Clone the repository
git clone git@github.com:gtfrans2re/Fedora-RamaLama-RAG-Om26.git
cd Fedora-RamaLama-RAG-Om26/

# Create and activate a virtual environment
python3 -m venv ramalama_env
source ramalama_env/bin/activate

# Upgrade core build tools
pip install --upgrade pip setuptools wheel
```

3. RamaLama Installation
The installation was performed using the Fedora native package manager to ensure full system compatibility.
```bash
sudo dnf install ramalama -y

# Verify installation
ramalama version
# Output: ramalama version 0.17.1
```

---

## Knowledge Base Preparation
To prepare for future RAG tasks, I curated a data/ directory containing the official Fedora Packaging Guidelines.
```bash
mkdir data
curl -o data/fedora-versioning.pdf [https://fedorapeople.org/~tmz/guidelines/packaging-guidelines/Versioning/Versioning.pdf](https://fedorapeople.org/~tmz/guidelines/packaging-guidelines/Versioning/Versioning.pdf)

# Track files in Git
git add data/
git commit -m "Add: official guidelines file."
git push origin main
```

---
## Model Management & Performance Testing

I successfully pulled and tested **four distinct models** across different transports.

### Successful Models

| # | Model Name       | Transport | Status  |
|---|----------------|----------|---------|
| 1 | **TinyLlama**   | `ollama://` | Success |
| 2 | **Granite-7b-lab** | `hf://`     | Success |
| 3 | **Gemma-2**     | `ollama://` | Success |
| 4 | **Tiny-Vicuna-1B** | `hf://`     | Success |

### Example Command (Step 4 Verification):
```bash
ramalama run tinyllama "What are the Four Foundations of the Fedora project?"
```

## Troubleshooting & Lessons Learned

Part of the experience involved navigating the constraints of a VM and registry permissions.

1. **Registry Authorization Issues (OCI/Quay.io)**

   **Error:** `unauthorized: access to the requested resource is not authorized.`

   - **Reason:** Directly pulling from `quay.io/ramalama/gemma-2-9b` failed due to permission constraints on that specific registry path.
   - **Solution:** I pivoted to the verified `ollama://` transport for Gemma-2, which pulled successfully.

2. **Registry 404 & Transport Stability (ModelScope)**

   **Error:** `HTTP Error 404: Not Found.`

   - **Reason:** The `ms://` (ModelScope) transport encountered path resolution issues for the Phi-3 model.
   - **Solution:** To maintain progress, I focused on HuggingFace and Ollama transports, which offered higher stability for the GGUF models in this environment.

3. **Resource Constraints (Disk Space)**

   **Error:** `OSError: [Errno 28] No space left on device.`

   - **Reason:** Attempting to pull multiple large models (like DeepSeek-R1) exceeded the VM's 50GB disk allocation.
   - **Solution:** I focused on Tiny-Vicuna-1B for the final verification, which allowed for a successful "end-to-end" test within the remaining storage limits.

4. **GPU/Container Health Checks**

   **Error:** `health check of container ... timed out after 20 seconds.`

   - **Reason:** Occurred when trying to run very large models (20B parameters) on a CPU-only VM environment.
   - **Solution:** I chose models sized between 1B and 7B parameters to ensure smooth performance on the available 4 CPUs and 8GB of RAM.

---

## Final Status

The Fedora environment is now fully configured with RamaLama. I have successfully demonstrated the ability to:

- Deploy a Fedora VM via Multipass  
- Manage a Python 3.14 virtual environment  
- Pull and run AI models across diverse registries  
- Troubleshoot real-world infrastructure errors (Disk I/O, Registry 404s, and Permissions)

**Next Step:** Moving toward indexing the `data/` guidelines into a local vector store using `ramalama rag`.

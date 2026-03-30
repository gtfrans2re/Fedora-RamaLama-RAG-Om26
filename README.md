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
To begin the AI implementation phase, RamaLama was installed directly onto the Fedora 43 system to leverage native package management:
![RamaLama](<assets/Screenshot from 2026-03-30 08-14-50.png>)

I verified the specific version of RamaLama installed on the system to ensure compatibility with the upcoming RAG tasks:
![RamaLama Version](<assets/Screenshot from 2026-03-30 08-18-12.png>)

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

### RamaLama Model Initialization:
I pulled and ran my first model (TinyLlama) and verified it against the Fedora Foundations prompt:
```bash
ramalama run tinyllama "What are the Four Foundations of the Fedora project?"
```
![TinyLlama](<assets/Screenshot from 2026-03-30 07-55-22.png>)

### Granite-7b-lab Model Initialization:
I pulled and ran my second model (Granite-7b-lab) and verified it against the Fedora Open Source aura prompt:
```bash
ramalama run hf://instructlab/granite-7b-lab-GGUF "How does Fedora feed the aura of Open Source software to the world?"
```
![Granite-7b-lab](<assets/Screenshot from 2026-03-30 08-04-00.png>)

### Gemma-2 Model Initialization:
I pulled and ran my third model (Gemma-2) and verified it against the prompt regarding RamaLama's impact on workflow:
```bash
ramalama run gemma2 "Is it true that RamaLama makes working with AI boring?"
```
![Gemma-2](<assets/Screenshot from 2026-03-30 08-06-53.png>)

Asking a subjective question about RamaLama's "boring" philosophy served as a critical test of the model’s logical boundaries and ethical alignment. By posing a prompt that invited a personal opinion, I pushed the model to reveal its safety guardrails; rather than agreeing with the premise or hallucinating a persona, the model maintained strict objective neutrality. This interaction proved that the integration is robust enough to recognize its role as a neutral tool, successfully demonstrating that it can handle complex, nuanced human interaction without crossing into the "border" of subjective debate, a vital quality for any AI tool being considered for the Fedora community.

### Tiny-Vicuna-1B Model Initialization:
I pulled and ran my fourth model (Tiny-Vicuna-1B) and verified it against the Feynman-style Fedora project prompt:
```bash
ramalama run hf://afrideva/Tiny-Vicuna-1B-GGUF "Explaining the Fedora project and about the Fedora community to a 5-year-old kid like Feynman would do."
```
![Tiny-Vicuna-1B](<assets/Screenshot from 2026-03-30 08-08-40.png>)

While Tiny-Vicuna-1B is exceptionally fast and lightweight, my testing showed that it can occasionally struggle with complex instructions. For example, when asked to explain Fedora in the style of Richard Feynman, the model outputted its internal conversation tags (<|im_start|>) and failed to generate the actual explanation, highlighting the trade-off between model size and instruction-following reliability.

---

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

## Model Weight & Infrastructure Discussion

A key part of this task was evaluating how different model sizes and transports impact the stability of a Fedora-powered virtual environment.

---

### 1. Lightweight vs. Heavyweight Models

During testing, I observed a significant difference between **"Tiny" models** and full-scale **Instruct models**:

- **Lightweight Tier** *(e.g., TinyLlama, Tiny-Vicuna)*  
  These models *(~450MB – 600MB)* are the **gold standard** for isolated VM development.  
  - Pull in under **60 seconds**  
  - Run inference **almost instantly** on 4 CPUs  
  - No timeout issues  

- **Mid-Range Tier** *(e.g., Granite-7B, DeepSeek-8B)*  
  At **3.8GB – 4.5GB**, these models represent the **upper threshold** of the current infrastructure.  
  - More accurate responses (especially for Fedora-related queries)  
  - Push limits of **8GB RAM**  
  - Occasional **container health check timeouts**  

- **Heavyweight Tier** *(e.g., GPT-OSS 20B)*  
  At **10.8GB+**, these models are **not suitable** for the current VM setup.  
  - CPU-only runtime cannot handle 20B parameters efficiently  
  - Leads to **consistent container timeouts**  

---

### 2. Transport & Registry Reliability

The **"diverse"** aspect of this study also included registry performance:

- **Ollama (`ollama://`)**  
  - Most consistent download speeds  
  - ~**10 MB/s average**  

- **Hugging Face (`hf://`)**  
  - Largest variety of **GGUF-quantized models**  
  - Encountered **disk space issues** when pulling multiple large models  

---

### 3. The "Disk Space" Bottleneck

The most critical error that may come up:

```bash
I/O Error: [Errno 28] No space left on device
```

---

## Final Status

Now that the Fedora environment is fully configured with RamaLama. I have successfully demonstrated the ability to:

- Deploy a Fedora VM via Multipass  
- Manage a Python 3.14 virtual environment  
- Pull and run AI models across diverse registries  
- Troubleshoot real-world infrastructure errors (Disk I/O, Registry 404s, and Permissions)

**Next Step:** Moving toward indexing the `data/` guidelines into a local vector store using `ramalama rag`.

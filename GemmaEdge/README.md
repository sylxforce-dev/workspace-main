# GEMMAEDGE V1.4 — SOVEREIGN SETUP & TROUBLESHOOTING GUIDE

**Project:** GemmaEdge (Local AI Reasoning Engine)
**Hardware Target:** NVIDIA GPU Only (GTX 1050 Ti 4GB Optimized)
**Base Logic:** Reasoning over Reflex™

---

## 0. ENVIRONMENT SETUP (The Isolated Base)
To prevent library conflicts and "Dependency Drift," ALWAYS use a Virtual Environment (venv).

1.  **Create Environment:**
    ```cmd
    python -m venv .venv
    ```
2.  **Activate Environment:**
    *   **Windows:** `.venv\Scripts\activate`
    *   **Linux/Mac:** `source .venv/bin/activate`
3.  **Install Dependencies:**
    ```cmd
    pip install -r requirements.txt
    ```

> ⚠️ **NOTE:** The `requirements.txt` uses `--index-url` to pull the CUDA 12.1 version of PyTorch directly from PyTorch servers instead of PyPI. This is required for GPU support. Do not remove this line.

---

## 1. DIAGNOSTIC TOOLS (The Safety Net)
Before starting the engine, use the `guide_scripts/` to verify your hardware-software alignment. This prevents 99% of setup errors.

*   **Step A:** Run `python guide_scripts/check_system.py`
    *(Verifies .venv, CUDA detection, and Torch GPU support.)*
    *Expected:* `CUDA Available: True | GPU: NVIDIA GeForce GTX XXXX`
    *If you see `CUDA Available: False` — your torch version is wrong. Reinstall using the `--index-url` command inside requirements.txt.*

*   **Step B:** Run `python guide_scripts/find_nvml_auto.py`
    *(Locates nvml.dll for you.)*
    *Expected:* `Found nvml.dll at C:\Windows\System32\nvml.dll`

*   **Step C:** Run `python guide_scripts/test_nvml_link.py`
    *(Confirms the Python-to-GPU telemetry bridge is active.)*
    *Expected:* `GPU Telemetry Active | VRAM: X.XX GB | Temp: XX C`

---

## 🛑 2. CONFIGURATION BARRIER (UPDATE config.yaml)
⚠️ **CRITICAL STEP:** Do not execute any further terminal commands or compilation scripts until you complete this step. The engine cannot automatically detect where your files are stored. You MUST define your local paths first.

Open the `config.yaml` file in your project root using any text editor and update the following configuration rules:

1.  **MiniLM Router Path**
    Locate the `engine` block and map your absolute snapshot folder path:
    ```yaml
    engine:
      model_path: "YOUR_ABSOLUTE_PATH_TO_MINILM_SNAPSHOT_FOLDER"
    ```

2.  **Modelfile Storage Path**
    Locate the `model_storage` block and define the root path where your compilation targets live:
    ```yaml
    model_storage:
      root_path: "YOUR_ABSOLUTE_PATH_TO_GemmaEdge/engineModels"
    ```

3.  **Router Threshold Tuning**
    Ensure the active threshold matches your target performance parameters (Default matrix setup: 0.30):
    ```yaml
    router:
      threshold: 0.30
    ```

4.  **Hardware limits (match to your GPU):**
    ```yaml
    hardware:
      gpu_vram_limit: 4.0   # <- change this to match your GPU VRAM
      keep_alive: 0         # <- keep at 0 for cold-swap VRAM purge
    ```

**Save the `config.yaml` file and proceed below.**

---

## 3. STARTUP SEQUENCE (Strict Hierarchy)
The system must be started in this specific order.

### STEP 1: DOWNLOAD THE BASE MODEL (one time only)
GemmaEdge runs on Gemma 4 2B in GGUF format (Q4_K_M quantization). Download the model file from HuggingFace:
*   **Model:** `gemma-4-E2B-it-Q4_K_M.gguf`
*   **Source:** [unsloth/gemma-4-E2B-it-GGUF](https://huggingface.co)

Place the downloaded file in your project's `engineModels` folder.

> ⚠️ **NOTE:** You can swap any compatible GGUF model here. Update the filename in each Modelfile if you use a different model: `FROM "./your-model-name.gguf"`. This is the framework's model-agnostic design — use what fits your hardware.

### STEP 2: COMPILE THE MODELFILES (one time only)
Each specialist module is compiled from its Modelfile.

#### 💡 HOW TO NAVIGATE IN WINDOWS CMD
Navigate to your project's `engineModels` folder in CMD. If your project is on a different drive, change the drive letter first:

```cmd
X:
cd your-project-folder\engineModels
```
*(Replace `X:` with your actual drive letter and `your-project-folder` with your actual path.)*

#### 🚀 QUICK COMPILE LIST (Copy & Paste)
Once inside the `engineModels` folder, paste this block to compile all 6 modules:

```cmd
ollama create misha-maincore   -f Misha_MainCore.Modelfile
ollama create misha-auditor    -f Misha_Auditor.Modelfile
ollama create misha-oracle     -f Misha_Oracle.Modelfile
ollama create misha-social     -f Misha_Social.Modelfile
ollama create misha-reasoning  -f Misha_Reasoning.Modelfile
ollama create misha-sovereign  -f Misha_Sovereign.Modelfile
```

Verify all 6 are compiled:
```cmd
ollama list
```
*Expected:* All 6 `misha-*` models appear in the list.

### STEP 3: BOOT THE SYSTEM
Open 3 separate terminals and run each command in its own terminal, in this order:

**Terminal 1 — Engine Warmup**
```cmd
python run_boot.py
```

**Terminal 2 — Backend API**
```cmd
python run_api.py
```

**Terminal 3 — Frontend Dashboard**
```cmd
streamlit run run_dashboard.py
```

*   **Access the dashboard at:** `http://127.0.0.1:8501`
*   **API runs at:** `http://127.0.0.1:8000`

---

## SWAP YOUR OWN MODEL
GemmaEdge is model-agnostic. To use a different GGUF model:

1.  Place your GGUF file in your project's `engineModels` folder.
2.  Edit each Modelfile — change the `FROM` line:
    ```dockerfile
    FROM "./your-model-name.gguf"
    ```
3.  Navigate to the `engineModels` folder in CMD:
    ```cmd
    X:
    cd your-project-folder\engineModels
    ```
4.  Recompile all 6 modules:
    ```cmd
    ollama create misha-maincore   -f Misha_MainCore.Modelfile
    ollama create misha-auditor    -f Misha_Auditor.Modelfile
    ollama create misha-oracle     -f Misha_Oracle.Modelfile
    ollama create misha-social     -f Misha_Social.Modelfile
    ```
5.  Boot normally — follow STEP 3 above.

The routing, cold-swap, and telemetry all work the same regardless of which model you use.

---

## TUNE THE ROUTER
All routing sensitivity is controlled in `config.yaml`:
```yaml
router:
  threshold: 0.40     # <- lower = more aggressive routing
  max_weight: 0.95
  mean_weight: 0.05
```
All routing keywords are in: `router/personality_matrix.json`
Edit the JSON to add your own routing triggers per module. No code changes required.

---
> *"The constraint is the point. Structure is the cure."*
> — **Misha Sovereign**

---

## !!! WARNING !!!
**DO NOT** install standard `torch` instead of the CUDA-indexed version specified in `requirements.txt`. Using the wrong version will cause GPU detection failures and will break the telemetry layer. Use the `--index-url` command inside `requirements.txt` exactly as written.

# 3D-GEN AGENT: ARCHITECTURAL BLUEPRINT

## 🎯 Mission Statement
To function as an autonomous design intelligence capable of transforming high-level human intent (text or images) into production-ready, optimized `.glb` models, optimized for execution on NVIDIA T4 GPU environments.

---

## 🏗️ System Architecture

The agent operates using a **Master-Worker** pattern, where a central LLM (The Brain) orchestrates specialized model workers (The Hands).

### 1. The Controller (The Brain)
* **Model**: Llama-3-8B (Quantized to 4-bit/8-bit via `bitsandbytes`).
* **Responsibility**: 
    * **Intent Parsing**: Analyzing if the user wants a "perfect geometric object" (Procedural) or a "creative/organic object" (Generative).
    * **Task Decomposition**: Breaking a prompt into a multi-step pipeline.
    * **Error Correction**: Inspecting mesh errors and re-running workers.

### 2. The Skillset (The Workers)

#### A. Generative Path (The Artists)
* **Text-to-Image**: `Stable Diffusion XL (SDXL)` — Generates high-fidelity, clean reference images.
* **Image-to-3D**: `TripoSR` — Rapidly converts 2D images into 3D meshes.
* **VRAM Management**: All workers use `FP16` or `INT4` quantization to ensure coexistence on a single 16GB T4.

#### B. Procedural Path (The Engineers)
* **Code Generation**: LLM generates Python scripts using `Trimesh` or `CadQuery`.
* **Execution**: A sandboxed Python environment runs the code to generate geometry mathematically.

#### C. Post-Processing (The Polishers)
* **Mesh Utility**: `Trimesh` — Used for manifold checks, scaling, and texture mapping.
* **Format Converter**: `Trimesh` / `Blender (bpy)` — Ensures final export is a valid, compressed `.glb` file.

---

## 🔄 Decision Logic & Workflow

The agent follows a specific decision tree upon receiving a prompt:

### Step 1: Intent Classification
**User Prompt** $\rightarrow$ **LLM Analysis**
* *Is it mathematical/geometric?* (e.g., "A 5mm screw", "A hexagon") $\rightarrow$ **[PROCEDURAL PATH]**
* *Is it artistic/organic?* (e.g., "A fantasy dragon", "A vintage chair") $\rightarrow$ **[GENERATIVE PATH]**

### Step 2: Execution Pipeline

#### [IF PROCEDURAL]
1.  **Code Synthesis**: LLM writes Python code using `Trimesh`.
2.  **Validation**: Agent runs code in a local shell.
3.  **Optimization**: Agent checks for "non-manifold" errors.
4.  **Export**: Save as `.glb`.

#### [IF GENERATIVE]
1.  **Prompt Expansion**: LLM expands "dragon" into a high-detail SDXL prompt.
2.  **Image Gen**: SDXL produces a high-res `.png` on the T4.
3.  **3D Reconstruction**: TripoSR consumes the `.png` to create a `.obj` mesh.
4.  **Refinement**: `Trimesh` converts `.obj` $\rightarrow$ `.glb` and fixes UV maps.

---

## 🛠️ Technical Stack (T4 Optimized)

| Component | Technology | Optimization Strategy |
| :--- | :--- | :--- |
| **Orchestrator** | Llama-3-8B | `bitsandbytes` 4-bit quantization |
| **Image Engine** | SDXL | `Diffusers` library + FP16 |
| **3D Engine** | TripoSR | TensorRT or FP16 |
| **Mesh Logic** | Trimesh / Python | CPU-bound (saves VRAM) |
| **Environment** | Linux / CUDA 12.x | Dockerized/Conda |

---

## 🚀 Implementation Roadmap

1.  **Phase 1: The Foundation** — Setup the Llama-3 controller and the `Trimesh` procedural worker.
2.  **Phase 2: The Vision** — Integrate `SDXL` and `TripoSR` into the generative pipeline.
3.  **Phase 3: The Polish** — Implement automated mesh repair and UV unwrapping logic.
4.  **Phase 4: The Interface** — Build a simple API/CLI for the agent.

# llama.cpp Pascal CUDA — Windows Compile Guide
## The "No One Talks About This" Setup for Legacy NVIDIA Hardware

---

## Status / Disclaimer
This is a blueprint, not a maintained project. It documents a working
setup as of ~2 months ago (tested and confirmed working at that time on
the hardware/versions listed in `COMPILE_REQUIREMENTS.md`). CUDA toolkit
versions, VS Build Tools, and llama-cpp-python itself all update
independently — I haven't re-verified this against the latest releases,
so some steps may need adjustment by the time you read this. Worth trying
if you're stuck on Pascal hardware, but go in expecting to troubleshoot
version drift, not a guaranteed one-shot fix.

---

## The Core Problem
Pascal cards max out at CUDA 12.1.
VS 2022 STL hardcodes a CUDA 12.4+ requirement.
Result: STL1002 error on every compile attempt. No flags fix it.
Solution: Visual Studio 2019 Build Tools only.

---

## Step 1 — Install VS 2019 Build Tools
Installs alongside VS 2022, no conflicts:
```
https://aka.ms/vs/16/release/vs_buildtools.exe
```
Select: C++ build tools workload only.

---

## Step 2 — Copy CUDA MSBuild Files
CUDA installer puts these in VS 2022 by default. VS 2019 needs them manually.
Run in **admin PowerShell**:

```powershell
Copy-Item "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.1/extras/visual_studio_integration/MSBuildExtensions/*" "C:/Program Files (x86)/Microsoft Visual Studio/2019/BuildTools/MSBuild/Microsoft/VC/v160/BuildCustomizations/" -Force
```

---

## Step 3 — Compile
Run in your **project venv terminal**:

```powershell
$env:CMAKE_ARGS="-DGGML_CUDA=on -DCMAKE_CUDA_ARCHITECTURES=61 -G 'Visual Studio 16 2019'"
pip install llama-cpp-python --no-cache-dir
```

Flag breakdown:
- `-DGGML_CUDA=on` — enable CUDA backend
- `-DCMAKE_CUDA_ARCHITECTURES=61` — target Pascal (compute capability 6.1)
- `-G 'Visual Studio 16 2019'` — force CMake to use VS 2019 not VS 2022

Compile time: ~30 minutes on 1050 Ti. Normal. PTX assembler runs per kernel.

---

## Step 4 — Test

```python
from llama_cpp import Llama

llm = Llama(
    model_path="path/to/your/model.gguf",
    n_gpu_layers=-1,
    verbose=True
)
```

Success looks like:
```
CUDA0 KV buffer size = X MiB
CUDA0 compute buffer size = X MiB
Process finished with exit code 0
```

---

## Notes
- Wheel is tied to your venv. Other projects need recompile with same commands.
- RTX 5060 / Ampere+: skip all of this, standard compile works natively.
- AMD cards: different problem entirely. Not covered here.
- This guide exists because nobody else documented this specific combination.

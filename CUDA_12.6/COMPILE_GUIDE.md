# llama.cpp Pascal CUDA 12.6 — Windows Compile Guide (VS2022 path)
 
## Status / Disclaimer
 
Verified working end-to-end on GTX 1050 Ti, 2026-08-12. This is a smaller, single-machine sample than the CUDA 12.1 + VS2019 guide in this repo, which has been re-verified across multiple sessions — treat the numbers below as "one real cold-boot test," not a large-sample benchmark.
 
## Why use this over the 12.1 + VS2019 path?
 
Mainly convenience — you stay on a single VS2022 install instead of adding VS2019 Build Tools alongside it. Performance-wise, tested inference speed on this hardware was roughly the same between 12.1 and 12.6 (see benchmarks below) — this is not a guide about a large performance win, it's an alternate toolchain for people who'd rather not install a second VS version.
 
## Step 1 — Set up venv
 
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
 
# If you have an OLD llama-cpp-python wheel installed already (PyPI CPU
# version, or a previous custom CUDA build), uninstall it first so the
# new custom build starts clean. On a brand-new venv this is a no-op —
# skip it if you know your venv has nothing installed yet.
pip uninstall llama-cpp-python -y
```
 
## Step 2 — Point environment at CUDA 12.6 and compile
 
```powershell
# These three point the build at CUDA 12.6 specifically, in case you have
# multiple CUDA Toolkit versions installed side-by-side (common if you've
# tried other guides/versions before). Without this, the build system can
# pick up whichever CUDA version happens to be first on PATH, silently
# compiling against the wrong one.
#   CUDA_PATH          - generic path some tools/scripts check
#   CUDA_PATH_V12_6     - version-specific path some tools check instead
#   PATH (prepended)    - makes sure `nvcc` and other CUDA binaries found
#                         on the command line resolve to 12.6, not another
#                         installed version
$env:CUDA_PATH="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.6"
$env:CUDA_PATH_V12_6="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.6"
$env:PATH="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.6\bin;$env:PATH"
 
# Optional — force the build to use your exact physical core count.
# Ninja/MSBuild will otherwise try to use every logical thread it sees,
# which can overload weaker CPUs (especially older Pascal-era rigs).
#
# Check your own core count first: Task Manager > Performance > CPU,
# or a tool like Core Temp / HWiNFO.
#
# Examples (physical cores -> recommended value):
#   i3-7100 (2 physical cores / 4 threads with HT)  -> 2
#   4 physical cores (8 threads with HT)             -> 4
#   6 physical cores (12 threads with HT)            -> 6
#   8 physical cores (16 threads with HT)            -> 8
#
# Use the physical core count, not the thread count — hyperthreaded
# "virtual" cores don't add real compute for CPU-bound compilation and
# oversubscribing them can slow the build down instead of speeding it up.
$env:CMAKE_BUILD_PARALLEL_LEVEL="2"   # <- replace with your own physical core count
 
# -T cuda=12.6 is required — without it, MSBuild silently falls back to
# whichever CUDA toolset was registered first (often an older version),
# even if CUDAToolkit_ROOT points at 12.6.
$env:CMAKE_ARGS="-DGGML_CUDA=on -DCMAKE_CUDA_ARCHITECTURES=61 -T cuda=12.6"
pip wheel llama-cpp-python --no-cache-dir -w ./dist_cuda126
 
pip install (Get-ChildItem .\dist_cuda126\*.whl).FullName
```
 
Use `pip wheel`, not `pip install` directly — it leaves a reusable `.whl` in `./dist_cuda126`. Copy it somewhere outside the venv once built.
 
 
Compile time: varies roughly 10–30 minutes on 1050 Ti depending on how many physical cores your CPU has and whether you set `CMAKE_BUILD_PARALLEL_LEVEL` above. If you leave it at default (no physical core count set), expect the build to take around 30 minutes. Normal. PTX assembler runs per kernel.
 
## Step 3 — Test
 
```python
from llama_cpp import Llama
 
llm = Llama(
    model_path="path/to/your/model.gguf",
    n_gpu_layers=-1,
    verbose=True
)
```
 
## What success actually looks like
 
There is no special vendor-branded "success string" to look for in the log — what you should actually see is standard llama.cpp/ggml output:
 
```
ggml_cuda_init: found 1 CUDA devices (Total VRAM: ...):
  Device 0: NVIDIA GeForce GTX 1050 Ti, compute capability 6.1, ...
...
load_tensors: offloaded 36/36 layers to GPU
...
llama_perf_context_print:        load time = ...
llama_perf_context_print: prompt eval time = ... tokens per second)
llama_perf_context_print:        eval time = ... tokens per second)
```
 
If `offloaded X/36 layers to GPU` shows fewer than your full layer count, GPU offload isn't fully working — check VRAM headroom and `n_gpu_layers=-1`.
 
## Verified Benchmarks (GTX 1050 Ti, 4GB, cold boot / reboot before test)
 
Model: `Gemma-4-E2B-it-Q4_K_M.gguf` (2.88 GB) | Layers offloaded: 36/36
 
| Metric | CUDA 12.1 (this repo's other guide) | CUDA 12.6 (this guide) |
|---|---|---|
| Load time | 7.59s (uncontrolled for cache) | 10.14s (cold, post-reboot) |
| Prompt eval | ~60.2 tok/s | ~67.7 tok/s |
| Inference | ~26–28 tok/s | ~28.95 tok/s |
 
**Important caveat on load time:** model load time is heavily affected by Windows' file-system cache — if the model file was recently read by any process, a re-run can appear several times faster purely from the file sitting in RAM, with no relation to CUDA version. The 10.14s figure above was measured cold (full reboot immediately before the run). Don't trust load-time comparisons between toolchains unless both were tested cold the same way.
 
The prompt-eval and inference numbers are computation-bound, not disk-read-bound, so they aren't subject to that cache effect — but they're still only a single test run each. Inference speed is close to identical between 12.1 and 12.6 on this hardware; if you're deciding between the two paths, pick based on which toolchain (VS2019 vs VS2022) is less hassle for your setup, not on an expected performance gain.
 
## Notes
 
- Wheel is tied to your venv/Python minor version — recompile or match venv version for other projects.
- AMD cards: not covered here.
 

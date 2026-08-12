# llama.cpp Pascal CUDA — Windows Compile Guide
 
## The "No One Talks About This" Setup for Legacy NVIDIA Hardware
 
---
 
## Status / Disclaimer
This is a blueprint, not a maintained project. Originally documented ~2
months ago, and re-verified working end-to-end on 2026-07-22 (fresh venv,
full recompile via `pip wheel`, tested with a small model). CUDA toolkit
versions, VS Build Tools, and llama-cpp-python itself all update
independently, so treat this as "confirmed working as of the last
verification date," not a permanent guarantee — expect to troubleshoot
version drift eventually, just maybe not today.
 
---
 
## The Core Problem
VS 2022's STL hardcodes a CUDA 12.4+ requirement, causing an STL1002 error
on every compile attempt with older CUDA Toolkits. No flags fix it.
Solution: Visual Studio 2019 Build Tools, paired with CUDA 12.1 — the
specific combination verified working on Pascal hardware in this guide.
 
**This is primarily a VS2022 toolchain wall, not a hard Pascal-vs-CUDA ceiling
at exactly 12.1.** The fuller picture, rechecked against NVIDIA's own release
notes: CUDA 12.7 and earlier support Pascal (compute capability 6.0/6.1)
cleanly and stably — no warnings, no asterisks. CUDA 12.8 and 12.9 are
*usable* but not *stable* for Pascal: they compile and run, but NVIDIA has
marked Maxwell/Pascal/Volta "feature-complete" and deprecated on those
versions, meaning no further fixes or improvements will land for this
hardware, and every offline compile emits a deprecation warning as a
reminder. CUDA 13.0 removes offline-compilation support for Pascal
entirely — not usable at all.
 
So in principle, CUDA 12.2–12.7 paired with **VS2019** might also work
cleanly and stably on Pascal — that specific pairing is still untested as of
this update. What *has* now been tested is CUDA 12.6 paired with **VS2022**
instead of VS2019 — a different toolchain combination, documented separately
in `llama-cpp-pascal-cuda126-vs2022-windows/` in this repo, since 12.6
satisfies the VS2022 STL1002 check on its own and doesn't need VS2019 at all.
 
**Performance-wise, that 12.6/VS2022 test showed no meaningful advantage over
this 12.1/VS2019 setup** — inference speed was within margin of error
(~27-29 tok/s both ways) on the same GTX 1050 Ti. An earlier draft claimed a
large load-time and prompt-eval speedup for 12.6; that turned out to be a
Windows file-cache artifact from testing both versions back-to-back without a
reboot, not a real difference — see the other guide's benchmark section for
the corrected, cold-boot numbers. If you're choosing between the two guides,
pick based on which toolchain is less hassle to set up on your machine, not
on an expected speed gain.
 
12.1 is documented here because it's what was actually verified end-to-end
on this hardware with VS2019, not because it's the only stable option. If
you try a newer CUDA paired with VS2019 specifically and it works, that's a
real data point worth reporting back via an Issue.
 
---
 
## Step 0 — Verify venv is healthy
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip --version
```
If `pip` fails or `Scripts\` is missing files (activate.ps1, pip.exe), delete
and recreate the venv before continuing — a partial venv wastes the full
30-minute compile on an install that can't run.
 
---
 
## Step 1 — Install VS 2019 Build Tools
Installs alongside VS 2022, no conflicts:
```
https://aka.ms/vs/16/release/vs_buildtools.exe
```
Select: C++ build tools workload only.
 
---
 
## Step 2 — Copy CUDA MSBuild Files
**One-time per machine** — not repeated for future compiles once done.
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
pip wheel llama-cpp-python --no-cache-dir -w ./dist
```
Use `pip wheel`, not `pip install` — it leaves a reusable `.whl` file in
`./dist` instead of only installing into this one venv. Copy it somewhere
outside any venv immediately after compiling (venvs get deleted/recreated
more often than you'd think, and there's no other copy of this file).
 
To install it after building: `pip install .\dist\*.whl` (or, in
PowerShell, `pip install (Get-ChildItem .\dist\*.whl).FullName` since
PowerShell doesn't expand wildcards for external commands the way bash does).
 
Flag breakdown:
- `-DGGML_CUDA=on` — enable CUDA backend
- `-DCMAKE_CUDA_ARCHITECTURES=61` — target Pascal (compute capability 6.1)
- `-G 'Visual Studio 16 2019'` — force CMake to use VS 2019 not VS 2022
Compile time: ~30 minutes on 1050 Ti. Normal. PTX assembler runs per kernel.
 
**Portability note:** the wheel filename encodes the Python version it was
built against (e.g. `cp311`, `cp312`). Compilation itself doesn't care which
Python version you use, but the resulting wheel only installs into a venv
running that same minor version — create a matching venv, or recompile, if
you need it elsewhere. Don't be alarmed if the llama-cpp-python wheel itself
shows `py3-none` instead of a cp-tagged name — that's fine, it's the
dependency wheels (numpy, markupsafe) that are version-locked.
 
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
- CUDA 12.1 + VS2019 is what's verified here — not necessarily the only
  working combination. CUDA 12.6 + VS2022 is now also verified (separate
  guide, see above) with comparable performance. VS2019 paired with
  anything newer than 12.1 remains untested.
- This guide exists because nobody else documented this specific combination.
 

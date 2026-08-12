# llama-cpp-pascal-cuda-windows
 
CUDA-accelerated `llama-cpp-python` compile guides for legacy NVIDIA Pascal
GPUs (GTX 1050/1060/1070/1080 series) on Windows, where modern Visual Studio
refuses to compile against older CUDA toolkits out of the box.
 
Two verified toolchain paths are documented here, depending on which
Visual Studio setup you'd rather deal with:
 
| Path | CUDA | Visual Studio | Folder |
| :--- | :--- | :--- | :--- |
| A | 12.1 | 2019 Build Tools (installed alongside 2022) | [`CUDA_12.1/`](CUDA_12.1/) |
| B | 12.6 | 2022 (no second VS install needed) | [`CUDA_12.6/`](CUDA_12.6/) |
 
Both are tested end-to-end on a GTX 1050 Ti with comparable inference
performance (~27-29 tok/s either way) — pick whichever avoids more hassle
on your machine. Path A needs a second VS install but keeps CUDA on an
older, longer-verified version. Path B avoids installing VS2019 but
requires the newer CUDA toolkit.
 
## Path A — CUDA 12.1 + VS2019
 
1. **[CUDA_12.1/COMPILE_REQUIREMENTS.md](CUDA_12.1/COMPILE_REQUIREMENTS.md)** —
   hardware/software versions this was tested on, why each version is
   pinned, and measured performance vs. Ollama.
2. **[CUDA_12.1/COMPILE_GUIDE.md](CUDA_12.1/COMPILE_GUIDE.md)** — the actual
   step-by-step compile process, with the VS 2019 Build Tools workaround
   and flag explanations.
Read the requirements first — it explains *why* the versions in the guide
are pinned the way they are, which makes the guide itself easier to follow
if something doesn't match your setup.
 
## Path B — CUDA 12.6 + VS2022
 
1. **[CUDA_12.6/COMPILE_REQUIREMENTS.md](CUDA_12.6/COMPILE_REQUIREMENTS.md)**
2. **[CUDA_12.6/COMPILE_GUIDE.md](CUDA_12.6/COMPILE_GUIDE.md)**
Same hardware target, no VS2019 required, includes its own benchmark data.
 

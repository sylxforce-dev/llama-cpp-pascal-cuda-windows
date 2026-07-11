# llama-cpp-pascal-cuda-windows
 
CUDA-accelerated `llama-cpp-python` compile guide for legacy NVIDIA Pascal
GPUs (GTX 1050/1060/1070/1080 series) on Windows — where CUDA 12.1 is the
last supported toolkit version, but VS 2022 refuses to compile against it.
 
## Start here
 
1. **[COMPILE_REQUIREMENTS.md](COMPILE_REQUIREMENTS.md)** — hardware/software
   versions this was tested on, why each version is pinned, and measured
   performance vs. Ollama.
2. **[COMPILE_GUIDE.md](COMPILE_GUIDE.md)** — the actual step-by-step compile
   process, with the VS 2019 Build Tools workaround and flag explanations.
Read the requirements first — it explains *why* the versions in the guide
are pinned the way they are, which makes the guide itself easier to follow
if something doesn't match your setup.
 

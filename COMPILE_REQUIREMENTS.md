# llama.cpp Pascal CUDA — Windows Compile Requirements
 
## Hardware
- NVIDIA GTX 1050 Ti (Pascal, compute capability 6.1)
- 4GB VRAM
- Applies to any Pascal card: GTX 1050, 1060, 1070, 1080
## Software Stack
| Component | Version | Why |
|---|---|---|
| Python | 3.12 | Tested and confirmed |
| CUDA Toolkit | 12.1 | Tested and confirmed working with this VS2019 setup — see note below |
| VS Build Tools | **2019 (v16)** | Required by VS2022's STL version check (see below) |
| Git | Any | Required by llama-cpp-python build system |
 
## Why NOT VS 2022
VS 2022 STL header `yvals_core.h` hardcodes:
```
error STL1002: Unexpected compiler version, expected CUDA 12.4 or newer.
```
No flags bypass this. VS 2019 sidesteps that check entirely, which is why this guide uses CUDA 12.1 + VS2019 as a known-working combination.
 
**Note on CUDA version:** This is a VS2022-vs-CUDA-version compatibility wall, not a Pascal architecture limitation — NVIDIA's own Pascal Compatibility Guide confirms Pascal (compute capability 6.0/6.1) is supported through the entire CUDA 12.x series, not just 12.1. In principle, CUDA 12.4+ paired with VS2019 (not VS2022) might also work on Pascal — that combination hasn't been tested here. 12.1 is documented because it's the version actually verified end-to-end on this hardware, not because it's a hard ceiling.
 
## Model Tested
- gemma-4-E2B-it-Q4_K_M.gguf
- GGUF V3, Q4_K_M quantization
- Quantized by Unsloth
## Performance vs Ollama
| Backend | Simple query | Complex query |
|---|---|---|
| Ollama wrapper | 30+ seconds | 30+ seconds |
| llama.cpp direct CUDA | ~3 seconds | 15-21 seconds |
 

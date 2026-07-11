# llama.cpp Pascal CUDA — Windows Compile Requirements

## Hardware
- NVIDIA GTX 1050 Ti (Pascal, compute capability 6.1)
- 4GB VRAM
- Applies to any Pascal card: GTX 1050, 1060, 1070, 1080

## Software Stack
| Component | Version | Why |
|---|---|---|
| Python | 3.12 | Tested and confirmed |
| CUDA Toolkit | 12.1 | Last version supporting Pascal architecture |
| VS Build Tools | **2019 (v16)** | Last version compatible with CUDA 12.1 |
| Git | Any | Required by llama-cpp-python build system |

## Why NOT VS 2022
VS 2022 STL header `yvals_core.h` hardcodes:
```
error STL1002: Unexpected compiler version, expected CUDA 12.4 or newer.
```
No flags bypass this. VS 2019 is the only working solution with CUDA 12.1.

## Model Tested
- gemma-4-E2B-it-Q4_K_M.gguf
- GGUF V3, Q4_K_M quantization
- Quantized by Unsloth

## Performance vs Ollama
| Backend | Simple query | Complex query |
|---|---|---|
| Ollama wrapper | 30+ seconds | 30+ seconds |
| llama.cpp direct CUDA | ~3 seconds | 15-21 seconds |

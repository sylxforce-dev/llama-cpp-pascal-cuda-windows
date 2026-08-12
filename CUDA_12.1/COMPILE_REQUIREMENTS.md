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

**Note on CUDA version:** This is primarily a VS2022-vs-CUDA-version compatibility wall, not a Pascal architecture ceiling at exactly 12.1. Rechecked against NVIDIA's own release notes: CUDA 12.7 and earlier support Pascal (compute capability 6.0/6.1) cleanly and stably — no warnings, no asterisks. CUDA 12.8 and 12.9 are *usable* but not *stable* for Pascal: they compile and run, but NVIDIA has marked Maxwell/Pascal/Volta "feature-complete" and deprecated on those versions, meaning no further fixes will land for this hardware, and every offline compile emits a deprecation warning. CUDA 13.0 removes offline-compilation support for Pascal entirely — not usable at all.

**Update (2026-08-12):** CUDA 12.6 has now actually been compiled and tested
on this same GTX 1050 Ti — but paired with **VS2022**, not VS2019 (12.6
satisfies VS2022's STL1002 check on its own, so VS2019 isn't needed for that
specific combo). See `llama-cpp-pascal-cuda126-vs2022-windows/` in this repo
for that guide and its real benchmark numbers. The **VS2019 + CUDA 12.2–12.7**
combination described above as untested is *still* untested — the 12.6 data
point we now have used VS2022, a different toolchain, so it doesn't confirm
or rule out the VS2019 pairing. If you try VS2019 with something newer than
12.1 and it works, that's still a genuinely new data point worth reporting.

12.1 remains documented here because it's what was actually verified
end-to-end with VS2019 on this hardware, not because it's the only stable
option.

## Model Tested
- gemma-4-E2B-it-Q4_K_M.gguf
- GGUF V3, Q4_K_M quantization
- Quantized by Unsloth

## Performance vs Ollama
| Backend | Simple query | Complex query |
|---|---|---|
| Ollama wrapper | 7+ seconds | 15+ seconds |
| llama.cpp direct CUDA | ~3 seconds | 5-15 seconds |

# llama.cpp Pascal CUDA 12.6 — Windows Compile Requirements (VS2022 path)

This is an alternate path to the CUDA 12.1 + VS2019 guide in this repo.
Use this if you'd rather stay on Visual Studio 2022 instead of installing
VS2019 Build Tools alongside it. Both paths are verified working on Pascal;
neither is "better," they're just different toolchains.

---

## Hardware

* **Target Architecture:** Pascal GPU Architecture (Compute Capability 6.1)
* **Verified Silicon:** NVIDIA GeForce GTX 1050 Ti (4GB VRAM). Should also
  apply to GTX 1060, 1070, 1080, 1080 Ti (same sm_61 target), but only the
  1050 Ti has been tested end-to-end for this specific guide.
* **CPU Requirement:** Modern multi-core CPU with AVX2 support.

## Required Software

| Tool | Spec |
| :--- | :--- |
| NVIDIA Graphics Driver | 555.xx or higher |
| CUDA Toolkit | 12.6 |
| Visual Studio | 2022 (Community, Professional, or Enterprise), "Desktop development with C++" workload |
| Python | 3.10, 3.11, or 3.12, on PATH |

**Why 12.6 works here but 12.1 needed VS2019 in the other guide:** VS2022's
STL header (`yvals_core.h`) hardcodes a check requiring CUDA 12.4 or newer.
12.1 fails that check under VS2022 (`error STL1002`), which is why the other
guide in this repo uses VS2019 instead. 12.6 satisfies the STL1002 check
directly, so VS2022 works without needing a second, older VS install.

---

## Step 1 — Fix NVIDIA's MSBuild Integration (one-time per machine)

The CUDA 12.6 installer can fail to copy its build customization files into
the Visual Studio 2022 compiler target directory. Without this, CMake fails
to find the CUDA toolset. Run in **admin PowerShell**:

```powershell
Copy-Item "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.6/extras/visual_studio_integration/MSBuildExtensions/*" "C:/Program Files/Microsoft Visual Studio/2022/Community/MSBuild/Microsoft/VC/v170/BuildCustomizations/" -Force
```

Adjust the destination path if you're using Professional or Enterprise
instead of Community.

---

## Environment Verification

Once done, close the admin window. Proceed to `COMPILE_GUIDE.md` in this
folder to build the wheel.

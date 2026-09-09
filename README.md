# Triton & SageAttention Installer for ComfyUI (PowerShell 7)

This repo contains a **single PowerShell script** that installs **Triton** and **SageAttention** for the **ComfyUI Windows portable** build. It detects your environment, installs a matching CUDA build of PyTorch (or uses the one you already have), fetches the correct SageAttention wheel from the **AI-windows-whl JSON index**, and—when needed—adds the **Python `include/` and `libs/`** folders required by Triton on Python 3.13.

---

## ✨ What the script does

* **Preflight**: Prints your Python version/CP tag, Torch version, CUDA runtime, and GPU/driver (via `nvidia-smi`).

  * In **`-DryRun`**, the same detection calls are executed (Python imports & `nvidia-smi`) — only installs/changes are skipped.
* **Torch auto-install**: If Torch is missing, installs it automatically (prefers CUDA builds; CPU fallback is last resort — SageAttention needs CUDA).
* **Torch-first mode**: Optionally force a specific Torch + CUDA combo (e.g., `torch==2.8.0` + `cu128`).
* **Triton**: Installs `triton-windows<3.4` and (on Python 3.13) downloads & places **only** the `include/` and `libs/` folders into `python_embeded/`.

  * *It **never** touches `Lib/`.*
* **SageAttention**: Selects a compatible wheel for **SageAttention 2.2 (SageAttention2++)** from the JSON index:

  * Source: `https://raw.githubusercontent.com/wildminder/AI-windows-whl/refs/heads/main/wheels.json`
  * **CUDA minor fallback** (e.g., 12.9 → 12.8) if necessary
  * **ABI3/py3 fallback** is automatic when Python exact match isn’t present
* **Optional extras** (if you opt in): `FlashAttention`, `NATTEN`, `xformers`, `bitsandbytes` (also resolved from the JSON index)
* **Post-install checks**: Verifies imports for `torch` and `sageattention`, prints CUDA availability.
* **Quality of life**: Creates runners, saves an environment snapshot, and stores the fetched JSON index for transparency.

---

## 🧰 Requirements

* **Windows 10/11, 64-bit**
* **PowerShell 7+**
* **NVIDIA GPU** with a working **NVIDIA driver** (`nvidia-smi` should run)
* **ComfyUI Windows portable** root (run the script from that folder), e.g.:

  ```
  .\ComfyUI\main.py
  .\python_embeded\python.exe
  ```
* Internet access (to download wheels and `wheels.json`)

> **Note:** `nvcc` (CUDA Toolkit) is **optional**. The script checks for it to inform you; it’s not required for ComfyUI.

---

## 📦 Files the script may create

* `.\logs\Install-SageAttention-*.log` (transcript)
* `.\logs\requirements.before.txt` and `.\logs\requirements.after.txt`
* `.\aiwheels_index.json` (saved copy of the JSON index; skipped in `-DryRun`)
* `.\run_nvidia_gpu_sageattention.bat` and/or `.\Run-ComfyUI-Sage.ps1` (runners)
* `.\python_embeded\include\` and `.\python_embeded\libs\` (for Triton on Python 3.13)

---

## 🚀 Quick start

1. Place `Install-SageAttention.ps1` into your **ComfyUI portable root** (same folder as `python_embeded` and `ComfyUI`).
2. Open **PowerShell 7** in that folder.
3. Run:

   ```powershell
   .\Install-SageAttention.ps1 -CreateBatRunner
   ```
4. After success, launch ComfyUI with SageAttention:

   ```bat
   .\run_nvidia_gpu_sageattention.bat
   ```

   or

   ```powershell
   .\python_embeded\python.exe -s .\ComfyUI\main.py --windows-standalone-build --use-sage-attention
   ```

---

## 🔧 Common options

| Category         | Parameter                                                            | What it does                                                                                                         |
| ---------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Execution        | `-DryRun`                                                            | Execute detection and planning, but don’t install/uninstall or write files.                                          |
| Output           | `-DebugLog` / `-TraceScript`                                         | Print timings and very verbose execution details.                                                                    |
| SageAttention    | `-SageVersion 2.2` (default), `3`                                   | Select SageAttention version. Defaults to stable SageAttention 2.2. Specify `3` for experimental SageAttention 3.     |
| Torch (force)    | `-TorchVersion 2.8.0 -CudaTag cu128`                                 | Install exactly this Torch + CUDA before SageAttention.                                                              |
| Torch (auto)     | *(default)*                                                          | If Torch is missing, tries CUDA builds that fit your GPU/driver; CPU only as last resort (SageAttention needs CUDA). |
| Pip control      | `-PipIndexUrl` / `-PipExtraIndexUrl`                                 | Point pip at custom indexes (e.g., PyTorch wheels).                                                                  |
| Caching          | `-NoCache`                                                           | Use `--no-cache-dir` for pip installs.                                                                               |
| Triton dev files | `-SkipTritonPyDev`, `-ForceTritonPyDev`, `-TritonPyDevZipUrl <url>`  | Control the Python 3.13 headers/libs step.                                                                           |
| Extras           | `-AutoFetchFromAIWheels -InstallFlashAttention -InstallXFormers ...` | Install extra packages from the same wheel index.                                                                    |
| Runners          | `-CreateBatRunner`, `-CreatePsRunner`                                | Create handy launchers for ComfyUI + SageAttention.                                                                  |
| JSON index       | `-WheelsJsonUrl <url>` / `-WheelsJsonOut <path>`                     | Override the AI-windows-whl JSON endpoint or the output path.                                                        |

---

## 🧪 Examples

Force a specific Torch + CUDA, then install SageAttention 2.2:

```powershell
.\Install-SageAttention.ps1 -TorchVersion 2.8.0 -CudaTag cu128 -CreateBatRunner
```

Let the script auto-detect Torch (prefer CUDA), install Triton & SageAttention, and create runners:

```powershell
.\Install-SageAttention.ps1 -CreateBatRunner -CreatePsRunner
```

Verbose diagnostics (network issues? pip errors?):

```powershell
.\Install-SageAttention.ps1 -DebugLog
```

Use the official PyTorch extra index explicitly (when forcing versions):

```powershell
.\Install-SageAttention.ps1 -TorchVersion 2.8.0 -CudaTag cu129 -PipExtraIndexUrl https://download.pytorch.org/whl/cu129
```

Install extras from the wheel index too:

```powershell
.\Install-SageAttention.ps1 -AutoFetchFromAIWheels -InstallFlashAttention -InstallXFormers
```

Plan only (no changes, but real detections and resolution):

```powershell
.\Install-SageAttention.ps1 -DryRun
```

---

## 🖥️ Sample output (condensed)

```
▶ Preflight
  ✓ Python: 3.13.6  (cp313)
  ✓ Torch: 2.8.0+cu129 (CUDA 12.9, available=True)
  • CUDA Toolkit (nvcc): not found
  ✓ GPU/Driver: NVIDIA GeForce RTX 4060 Ti, 581.15

▶ Triton prerequisites
  ✓ Triton prerequisite: Python headers & libs already present.

▶ Install plan
  • Installing Triton …
  ✓ Triton ready.

▶ SageAttention
  • Selecting a compatible wheel (Torch 2.8.0, CUDA cu129, Python 3.13) …

▶ Fetch wheels.json
  ✓ Saved wheels.json → .\aiwheels_index.json
  ✓ Wheel selected: sageattention-2.2.0+cu128torch2.8.0-cp313-cp313-win_amd64.whl
  • Installing SageAttention …
  ✓ SageAttention installed.

▶ Verify
  ✓ torch import OK (2.8.0+cu129)
  ✓ sageattention import OK ()
  • CUDA runtime: 12.9, available: True

▶ Done
  ✓ Installation finished.
  Start ComfyUI with SageAttention:
    .\run_nvidia_gpu_sageattention.bat
```

---

## ⚡ SageAttention 2.2 vs. SageAttention 3

* **SageAttention 2.2 (`sageattention`) [Default]**: The stable, widely supported standard for ComfyUI, Wan2.1, Anima, etc. Enabled globally via ComfyUI's `--use-sage-attention` launch flag. Works across PyTorch versions (including PyTorch 2.10+ and 2.13+ on CUDA 13.0 / 12.8).
* **SageAttention 3 (`sageattn3`) [Optional]**: Experimental FP4 attention for Blackwell architectures. Specify `-SageVersion 3` if you wish to install it. Note: Windows support for SageAttention 3 is currently experimental and may encounter CUDA alignment errors.

---

## 📝 Notes on Python 3.13 (Triton)

Triton’s Windows wheels expect the Python **developer files** to exist alongside the embedded interpreter.
The script **automatically** downloads a ZIP that contains **two folders**:

* `include/`
* `libs/`

It copies them into `python_embeded\`.
**Do not** confuse `libs/` with `Lib/` — the script never touches `Lib/`.

You can override the download URL with `-TritonPyDevZipUrl`.

---

## 🧯 Troubleshooting

**“Installed Torch build is CPU-only”**

* Reinstall with a CUDA tag:

  ```powershell
  .\Install-SageAttention.ps1 -TorchVersion 2.8.0 -CudaTag cu128
  ```

**“No matching wheel for your combo”**

* The resolver automatically tries **CUDA minor fallback** (12.9 → 12.8) and allows **ABI3/py3** when needed.
* Consider aligning to a common pair (e.g., Torch `2.8.0` + `cu128`).

**Pip cannot connect / corporate networks**

* Set `-PipIndexUrl` / `-PipExtraIndexUrl` to mirrors you can access.
* Configure proxy env vars: `HTTP_PROXY`, `HTTPS_PROXY`.

**Execution policy blocks the script**

* Launch with:

  ```powershell
  pwsh -ExecutionPolicy Bypass -File .\Install-SageAttention.ps1
  ```

**`nvcc` not found**

* That’s OK; only the driver and CUDA runtime bundled with the Torch wheel are required. `nvcc` is optional.

---

## 🔍 How the wheel is selected (JSON-based)

1. Downloads **`wheels.json`** from **AI-windows-whl**.
2. Scans packages and their wheel entries.
3. Matches on:

   * **Torch** version (exact or same major.minor),
   * **CUDA** (pretty version, with **12.9 → 12.8** fallback),
   * **Python** (major.minor; **ABI3/py3** allowed if needed).
4. Installs the selected wheel via pip.
5. Saves a copy of the JSON as `aiwheels_index.json` (skipped in `-DryRun`).

---

## 🙌 Credits

* Community wheel index: **wildminder/AI-windows-whl**
* Windows wheels: **woct0rdho** (SageAttention, Triton Windows)
* PyTorch, Triton, and SageAttention maintainers & contributors
* ComfyUI project & community

---

## 🛡️ License

This repository is released under the **MIT License**. See [LICENSE](./LICENSE) for details.



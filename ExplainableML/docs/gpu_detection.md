# Reasons for PyTorch Not Detecting RTX 6020 GPU

PyTorch may fail to recognize the RTX 6020 GPU due to several common issues:

## Version Mismatch

+ NVIDIA Driver: Ensure that the installed NVIDIA driver supports the CUDA version required by your PyTorch installation.
+ CUDA Runtime: The CUDA version bundled with PyTorch must be compatible with the installed NVIDIA driver.

## PyTorch Build

+ Lack of CUDA Support: If you have installed a CPU-only version of PyTorch, it will not detect any GPU. Ensure you have the correct version that includes CUDA support.

## Troubleshooting Steps

To resolve the issue, follow these steps:

1. Check NVIDIA Driver Version:
+ Run nvidia-smi in the terminal to see the driver version and the maximum supported CUDA version.
+ Results from 2026-07-09 query shown below:

```
PS C:\Users\Admin> nvidia-smi
Thu Jul  9 11:05:04 2026
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 591.86                 Driver Version: 591.86         CUDA Version: 13.1     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                  Driver-Model | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 2060      WDDM  |   00000000:0A:00.0  On |                  N/A |
| 20%   37C    P8             22W /  170W |     746MiB /   6144MiB |      1%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            1912    C+G   C:\Windows\System32\dwm.exe           N/A      |
|    0   N/A  N/A            2264    C+G   ...s\Mozilla Firefox\firefox.exe      N/A      |
|    0   N/A  N/A            7172    C+G   ...yb3d8bbwe\WindowsTerminal.exe      N/A      |
|    0   N/A  N/A            7184    C+G   ...nternet Access\pia-client.exe      N/A      |
|    0   N/A  N/A            8432    C+G   C:\Windows\explorer.exe               N/A      |
|    0   N/A  N/A            8504    C+G   ...indows\System32\ShellHost.exe      N/A      |
|    0   N/A  N/A            8608    C+G   ...em32\ApplicationFrameHost.exe      N/A      |
|    0   N/A  N/A            8904    C+G   ...2txyewy\CrossDeviceResume.exe      N/A      |
|    0   N/A  N/A            9364    C+G   ..._cw5n1h2txyewy\SearchHost.exe      N/A      |
|    0   N/A  N/A            9376    C+G   ...y\StartMenuExperienceHost.exe      N/A      |
|    0   N/A  N/A           11220    C+G   ....0.4078.48\msedgewebview2.exe      N/A      |
|    0   N/A  N/A           12768    C+G   ...s\Mozilla Firefox\firefox.exe      N/A      |
|    0   N/A  N/A           15692    C+G   ...ntrolPanel\SystemSettings.exe      N/A      |
|    0   N/A  N/A           15924    C+G   ...t\Edge\Application\msedge.exe      N/A      |
+-----------------------------------------------------------------------------------------+
```
2. Verify PyTorch Installation:

Use the command: 

``` python
import torch
print(torch.version.cuda)
```

+ If the output is `None`, you have a CPU-only build. (This was the case when this was run on 2026-07-09.)

3. Reinstall PyTorch with CUDA Support:

+ If necessary, uninstall the current version: 

`pip uninstall torch torchvision torchaudio`

+ Install the correct version with CUDA support:

``` bash
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu124
```

## PROMPT (follow up): The version of CUDA installed on my system is 13.1, but recommended reinstall command was `pip install torch torchvision torchaudio --extra-index-url download.pytorch.org/whl/cu124`. Shouldn't I use the command:

pip install torch torchvision torchaudio --extra-index-url download.pytorch.org/whl/cu131

## RESPONSE:

Not necessarily.

PyTorch wheels (the `--extra-index-url .../whl/cuXXX` ones) bundle the CUDA *runtime* they need; they generally don’t use your installed local CUDA toolkit version. So your “CUDA 13.1 installed” mostly doesn’t determine what you should install.

What matters is NVIDIA driver compatibility with the CUDA major version your PyTorch wheel expects. If the guide recommends `cu124`, it usually means “install a PyTorch build whose CUDA major is known to work with your driver.”

So:
- If you follow the recommended `cu124` command and your NVIDIA driver is new enough, it will work even if your system also has CUDA 13.1 installed.
- Use `cu131` only if you’re sure there’s a `cu131` PyTorch wheel build that matches your OS/Python/PyTorch version *and* your driver supports it.

Quick check to decide: run `nvidia-smi` and look at the **Driver Version**—tell me that plus your **OS** and **Python version**, and I’ll tell you which `cuXXX` wheel line is the best match.

## PROMPT (follow up): 

Driver Version: 591.86\
OS: Windows 11\
Python version: 3.13.7

## RESPONSE:

Use `cu124` (as recommended) unless you have a specific reason to target `cu131`.

Two key points for your case:

    Your “CUDA 13.1 installed” is not what PyTorch uses. PyTorch CUDA wheels ship their own CUDA runtime dependencies; they only require that your NVIDIA driver can run that CUDA runtime. (pytorch.org, pytorch.org)

    Your command target (cu124 vs cu131) selects which PyTorch build (CUDA runtime version) you install. So if the guide tells you cu124, that’s the safest match for the driver/compat they’re assuming—not your local toolkit version. (pytorch.org)

Also: you’re on Python 3.13. Even though PyTorch supports Python 3.13 generally, the CUDA wheel you pick still must exist for Windows + cp313; if a specific CUDA variant doesn’t have cp313 Windows wheels, pip may fall back or behave unexpectedly. (GitHub)

Run this to verify what you actually installed and whether it can see the GPU:

``` python
import torch
print(torch.__version__)
print(torch.version.cuda)
print(torch.cuda.is_available())
```

If `torch.cuda.is_available()` is `False`, paste the output and I’ll tell you the exact install command to use next.

## PROMPT (follow up): 

```
Torch version: 2.13.0+cpu
Torch CUDA version: None
CUDA Available: False
```

## RESPONSE:

Your install is the CPU-only PyTorch build (`2.13.0+cpu`), so `torch.version.cuda` is `None` and CUDA can’t be detected until you reinstall a CUDA wheel.

```
pip uninstall -y torch torchvision torchaudio
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```

Then verify:

``` python
import torch
print(torch.__version__)
print(torch.version.cuda)
print(torch.cuda.is_available())
```

If `torch.cuda.is_available()` is still `False`, paste the output of those 3 lines (and your GPU model from Device Manager or `nvidia-smi`).

## Results of the verification

```
Torch version: 2.6.0+cu124
Torch CUDA version: 12.4
CUDA Available: True
```

So it looks like pytorch now see the available GPU. As a final verification, the results from running this code:

```
print(f"-------------------------------------------")
print(f"Number of GPUs: {torch.cuda.device_count()}")

if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)} is available.")
else:
    print("No GPU available. Training will run on CPU.")
```

was the following:

```
-------------------------------------------
Number of GPUs: 1
GPU: NVIDIA GeForce RTX 2060 is available.
```
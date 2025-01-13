─────────────────────────────────────────────────────────────────────────
README.md
─────────────────────────────────────────────────────────────────────────

# MyCUDA

MyCUDA is a minimal Python library showcasing how to integrate CUDA/C kernel code with Python using [pybind11](https://github.com/pybind/pybind11). It includes simple GPU‐accelerated matrix multiplication and elementwise addition, and can automatically detect and set the “best” GPU device available on your system.

--------------------------------------------------------------------------------
Table of Contents
--------------------------------------------------------------------------------
1. Overview  
2. Prerequisites  
3. Installation  
   3.1. Install via PyPI (Example)  
   3.2. Build from Source  
4. Usage  
   4.1. Simple Example in Python  
5. Platform‐Specific Notes  
   5.1. Linux  
   5.2. Windows  
6. Contributing and Extending  

--------------------------------------------------------------------------------
1. Overview
--------------------------------------------------------------------------------
This library demonstrates a minimal yet scalable approach to building a Python package that uses CUDA kernels for GPU acceleration. The code structure can be expanded with additional kernels, device dispatch logic, or specialized optimizations as your needs grow.

--------------------------------------------------------------------------------
2. Prerequisites
--------------------------------------------------------------------------------
• NVIDIA GPU with a suitable Compute Capability (e.g., SM 70+)  
• NVIDIA CUDA Toolkit (e.g., CUDA 11.x or newer)  
• Drivers supporting your GPU’s CUDA version  

Additionally, you should have:  
• Python 3.7+ (3.8 or higher recommended)  
• A C++ compiler that supports at least C++14  
• For building from source, the following Python packages are typically needed:  
  – [setuptools](https://pypi.org/project/setuptools/)  
  – [wheel](https://pypi.org/project/wheel/)  
  – [scikit-build](https://pypi.org/project/scikit-build/)  
  – [cmake](https://pypi.org/project/cmake/) (≥ 3.18)  
  – [ninja](https://pypi.org/project/ninja/) (recommended but not strictly required)  
  – [pybind11](https://pypi.org/project/pybind11/)  

--------------------------------------------------------------------------------
3. Installation
--------------------------------------------------------------------------------

### 3.1. Install via PyPI (Example)
If you published this library to PyPI (not shown in this demo project), you could install it with:
  
  pip install mycuda

(This automatically installs the pre‐built wheels if they are available for your platform and CUDA version.)

### 3.2. Build from Source
If you are working from the source code in this repository or prefer to build locally:

1. Clone or download this repo:  
       git clone https://github.com/YourUsername/mycuda.git
       cd mycuda

2. Ensure Python 3.7+ is installed and create/activate a virtual environment (recommended).  
   For example, on Linux:
       python -m venv .venv
       source .venv/bin/activate
   On Windows (PowerShell):
       python -m venv .venv
       .\.venv\Scripts\activate

3. Install build prerequisites:
       pip install -U setuptools wheel scikit-build cmake ninja pybind11

4. (Optional) Check that the CUDA Toolkit is in PATH or specify it if necessary:
   • On Linux, ensure nvcc is on PATH (often /usr/local/cuda/bin).  
   • On Windows, ensure the “NVIDIA CUDA Toolkit” folder is in your PATH or specify it in the CMake configuration step.

5. Install MyCUDA in editable mode or full build:
       pip install .

   This will invoke scikit‐build (which internally runs CMake) and compile the CUDA extension. Once finished, a “mycuda” package will be installed in your Python environment.  

If you prefer the manual approach:
   python setup.py build
   python setup.py install

--------------------------------------------------------------------------------
4. Usage
--------------------------------------------------------------------------------

### 4.1. Simple Example in Python

After installation:

  import mycuda

  # 1) Initialize and detect GPU device:
  mycuda.init_device()

  # 2) Create two arrays (e.g., NumPy float32)
  import numpy as np
  A = np.random.rand(4, 5).astype(np.float32)
  B = np.random.rand(5, 3).astype(np.float32)

  # 3) Perform matrix multiplication on GPU
  C = mycuda.matmul(A, B)
  print("C shape:", C.shape)  # should be (4, 3)

  # 4) Perform elementwise addition
  D = mycuda.add(A, A)
  print("D shape:", D.shape)  # same as A, (4, 5)

--------------------------------------------------------------------------------
5. Platform‐Specific Notes
--------------------------------------------------------------------------------

### 5.1. Linux
• Install the [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads) for your distribution.  
• Make sure your GPU driver is up to date and supports the CUDA version you installed.  
• The typical location of nvcc is /usr/local/cuda/bin/nvcc, and libraries are under /usr/local/cuda/lib64.  

### 5.2. Windows
• Install the [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads) for your Windows version.  
• Ensure that “nvcc.exe” and associated tools are on your PATH.  
• Use an up‐to‐date Visual Studio or MSVC Build Tools for proper C++14 support.  
• If using PowerShell or Command Prompt, you may need to restart the shell after installing the CUDA Toolkit so that environment variables are recognized.

--------------------------------------------------------------------------------
6. Contributing and Extending
--------------------------------------------------------------------------------
• To add new GPU kernels, create additional .cu/.h files in the “src/” directory, implement your kernels, and then expose them in “bindings.cpp.”  
• For advanced dispatch or multi‐GPU scenarios, edit or expand “detect_device.cu” to suit your requirements.  
• Pull requests, bug reports, or feature requests are welcome!

For more details, see the source files in the “src/” and “mycuda/” directories. Feel free to expand, optimize, or adapt this template for your own CUDA‐accelerated Python projects.

─────────────────────────────────────────────────────────────────────────
End of README.md
─────────────────────────────────────────────────────────────────────────

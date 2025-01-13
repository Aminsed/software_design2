--------------------------------------------------------------------------------
README.md
--------------------------------------------------------------------------------

# MyCUDA Project Template

A minimal, scalable Python package that integrates CUDA kernels to accelerate math operations (e.g., matrix multiplication and addition). This template shows how to structure your code so you can:

1. Build and install it easily with pip.  
2. Detect and initialize the best CUDA device automatically.  
3. Keep your Python “front end” clean and stable while easily adding new GPU kernels.  

--------------------------------------------------------------------------------
## Table of Contents
1. [Project Overview](#project-overview)  
2. [Directory Structure](#directory-structure)  
3. [Mermaid Diagram: Data & Control Flow](#mermaid-diagram-data--control-flow)  
4. [Prerequisites](#prerequisites)  
5. [Build & Install](#build--install)  
6. [Usage](#usage)  
7. [Extending and Adding New Kernels](#extending-and-adding-new-kernels)  
8. [License](#license)  

--------------------------------------------------------------------------------
## Project Overview

This project demonstrates a “gold standard” structure for combining Python and CUDA/C++ in a single, pip‐installable package. Key features:

• Uses [CMake](https://cmake.org/) + [scikit‐build](https://scikit-build.readthedocs.io/) for building native CUDA extensions.  
• [pybind11](https://github.com/pybind/pybind11) for binding C++ functions to Python.  
• A simple device detection mechanism to choose the GPU with the largest global memory at runtime.  
• Example kernels for matrix multiplication and elementwise addition.  
• Organized folder structure that you can expand with more kernels or advanced hardware optimizations.

--------------------------------------------------------------------------------
## Directory Structure

Below is the recommended layout. You can rename the top‐level folder (“mycuda”) to whatever your project name is.  

<pre>
mycuda/
├── pyproject.toml          # Tells pip (and other tools) how to build
├── setup.py                # scikit-build setup; runs CMake under the hood
├── CMakeLists.txt          # Top-level CMake config
├── src/
│   ├── CMakeLists.txt      # CMake config for the native extension
│   ├── bindings.cpp        # pybind11 bindings
│   ├── detect_device.cu    # CUDA code to pick "best" GPU device
│   ├── detect_device.h     # Corresponding header for detect_device.cu
│   ├── kernels.cu          # Example CUDA kernels (matmul, add, etc.)
│   └── kernels.h           # Corresponding header for kernels.cu
└── mycuda/
    ├── __init__.py         # Python package init
    └── core.py             # Python "front end" calling into native extension
</pre>

--------------------------------------------------------------------------------
## Mermaid Diagram: Data & Control Flow

Below is a conceptual flow of how a matrix multiplication call (and the device initialization) travels from Python code to the GPU.

```mermaid
flowchart TB
    subgraph Python Side
        A[mycuda.core.py<br>(matmul, add)] --> B[_mycuda extension<br>(bindings.cpp)]
    end

    subgraph Native / CUDA Side
        B --> C[detect_device.cu<br>detect_and_set_device()]
        B --> D[kernels.cu<br>(matmul_cuda(), add_cuda())]
    end

    C --> E[GPU Device Selection]
    D --> F[Matrix Multiply and Addition Kernels]
    E -->|Sets best device| GPU
    F -->|Launches on GPU| GPU
```

• The Python code in mycuda/core.py calls into the compiled extension (._mycuda).  
• The extension sets up the best device, then invokes GPU kernels in kernels.cu.  

--------------------------------------------------------------------------------
## Prerequisites

1. [NVIDIA CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit) installed and a compatible GPU.  
2. A Python environment (e.g., conda or venv) with pip.  
3. Development packages for building C++ and CUDA code (e.g., GCC on Linux or MSVC on Windows).  
4. Python packages for building:  
   • setuptools ≥ 42  
   • wheel  
   • scikit‐build  
   • cmake ≥ 3.18  
   • ninja (optional, but recommended)  
   • pybind11  

Example installation (conda environment or system Python):  
```
pip install -U setuptools wheel scikit-build cmake ninja pybind11
```

--------------------------------------------------------------------------------
## Build & Install

1. Clone (or copy) this repository.  
2. Navigate to the top‐level “mycuda” directory (where setup.py is located).  
3. Run:
   ```
   pip install .
   ```
   This triggers CMake via scikit‐build, compiles the CUDA extension, and installs the “mycuda” Python package into your current environment.  

4. (Optional) To verify or debug the build:  
   ```
   pip install . --verbose
   ```  
   Or if you prefer manual build/test:
   ```
   python setup.py build
   python setup.py install
   ```

--------------------------------------------------------------------------------
## Usage

After installation, you can run Python and import the package:

```python
import mycuda
import numpy as np

# 1. Initialize and detect the best GPU device at runtime
mycuda.init_device()

# 2. Prepare sample data (float32)
A = np.random.rand(4, 5).astype(np.float32)
B = np.random.rand(5, 3).astype(np.float32)

# 3. Matrix multiplication on the GPU
C = mycuda.matmul(A, B)
print("Matrix multiply result:\n", C)

# 4. Elementwise addition
D = mycuda.add(A, A)
print("Elementwise add result:\n", D)
```

Expected output:
1. A log line indicating which GPU was selected (e.g. “Using CUDA device: Tesla V100...”).  
2. The resultant arrays for both matmul and addition.

--------------------------------------------------------------------------------
## Extending and Adding New Kernels

Below are some suggestions for growing this template into a full‐featured GPU library:

1. Create a new CU file (e.g., src/new_ops.cu) to implement your custom kernel(s).  
2. Add function prototypes in a matching header (new_ops.h).  
3. Write a separate host “launch” function to configure block/grid sizes and call your kernel(s).  
4. Expose the new functionality in src/bindings.cpp via pybind11. For example:  
   ```cpp
   // In src/bindings.cpp
   m.def("my_new_op", &my_new_op_cuda, "Description of my new op");
   ```
5. Finally, add a Python function in mycuda/core.py that calls the newly bound function.  
6. (Optional) If you have multiple GPU implementations for different hardware (e.g., Tensor Cores vs. older GPUs), you can implement a runtime dispatcher in detect_device.cu to pick the most suitable kernel paths.  

--------------------------------------------------------------------------------
## License

(Replace this section with your project’s license terms. For example, MIT / Apache / BSD / GPL, etc.)

--------------------------------------------------------------------------------

Enjoy building your GPU‐accelerated Python projects! This README is meant to serve as a reusable roadmap. Just copy, rename, and adapt as needed for future CUDA+Python libraries.  

Happy hacking!

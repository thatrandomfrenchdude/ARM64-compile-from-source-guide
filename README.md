# ARM64 Python Package Compile from Source Guide
*NOTE: This guide is a work in progress and has yet to be full tested. This note will be removed when complete*

## 1. Prerequisites & Environment Setup

(These steps are the same as in the original guide.)

### Install Required Tools

- **Python (ARM64):**  
  Download the Windows ARM64 installer from [python.org](https://www.python.org/downloads/windows/) and install it (be sure to add Python to your PATH).

- **Visual Studio 2022:**  
  Install the Community Edition from [visualstudio.microsoft.com](https://visualstudio.microsoft.com/downloads/) with the **Desktop development with C++** workload. In addition, ensure the ARM64 compilers and libraries are selected in the “Individual components” tab.

- **CMake:**  
  Get the installer from [cmake.org](https://cmake.org/download/). Make sure to add CMake to your system PATH.

- **Git (Optional):**  
  Download from [git-scm.com](https://git-scm.com/download/win) if you plan to clone repositories.

### Get the Source Code (Using OpenCV as the Example)

1. Open a Command Prompt or PowerShell and navigate to your workspace.
2. Clone the repositories:
   ```bash
   git clone https://github.com/opencv/opencv.git
   git clone https://github.com/opencv/opencv_contrib.git
   ```
   *Note:* The `opencv_contrib` repo holds extra modules you may wish to include.

---

## 2. Configure and Build the Library

### Configure the Build with CMake

1. **Create a Build Directory:**
   ```bash
   cd opencv
   mkdir build
   cd build
   ```
2. **Run CMake to Configure the Project:**  
   For an ARM64 build with Visual Studio 2022 and Python bindings, run:
   ```bash
   cmake -G "Visual Studio 17 2022" -A ARM64 ^
         -DBUILD_opencv_python3=ON ^
         -DPYTHON_EXECUTABLE="C:\Path\To\python.exe" ^
         -DOPENCV_EXTRA_MODULES_PATH=..\opencv_contrib\modules ^
         ..
   ```
   **Flags explained:**
   - **-G "Visual Studio 17 2022":** Specifies the generator.
   - **-A ARM64:** Targets the ARM64 architecture.
   - **-DBUILD_opencv_python3=ON:** Enables building the Python 3 module.
   - **-DPYTHON_EXECUTABLE=...:** Provides the path to your ARM64 Python interpreter.
   - **-DOPENCV_EXTRA_MODULES_PATH=...:** (Optional) Adds additional modules.

### Build the Library

After configuration, build using either:

#### Visual Studio IDE
1. Open the generated `OpenCV.sln` file.
2. Select the **Release** configuration (or Debug if preferred).
3. Build the solution (this includes compiling the Python bindings).

#### Command Line
From the build directory, run:
```bash
cmake --build . --config Release
```
*Note:* Building may take some time given the size of OpenCV.

### Locate the Compiled Module

Once built, you should find a Python module (e.g., a `.pyd` file such as `cv2.cpython-<ver>-win_arm64.pyd`) in the output directories (often under a `\bin\Release\` folder). Test it using:
```bash
python -c "import cv2; print(cv2.__version__)"
```
A successful test indicates your build was completed properly.

---

## 3. Packaging as a Pip-Installable Wheel

Now that you have built the library, the next step is to package it as a wheel file. This section outlines a generic process that you can adapt to any C/C++‑based Python extension.

### Step 3.1: Prepare a Package Structure

Create a folder for your package (e.g., `opencv_python_package`). Inside that folder, create a subdirectory to hold the module:
```
opencv_python_package/
├── setup.py
└── opencv_python/
    ├── __init__.py
    └── cv2.pyd          # (Rename or copy the compiled .pyd file here)
```
*Tips:*
- Rename the `.pyd` file if needed (e.g., to `cv2.pyd` for consistency).
- The folder name `opencv_python` is arbitrary—you can choose any valid package name.

### Step 3.2: Write a `setup.py` Script

Create a minimal `setup.py` file. This script tells setuptools how to package your module. For example:
```python
from setuptools import setup, find_packages
import os

# Optionally, read version info from a file or define it here
VERSION = "4.5.5"  # Adjust this to your version

setup(
    name="opencv_python",  # Change this for your package
    version=VERSION,
    description="OpenCV Python bindings compiled for Windows ARM64",
    packages=find_packages(),
    package_data={
        # Include the compiled module file in the package
        "opencv_python": ["*.pyd"]
    },
    classifiers=[
        "Programming Language :: Python :: 3",
        "Operating System :: Microsoft :: Windows",
        "Topic :: Software Development :: Libraries :: Python Modules",
    ],
)
```
*Note:* This example is specific to our OpenCV use-case, but the structure applies to any package that includes a prebuilt binary module.

### Step 3.3: Build the Wheel

1. **Install Packaging Tools:**  
   Upgrade pip, setuptools, and wheel if you haven’t already:
   ```bash
   python -m pip install --upgrade pip setuptools wheel
   ```
2. **Run the Build Command:**  
   In the `opencv_python_package` directory (where your `setup.py` is located), run:
   ```bash
   python setup.py bdist_wheel
   ```
   This command creates a wheel file in the `dist` directory.

### Step 3.4: Install and Test the Wheel

Once the wheel is built, you can install it using pip:
```bash
pip install dist/opencv_python-4.5.5-py3-none-any.whl
```
Then verify by running:
```bash
python -c "import cv2; print(cv2.__version__)"
```
A successful output indicates that your wheel is properly built and installable.

---

## 4. Summary & Package-Agnostic Notes

While this example uses OpenCV, the steps here can be generalized for any Python package that involves compiling C/C++ code:

1. **Set Up Your Environment:** Install the necessary compilers, Python, CMake, and version control tools.
2. **Obtain and Configure the Source:** Clone your repository and use your build system (e.g., CMake) to configure the build for your target architecture.
3. **Compile the Library:** Use your IDE or command-line tools to build the source code.
4. **Package the Build:** Organize your package structure, write a minimal `setup.py` (or use an alternative like pyproject.toml), and build a wheel using setuptools.
5. **Test Installation:** Ensure the wheel installs correctly and that the module functions as expected.

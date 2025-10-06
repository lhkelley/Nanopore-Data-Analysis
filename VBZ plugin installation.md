# ------------------------------
# VBZ Plugin Installation Steps
# ------------------------------

# 1️⃣ Activate Conda environment
conda activate dinoPore2

# 2️⃣ Prepare source directory
cd /N/slate/lhkelley/vbz_compression
mkdir -p vbz_src/build
cd vbz_src/build

# 3️⃣ Configure CMake
cmake .. \
  -DENABLE_PYTHON=ON \
  -DENABLE_CONAN=ON \
  -DBUILD_SHARED_LIBS=ON

# 4️⃣ Build the plugin
make -j4

# 5️⃣ Verify the build
ctest -j4

# Optional: Check Python bindings
python -c "import pyvbz; print(pyvbz.__version__)"

# Optional: End-to-end compression/decompression test
python -c "
import pyvbz
import numpy as np
arr = np.array([1,2,3,4,5,6,7,8], dtype=np.uint32)
compressed = pyvbz.compress(arr)
decompressed = pyvbz.decompress(compressed, arr.shape[0])
print(np.array_equal(arr, decompres_

# Using Dinopore to detect inosines with ONT seqeuencing data

Step 1: Cry.

Step 2: Move on with life.

## Preparing environments

```
# Inside your conda env with R
mkdir -p ~/Rlibs
export R_LIBS_USER=~/Rlibs
```
Then in R:
```
# Use personal library
.libPaths("~/Rlibs")

# Install pacman if needed
if (!requireNamespace("pacman", quietly = TRUE)) {
  install.packages("pacman", repos="https://cloud.r-project.org")
}
library(pacman)
```

```
library(keras3)

use_condaenv("dinopore_r2", required = TRUE)

# Then check if TensorFlow is available
library(tensorflow)
tf_config()
```

```
dir.create("~/R/library", showWarnings = FALSE, recursive = TRUE)
.libPaths("~/R/library")   # sets your personal library for this session
install.packages("keras3", lib="~/R/library")
install.packages("dotty", lib="~/R/library")
```

```
library(reticulate)

# Point R to the Python inside your dinopore_r2 conda env
use_python("/N/u/lhkelley/Quartz/.conda/envs/dinopore_r2/bin/python", required = TRUE)

# Verify it worked
py_config()
```

```
library(tensorflow)
tf <- tensorflow::tf
tf$constant("Dinopore test")
```

Output of that should be:
```
tf.Tensor(b'Dinopore test', shape=(), dtype=string)
```

```
install.packages("vroom", type = "source", lib="~/R/library",
                 repos="https://cloud.r-project.org")
```




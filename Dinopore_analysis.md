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

# Install keras and tensorflow
p_load(keras, tensorflow)
```


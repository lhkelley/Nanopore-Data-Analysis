# Using Dinopore to detect inosines with ONT seqeuencing data

Step 1: Cry.

Step 2: Move on with life.

## Preparing environments

```
source /N/soft/rhel8/miniconda/python3.11.7/24.1.2/etc/profile.d/conda.sh
conda activate dinoPore
```

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
cd ~/vbz_compression/build
cmake .. -DENABLE_PYTHON=OFF -DENABLE_CONAN=OFF -DCMAKE_BUILD_TYPE=Release
make -j4

## Testing subsets of data to get the second half to run

```
#Create directory structure
mkdir -p /N/scratch/lhkelley/N2_rep1_ont_1M_seq/aggregate_reads

#Copy reference files
cp /N/scratch/lhkelley/N2_rep1_ont_250M_seq/wb.ref.ws275.genomic.fa* /N/scratch/lhkelley/N2_rep1_ont_1M_seq/

#Create 100M subset (will take ~5 minutes)
head -n 1000001 /N/scratch/lhkelley/N2_rep1_ont_250M_seq/aggregate_reads/N2_rep1_converted_fast5.tsv_nnpl_inAE.txt_grpN2_rep1_group > /N/scratch/lhkelley/N2_rep1_ont_1M_seq/aggregate_reads/N2_rep1_converted_fast5.tsv_nnpl_inAE.txt_grpN2_rep1_group

#Verify
wc -l /N/scratch/lhkelley/N2_rep1_ont_1M_seq/aggregate_reads/N2_rep1_converted_fast5.tsv_nnpl_inAE.txt_grpN2_rep1_group
```

```
install.packages("vroom", type = "source", lib="~/R/library",
                 repos="https://cloud.r-project.org")
```




# Nanopore Data Analysis
#### September 2025

## Quality checks

Install Nanoplot:
```
conda install -c bioconda nanoplot
```

Remove reads without quality scores (likely if you combined the pass and skipped pod5 files before basecalling):
```
awk 'NR%4==0 && length($0)==0 {next} {print prev} {prev=$0}' FBD39849_fastq_pass_66715812_a178e498_0.fastq > /N/slate/lhkelley/FBD39849_fastq_pass_66715812_a178e498_0_remove_miss_qscores.fastq
```

Count the number of reads in the FASTQ file that you'll input into Nanoplot:
```
awk 'NR%4==1 {count++} END{print count}' your_fastq.fastq
```

Remove incomplete reads (don't have @ as the first character):
```
awk '
BEGIN { kept=0; stripped=0; }
/^@/ {
    header=$0; getline seq; getline plus; getline qual;
    if (seq && plus && qual) {
        print header "\n" seq "\n" plus "\n" qual
        kept++
    } else {
        stripped++
    }
}
END { 
    print "Reads kept:", kept > "/dev/stderr"
    print "Reads stripped:", stripped > "/dev/stderr"
}' remove_miss_qscores.fastq > remove_miss_qscores_stripped.fastq
```

Check the number of stripped reads:
```
tail -n 2 remove_miss_qscores_stripped.fastq
```

Run Nanoplot on passed FASTQ reads from Dorado:
```
NanoPlot --fastq N2_rep1_basecall_out_12Sept25_new/*/fastq_pass/*.fastq \
         -o N2_rep1_nanoplot_out_pass \
         --threads 12
```

## Align to genome

Install minimap2 (only have to do once):
```
conda install bioconda::minimap2
```

Find the reference genome version you want to use and download:
```
wget https://downloads.wormbase.org/releases/WS275/species/c_elegans/PRJNA13758/c_elegans.PRJNA13758.WS275.genomic.fa.gz -O wb.ref.ws275.genomic.fa.gz
gunzip wb.ref.ws275.genomic.fa.gz
```

Align to reference genome, while being splice aware (run in background):
```
minimap2 -ax splice -uf -k14 wb.ref.ws275.genomic rna_reads.fastq > aln.sam &
```
```-ax splice``` accounts for introns/large gaps in the alignment.

```-u``` denotes that this is long-read data.

```-f``` means "full-length" as in full-length cDNA or direct RNA.

```-k14``` sets the k-mer size; the default is 15 for ONT. Reducing the number makes it more sensitive, but takes longer.

Try out Graphmap2:
```
git clone https://github.com/lbcb-sci/graphmap2.git
```

Install:
```
conda install -c bioconda nanopolish=0.11.1 samtools=1.9 scipy=1.7.1 pillow=8.3.1 pyyaml=5.4.1 requests=2.26.0 -y

```
I had to use ```pip``` because the conda installation wanted a more recent version of Python installed, but I thought it was best to keep the Python version that Dinopore was published with.

Find the plugin path:
```
python -c "import hdf5plugin, os; print(os.path.dirname(hdf5plugin.__file__))"
/N/u/lhkelley/Quartz/miniconda3/envs/dinoPore/lib/python3.8/site-packages/hdf5plugin
```

Set the plugin path:
```
export HDF5_PLUGIN_PATH=/N/u/lhkelley/Quartz/miniconda3/envs/dinoPore/lib/python3.8/site-packages/hdf5plugin
```
## Check if you have really short or long reads in your FASTQ files that will cause GraphMap2 to seqfault:

```
FASTQ=N2_rep1_converted_fast5.combined.fastq

# total reads (remove commas so arithmetic works)
total=$(seqkit stats "$FASTQ" | awk 'NR==2 {gsub(",","",$4); print $4}')

# reads shorter than 50 bp
small=$(seqkit seq -m 1 -M 49 "$FASTQ" -n | wc -l)

# reads longer than 100000 bp (100 kb) — change 100000 to another cutoff if desired
large=$(seqkit seq -m 100001 "$FASTQ" -n | wc -l)

# reads that would remain after filtering >=50 and <=100000
kept=$(( total - small - large ))

# percentages
pct_small=$(awk -v s=$small -v t=$total 'BEGIN{printf "%.3f", (s/t)*100}')
pct_large=$(awk -v l=$large -v t=$total 'BEGIN{printf "%.3f", (l/t)*100}')
pct_kept=$(awk -v k=$kept -v t=$total 'BEGIN{printf "%.3f", (k/t)*100}')

echo "TOTAL reads: $total"
echo "SHORT (<50 bp): $small ($pct_small%)"
echo "LONG (>100000 bp): $large ($pct_large%)"
echo "KEPT (>=50 & <=100000): $kept ($pct_kept%)"
```
When I ran this on my N2_rep1 file, I had 6,683 short reads (<50 bp, 0.145%) and 15 very long reads (>100 kb, 0.000%), so 99.9% of my reads are remaining. When I checked 50kb reads, there were 38 reads, which is 0.001%. I'm going to remove reads less than 50 bp and greater than 50 kb.

Pulling out only the short or long reads:
```
FASTQ=N2_rep1_converted_fast5.combined.fastq

# short reads (<50 bp)
seqkit seq -m 1 -M 49 "$FASTQ" -o shorts.fastq
seqkit stats shorts.fastq

# long reads (>50000 bp)
seqkit seq -m 50001 "$FASTQ" -o longs.fastq
seqkit stats longs.fastq
```
When I ran GraphMap2 on the short reads, it was fine. But, when I ran the long reads, it crashed out at the 300kb long reads. 

```
/N/slate/lhkelley/Dinopore/code/misc/graphmap2 align -x rnaseq -t 4 -r "$REF" -d longs.fastq -o longs_test.sam
```
This was the error:
```
[11:24:12 BuildIndexes] Loading reference sequences.
[11:24:13 SetupIndex_] Loading index from file: '/N/slate/lhkelley/N2_rep1_ont/wb.ref.ws275.genomic.fa.gmidx'.
[11:24:14 Index] Memory consumption: [currentRSS = 2164 MB, peakRSS = 2644 MB]
[11:24:14 Run] Hits will be thresholded at the percentil value (percentil: 99.000000%, frequency: 80).
[11:24:14 Run] Minimizers will be used. Minimizer window length: 5
[11:24:14 Run] Reference genome is assumed to be linear.
[11:24:14 Run] One or more similarly good alignments will be output per mapped read. Will be marked secondary.
[11:24:14 ProcessReads] All reads will be loaded in memory.
[11:24:14 ProcessReads] All reads loaded in 0.04 sec (size around 8 MB). (4444569 bases)
[11:24:14 ProcessReads] Memory consumption: [currentRSS = 2173 MB, peakRSS = 2644 MB]
[11:24:14 ProcessReads] [CPU time: 0.04 sec, RSS: 2173 MB] Read: 0/38 (0.00%) [m: 0, u: 0], length = 257226, qname: 9cc7c99a-5e72-4d2e-b0ef-[11:24:25 ProcessReads] [CPU time: 43.53 sec, RSS: 2794 MB] Read: 10/38 (26.32%) [m: 2, u: 5], length = 66320, qname: e058521d-f17b-488c-ba9[11:24:27 ProcessReads] [CPU time: 51.37 sec, RSS: 2956 MB] Read: 13/38 (34.21%) [m: 2, u: 8], length = 199188, qname: 0fe34e96-9c30-491d-b3[11:24:36 ProcessReads] [CPU time: 85.28 sec, RSS: 2547 MB] Read: 17/38 (44.74%) [m: 4, u: 10], length = 110289, qname: 3ad84707-cce5-46c9-8[11:24:38 ProcessReads] [CPU time: 92.19 sec, RSS: 3499 MB] Read: 20/38 (52.63%) [m: 4, u: 13], length = 59616, qname: 3de4bf4a-07b6-46ae-ae[11:24:41 ProcessReads] [CPU time: 107.65 sec, RSS: 2811 MB] Read: 25/38 (65.79%) [m: 5, u: 17], length = 331947, qname: 2b1f39af-815c-4661-ad4d-256e75b52e3a ru...Segmentation fault (core dumped)
```

So, I'm removing reads that are >50kb from my FASTQ file, which is only 38 reads.

```
Code to see the estimated time that your Slurm job will start. Replace jobID when your jobID number:
```
 watch -n 30 '
scontrol show job 7170570 | grep -E "JobState|StartTime" | while read line; do
    if [[ $line == StartTime* ]]; then
        # extract the timestamp and convert to 12-hour
        ts=$(echo $line | awk -F= "{print \$2}" | awk "{print \$1}")
        ts_12=$(date -d "$ts" +"%m/%d/%Y %I:%M:%S %p")
        echo "StartTime (12-hr format): $ts_12"
    else
        echo "$line"
    fi
done
'
```

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

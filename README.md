# velvet_for-assembling-and-optimizing
## To install


## script for running
### To run velveth and velvetg script
```
#!/bin/bash

# Define input directory and output directory
INPUT_DIR="/home/sequser/SILANTOI/miniproject/ecoli"
OUTPUT_DIR="ASSEMBLED"
KMER_SIZE=31

# Step 1: Run velveth
echo "Running velveth..."
velveth $OUTPUT_DIR $KMER_SIZE -fastq.gz $INPUT_DIR/*.gz

# Step 2: Run velvetg
echo "Running velvetg..."
velvetg $OUTPUT_DIR -scaffolding no

echo "Velvet assembly completed!"
```
### To assemble and optimize
```
#!/bin/bash

# Define input directory and output directories
INPUT_DIR="/home/sequser/SILANTOI/miniproject/ecoli"
OUTPUT_DIR_VELVET="ASSEMBLED"
OUTPUT_DIR_OPTIMISED="OPTIMIZED"
KMER_MIN=25
KMER_MAX=55

# Step 1: Run velveth
echo "Running velveth..."
velveth $OUTPUT_DIR_VELVET $KMER_MIN,$KMER_MAX,-2 -fastq.gz \
    $INPUT_DIR/ERR775888_fastq1.gz $INPUT_DIR/ERR775888_fastq2.gz \
    $INPUT_DIR/ERR775884_fastq1.gz $INPUT_DIR/ERR775884_fastq2.gz

# Step 2: Run velvetg
echo "Running velvetg..."
velvetg $OUTPUT_DIR_VELVET -scaffolding no

# Step 3: Run velvetoptimiser
echo "Running velvetoptimiser..."
velvetoptimiser -s $KMER_MIN -e $KMER_MAX -f '-fastq.gz' -i $OUTPUT_DIR_VELVET -o $OUTPUT_DIR_OPTIMISED

echo "Velvet assembly and optimization completed!"
```

### using wildcards for multiple sequences
```
#!/bin/bash

# Define input directory and output directories
INPUT_DIR="/home/sequser/SILANTOI/miniproject/ecoli"
OUTPUT_DIR_VELVET="ASSEMBLED"
OUTPUT_DIR_OPTIMISED="OPTIMIZED"
KMER_MIN=25
KMER_MAX=55

# Step 1: Run velveth
echo "Running velveth..."
velveth $OUTPUT_DIR_VELVET $KMER_MIN,$KMER_MAX,-2 -fastq.gz $INPUT_DIR/*.gz

# Step 2: Run velvetg
echo "Running velvetg..."
velvetg $OUTPUT_DIR_VELVET -scaffolding no

# Step 3: Run velvetoptimiser
echo "Running velvetoptimiser..."
velvetoptimiser -s $KMER_MIN -e $KMER_MAX -f '-fastq.gz' -i $OUTPUT_DIR_VELVET -o $OUTPUT_DIR_OPTIMISED

echo "Velvet assembly and optimization completed!"
```

### Without using a script command for assembling multiple sequences that have been splitted
```
velveth trials 31 -fastq.gz *_1.fastq.gz *_2.fastq.gz | velvetg - -scaffolding no
```

### Combining velvet assembling command with velvelOptimiser
```
velveth trials 31 -fastq.gz *_1.fastq.gz *_2.fastq.gz | velvetg - -scaffolding no | velvetoptimiser -s 25 -e 55 -f '-fastq.gz' -i - -o optimized
```

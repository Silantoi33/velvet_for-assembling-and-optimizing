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

# Activate Conda environment
source activate velvet_environment

# Define input directory and output directory
INPUT_DIR="/home/sequser/SILANTOI/miniproject/ecoli"
OUTPUT_DIR="ASSEMBLED"
KMER_SIZE=31

# Step 1: Run velveth
echo "Running velveth..."
for file in $INPUT_DIR/*_1.fastq.gz; do
    base_name=$(basename $file _1.fastq.gz)
    velveth $OUTPUT_DIR $KMER_SIZE -fastq.gz \
        $INPUT_DIR/${base_name}_1.fastq.gz $INPUT_DIR/${base_name}_2.fastq.gz
done

# Step 2: Run velvetg
echo "Running velvetg..."
velvetg $OUTPUT_DIR -scaffolding no

echo "Velvet assembly completed!"

# Deactivate Conda environment
conda deactivate
```

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

##Using spades and Links. 
spades for assembling and links for scaffolding
```
#!/bin/bash

# Define paths
READS_DIR="/home/sequser/SILANTOI/miniproject/ecoli-project"
SPAdes_OUTPUT="$READS_DIR/spades_output"
LINKS_OUTPUT="$READS_DIR/links_output"

# Activate SPAdes environment
source activate spades_env

# Run SPAdes assembly
spades.py -1 "$READS_DIR/reads_R1.fastq.gz" -2 "$READS_DIR/reads_R2.fastq.gz" -o "$SPAdes_OUTPUT"

# Deactivate SPAdes environment
conda deactivate

# Activate LINKS environment
source activate links_env

# Run LINKS for scaffolding
LINKS -f "$SPAdes_OUTPUT/scaffolds.fasta" -s "$READS_DIR/reads_R1.fastq.gz" -s "$READS_DIR/reads_R2.fastq.gz" -k 20 -l 500 -t 8 -x 0.1 -a 0.5 -b "$LINKS_OUTPUT"

# Deactivate LINKS environment
conda deactivate
```
# LINKS TOOL FOR SCAFFOLDINGS ASSEMBLED CONTIGS FROM VELVET
## command
```
LINKS -f contigs.fa -s paired_end_reads_1.fastq -s paired_end_reads_2.fastq -k 20 -l 100 -t 8 -a 0.5 -b scaffolds
```
## Use this script.
Make sure you have the splitted fastq.gz files and contigs.fa file containing the assembled sequences from velvet.
```
#!/bin/bash

# Iterate over all paired-end reads files ending with 1.fastq.gz
for file in *_1.fastq.gz; do
    # Get the base filename without the extension
    base=$(basename "$file" _1.fastq.gz)
    
    # Check if the corresponding *_2.fastq.gz file exists
    if [ -f "${base}_2.fastq.gz" ]; then
        # Run LINKS with the paired-end reads files
        LINKS -f contigs.fa -s "$file" -s "${base}_2.fastq.gz" -k 20 -l 100 -t 8 -a 0.5 -b "${base}_scaffolds"
    else
        echo "Corresponding paired-end reads file ${base}_2.fastq.gz not found."
    fi
done
```

#### Using the path
```
#!/bin/bash

# Change directory to where the files are located
cd /home/sequser/SILANTOI/mini-project/ecoli-project

# Run LINKS for all paired-end reads files ending with 1.fastq.gz
for file in *_1.fastq.gz; do
    base=$(basename "$file" _1.fastq.gz)
    if [ -f "${base}_2.fastq.gz" ]; then
        LINKS -f contigs.fa -s "$file" -s "${base}_2.fastq.gz" -k 20 -l 100 -t 8 -a 0.5 -b "${base}_scaffolds"
    else
        echo "Corresponding paired-end reads file ${base}_2.fastq.gz not found."
    fi
done
```

#### Using Spades to optimize velvet assembled file
```
#!/bin/bash

# Define variables
CONTOIGS_FILE="/home/SILANTOI/mini-project/ecoli-project/ASSEMBLED/contigs.fa"
OUTPUT_DIR="/home/SILANTOI/mini-project/ecoli-project/ASSEMBLED/spades_output"

# Run SPAdes
spades.py -s "$CONTOIGS_FILE" -o "$OUTPUT_DIR"
```

#### echo for error messages
```
#!/bin/bash

# Change directory to where the files are located
cd /home/sequser/SILANTOI/mini-project/ecoli-project

# Run LINKS for all paired-end reads files ending with 1.fastq.gz
for file in *_1.fastq.gz; do
    base=$(basename "$file" _1.fastq.gz)
    if [ -f "${base}_2.fastq.gz" ]; then
        echo "Running LINKS for ${file} and ${base}_2.fastq.gz..."
        LINKS -f contigs.fa -s "$file" -s "${base}_2.fastq.gz" -k 20 -l 100 -t 8 -a 0.5 -b "${base}_scaffolds"
        echo "LINKS completed successfully."
    else
        echo "ERROR: Corresponding paired-end reads file ${base}_2.fastq.gz not found."
    fi
done
```

```
#!/bin/bash

# Change directory to where the files are located
cd /home/sequser/SILANTOI/mini-project/ecoli-project/assembled-ecoli

# Run VelvetOptimiser
VelvetOptimiser.pl -s 29 -e 31 -f '-shortPaired -fastq.gz' -o 'optimized-ecoli' --d /path/to/velvet
```

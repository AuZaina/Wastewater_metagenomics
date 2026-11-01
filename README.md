
# Wastewater Metagenomics

Exploratory metagenomic analysis of publicly available paired-end shotgun sequencing data from wastewater influent and effluent samples.

## Dataset 

The sequencing data were obtained from the NCBI Sequence Read Archive (SRA) under BioProject [PRJNA693629](https://www.ncbi.nlm.nih.gov/sra/?term=PRJNA693629), associated with the study by Azli et al. (2022), *Metagenomics Insights Into the Microbial Diversity and Microbiome Network Analysis on the Heterogeneity of Influent to Effluent Water*, published in *Frontiers in Microbiology* on 14 April 2022 ([DOI](https://doi.org/10.3389/fmicb.2022.779196))

## Workflow

### 1. Data Retrieval

#### Download sequencing runs from NCBI SRA

```bash
prefetch SRR13612675
prefetch SRR13612676
```

#### Convert SRA files into paired-end FASTQ files

```bash
fasterq-dump SRR13612675 --split-files
fasterq-dump SRR13612676 --split-files
```

### 2. Quality Control and Read Preprocessing

#### Assess raw reads using FastQC

#### Trim and filter paired-end reads using fastp

```bash
fastp \
    -i in.R1.fq.gz \
    -I in.R2.fq.gz \
    -o out.R1.fq.gz \
    -O out.R2.fq.gz \
    --detect_adapter_for_pe \
    --trim_poly_g \
    --qualified_quality_phred 20 \
    --length_required 50 \
    --thread 8 \
    --json qc_result/sample.json \
    --html qc_result/sample.html
```

#### Remove host genome using BWA

```bash
##Index human reference genome
bwa index -p human -a bwtsw GCF_000001405.40_GRCh38.p14_genomic.fna 

##Align paired-end reads against the human reference genome
bwa mem -t 4 human \
    sample_R1.fastq.gz \
    sample_R2.fastq.gz \
    > sample_human.sam

##Filter human-mapped reads using SAMtools
samtools view -u -f 12 -F 2304 sample_human.sam |
samtools fastq \
    -1 filtered_R1.fastq.gz \
    -2 filtered_R2.fastq.gz \
    -0 /dev/null \
    -s /dev/null \
    -n -
```

#### Assess processed reads using FastQC
#### Aggregate quality-control reports using MultiQC

### 3. Metagenomic Assembly

#### Assemble quality-controlled paired-end reads using MEGAHIT

```bash
megahit \
    -1 _R1.fastq.gz \
    -2 _R2.fastq.gz \
    -o sample_assembly
```

### 4. Contig Filtering and Assembly Statistics using seqkit

#### Retain contigs of at least 500 bp and calculate statistics for filtered assembly

```bash
seqkit seq -m 500 _assembly > filtered_assembly
seqkit stats -a filtered_assembly
```

### 5. Read Mapping

```bash
## Build Bowtie2 indexes from the filtered assemblies
bowtie2-build filtered_assembly filtered_assembly_index

##Align paired-end reads against the filtered assembly
bowtie2 \
    -x filtered_assembly_index \
    -1 reads_R1.fastq.gz \
    -2 reads_R2.fastq.gz \
    -S aligned_reads.sam
```

### 6. Gene Prediction 

#### Predict protein coding genes using Prodigal

```bash
prodigal \
    -i filtered_assembly.fa \
    -o predicted_genes.gff \
    -f gff \
    -a predicted_proteins.faa \
    -d predicted_genes.fna \
    -p meta \
    -g 11
```

## Results

### Quality control stats

|  | Effluent Raw | Effluent Filtered | Influent Raw | Influent Filtered |
|:---|---:|---:|---:|---:|
| Total reads | 43,311,390 | 40,894,194 | 54,374,174 | 50,298,922 |
| Total bases (bp) | 6,496,708,500 | 6,132,376,615 | 8,156,126,100 | 7,541,725,503 |
| GC (%) | 55.65 | 55.75 | 55.72 | 55.91 |
| Q20 (%) | 94.27 | 95.24 | 96.04 | 96.62 |
| Q30 (%) | 87.25 | 88.53 | 90.40 | 91.13 |

### Metagenomic assembly stats

|  | Effluent | Influent |
|:---|---:|---:|
| Number of contigs | 278,073 | 299,949 |
| Total assembly length (bp) | 350,438,110 | 375,511,572 |
| Average contig length (bp) | 1,260.2 | 1,251.9 |
| Minimum contig length (bp) | 500 | 500 |
| Maximum contig length (bp) | 109,580 | 151,538 |
| N50 (bp) | 1,458 | 1,451 |
| GC content (%) | 54.70 | 56.19 |

### Overall Alignment Rates

| Sample | Overall alignment rate |
|---|---:|
| Influent | 83.47% |
| Effluent | 80.40% |

### Gene prediction stats

|  | Effluent | Influent |
|:---|---:|---:|
| Predicted ORFs | 582,878 | 576,834 |
| Total gene length (bp) | 320,944,413 | 333,075,159 |
| Average gene length (bp) | 550.6 | 577.4 |
| Minimum gene length (bp) | 60 | 60 |
| Maximum gene length (bp) | 47,490 | 12,978 |
  
## Next Steps

### 7. Functional Annotation
Assign putative functions to predicted genes.

### 8. Taxonomic Profiling
Characterise the taxonomic composition of the influent and effluent metagenomes.

### 9. Antibiotic Resistance Gene Analysis
Identify and profile potential antibiotic resistance genes.


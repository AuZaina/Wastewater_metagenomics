
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

### 2. Quality Control

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

#### Assess processed reads using FastQC
#### Aggregate quality-control reports using MultiQC

### 3. Metagenomic Assembly

#### Assemble quality-controlled paired-end reads using MEGAHIT

```bash
megahit \
    -1 _R1.fastq.gz \
    -2 _R2.fastq.gz \
    -o _assembly
```

### 4. Contig Filtering and Assembly Statistics using seqkit

#### Retain contigs of at least 500 bp and calculate statistics for filtered assembly

```bash
seqkit seq -m 500 _assembly > filtered_assembly
seqkit stats filtered_assembly
```

### 5. Read Mapping

#### Build Bowtie2 indexes from the filtered assemblies

```bash
bowtie2-build filtered_assembly filtered_assembly_index
```

#### Align paired-end reads against the filtered assembly

```bash
bowtie2 \
    -x filtered_assembly_index \
    -1 reads_R1.fastq.gz \
    -2 reads_R2.fastq.gz \
    -S aligned_reads.sam
```

#### Overall Alignment Rates

| Sample | Overall alignment rate |
|---|---:|
| Influent | 83.60% |
| Effluent | 80.39% |

Alignment rates represent reads mapped to their corresponding filtered assemblies.

## Results

- [MultiQC quality-control report](QC/multiqc/multiqc_report.html)
- [Influent fastp report](QC/fastp/Influent_sample.html)
- [Effluent fastp report](QC/fastp/Effluent_sample.html)
- [Assembly statistics](assembly/filtered_stat)
- [Influent read-mapping log](mapping/influent_align.log)
- [Effluent read-mapping log](mapping/effluent_align.log)

## Next Steps

### 6. Gene Prediction

Predict protein-coding genes from the assembled contigs.

### 7. Functional Annotation

Assign putative functions to predicted genes.

### 8. Taxonomic Profiling

Characterise the taxonomic composition of the influent and effluent metagenomes.

### 9. Antibiotic Resistance Gene Analysis

Identify and profile potential antibiotic resistance genes.


# Project 2 Proposal: Intron Presence and Gene Expression in Arabidopsis thaliana


---

## 1. Project Overview

### Overarching Goal

Investigate the relationship between intron presence and gene expression levels in *Arabidopsis thaliana* to understand whether introns contribute to increased transcript abundance.

---

### Rationale

Introns are non-coding sequences removed during mRNA processing, yet approximately 80% of *Arabidopsis* genes contain at least one intron. This prevalence suggests introns may confer functional advantages beyond being "junk DNA".

**Why this matters:**
- **Intron-mediated enhancement (IME):** Introns have been shown to increase gene expression in plants through mechanisms like enhanced transcription, improved mRNA stability, and increased translation efficiency
- **Agricultural applications:** Identifying expression-enhancing features could improve crop engineering strategies

**Central question:** Do *Arabidopsis* genes with introns exhibit higher expression levels compared to intronless genes?

### Specific Aims

#### **Aim 1: Characterize intron architecture across the Arabidopsis genome**

**Aim statement:** Identify all expressed genes in the TAIR10 reference genome and classify them based on intron presence/absence and intron count.

**Expected outcomes:**
- A comprehensive catalog of intron-containing vs. intronless genes
- Distribution statistics:
  - Percentage of genes with introns vs. without
  - Distribution of intron numbers per gene (0, 1, 2-5, 5+)
  - Average intron length and position (5' UTR, coding sequence, 3' UTR)

**Potential challenges:**
- **Challenge 1:** Incomplete or ambiguous gene annotations
  - *Solution:* Use different methods to annotate the Arabidopsis genome
- **Challenge 2:** Alternative splicing creates multiple isoforms per gene
  - *Solution:* Focus on the primary/canonical transcript for each gene

---

#### **Aim 2: Quantify the relationship between intron presence and gene expression levels**

**Aim statement:** Determine whether genes with introns exhibit significantly different expression levels compared to intronless genes using RNA-seq data.

**Expected outcomes:**
- Statistical comparison of expression levels between:
  - Intron-containing vs. intronless genes
  - Genes grouped by intron count (0, 1, 2-5, 5+)
- Identification of confounding factors (gene length, GC content, chromosomal location)
- Quantification of effect size: How much does intron presence correlate with expression?

**Potential challenges:**
- **Challenge:** Expression level differences may be confounded by gene function (e.g., housekeeping genes vs. tissue-specific genes)
  - *Solution:* I am not sure how to tackle this.

---

## 2. Data

### Dataset Description

#### **Source:**
- **Genome annotation:** TAIR10 (*Arabidopsis thaliana* reference genome)
  - URL: arabidopsis.org/download_files/Genes/TAIR10_genome_release/TAIR10_chromosome_files/TAIR10_chr_all.fas.gz
  - Files needed:
    - `TAIR10_***G.gff` - Gene structure annotations 
    - `TAIR10_genome_release.fasta` - Genome sequence 

- **RNA-seq expression data:** https://www.arabidopsis.org/download/list?dir=Genes%2FTAIR10_genome_release%2FTAIR10_gene_transcript_associations

#### **Size:**
- **Genome:** ~135 MB (FASTA + GFF3)
- **RNA-seq:** 25-100 GB (raw FASTQ files)
- **Processed data:** ~500 MB (count matrices, TPM values)
- **Total storage needed:** ~150 GB (including intermediate files)

#### **Format:**
- **Input:**
  - GFF3 (genome annotation)
  - FASTA (genome sequence)
  - FASTQ (raw RNA-seq reads)
- **Intermediate:**
  - BAM (aligned reads)
  - BED (gene coordinates)
- **Output:**
  - TSV/CSV (gene expression matrices, intron annotations)
  - HDF5 (for efficient storage of large matrices)

---

### Data Suitability

**Current format limitations:**
- FASTQ files are very large and need alignment/quantification

**Necessary transformations:**

1. **Annotation processing:**
   - Parse GFF3 to extract:
     - Gene coordinates
     - Exon/intron boundaries
     - Number of introns per gene
   - Calculate intron features (count, total length, position)

2. **Expression quantification:**
   - Align RNA-seq reads to genome (STAR or HISAT2)
   - Quantify gene expression (featureCounts or Salmon)
   - Normalize to TPM (transcripts per million)

3. **Data integration:**
   - Merge intron annotations with expression values
   - Filter for expressed genes (TPM > 1)

---

### Storage and Data Management

**Storage locations:**
- **Raw data:** Farmshare `/farmshare/user_data/isagolda/project2/raw_data/`
- **Processed data:** `/farmshare/user_data/isagolda/project2/processed/`
- **Results:** Git repository `~/repos/BIOS270-AU25/Project2/`
- **Analysis scripts:** Git repository 

**Backup strategy:**
- **Primary:** Farmshare storage (backed up by Stanford IT)
- **Secondary:** Google Cloud Storage bucket (`gs://bios270-isagolda-project2/`)
- **Code:** GitHub repository (https://github.com/isabelgolda/BIOS270-AU25)

**Sharing with collaborators:**
- **Code:** GitHub (public or private repository)
- **Small data files (<100 MB):** Include in Git repository
- **Large data files:** Share via Google Cloud Storage with read access
- **Results:** Jupyter notebooks with embedded visualizations (maybe?)

## 3. Environment

### Coding Environment

**Primary platform:** Stanford Farmshare or Sherlok HPC cluster
- **Why:** Large RNA-seq files require significant compute resources

---

### Dependencies

**Core tools:**

**RNA-seq processing:**
- STAR (v2.7+) - RNA-seq alignment
- featureCounts (subread v2.0+) - read counting
- or Salmon (v1.9+) - transcript quantification

**Python packages:**
- pandas (v1.5+) - data manipulation
- numpy (v1.23+) - numerical operations
- matplotlib (v3.6+) - visualization
- seaborn (v0.12+) - statistical plots
- scipy (v1.10+) - statistical tests
- scikit-learn (v1.2+) - machine learning
- biopython (v1.80+) - sequence parsing

**File parsing:**
- pysam - BAM file manipulation
- gffutils - GFF3 parsing
- HTSeq - genomic feature counting

---
## 4. Pipeline

### Analysis Workflow

```
Raw Data (FASTQ + GFF3)
    ↓
[1] Genome Annotation Processing
    ↓
Intron catalog (gene_id, intron_count, intron_lengths)
    ↓
[2] RNA-seq Alignment & Quantification
    ↓
Expression matrix (gene_id, TPM values)
    ↓
[3] Data Integration & Filtering
    ↓
Merged dataset (gene_id, intron_features, expression)
    ↓
[4] Statistical Analysis
    ↓
Results (plots, statistics, model predictions)

## 5. Machine Learning
```
---

### Task Definition

**Supervised learning problem:** Predict gene expression level based on intron features and other genomic characteristics.

**Problem type:** Regression

**Why appropriate:** 
- We have labeled data (known expression levels)
- Goal is to predict a continuous value (TPM)
- Can identify which features contribute most to expression

### Feature Representation

**Input features (X):**

1. **Intron features:**
   - `intron_count` - Number of introns (0, 1, 2, 3, ...)
   - `total_intron_length` - Sum of all intron lengths (bp)
   - `avg_intron_length` - Mean intron length
   - `has_intron` - Binary (0 or 1)
   - `intron_density` - Introns per kb of gene
   - `first_intron_length` - Length of first intron (if present)
   - `first_intron_position` - Distance from TSS to first intron

2. **Gene structure features:**
   - `gene_length` - Total gene length (bp)
   - `exon_count` - Number of exons
   - `CDS_length` - Coding sequence length
   - `utr5_length` - 5' UTR length
   - `utr3_length` - 3' UTR length

3. **Sequence features:**
   - `gc_content` - GC percentage
   - `codon_adaptation_index` - Translation efficiency proxy

4. **Genomic context:**
   - `chromosome` - Chromosomal location (1-5)
   - `distance_to_nearest_gene` - Intergenic distance
   - `gene_density` - Genes per Mb in local region

---

### Model Selection

**Models to compare:**

#### **1. Linear Regression (Baseline)**
- **Why:** Simple, interpretable, establishes baseline performance


#### **2. Ridge Regression (L2 regularization)**
- **Why:** Handles multicollinearity between features


#### **3. Random Forest Regressor**
- **Why:** Captures non-linear relationships, handles feature interactions


#### **4. Gradient Boosting (XGBoost)**
- **Why:** Often best performance for tabular data

---

### Evaluation Metrics

#### **Primary metrics:**

**1. R² (Coefficient of Determination)**

**2. RMSE (Root Mean Squared Error)**

**3. MAE (Mean Absolute Error)**

**4. Pearson Correlation**

---
## References

1. Rose, A. B. (2019). Introns as Gene Regulators: A Brick on the Accelerator. *Frontiers in Genetics*, 9, 672.

2. Gallegos, J. E., & Rose, A. B. (2015). The enduring mystery of intron-mediated enhancement. *Plant Science*, 237, 8-15.

3. TAIR - The Arabidopsis Information Resource. https://www.arabidopsis.org/
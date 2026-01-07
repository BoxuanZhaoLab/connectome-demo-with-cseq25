# Connectome-seq Analysis Pipeline

This repository documents a **connectome-seq** computational workflow, including both a standardized single-cell mRNA-seq analysis pipeline and a customized barcode matching analysis pipeline.  
A demo dataset (**cseq25**) and example commands are provided to allow users to reproduce the core analysis steps.

---

## 1. Overview of the Connectome-seq Workflow

The connectome-seq workflow consists of **two main parts**:

### a. Single-cell mRNA Sequencing Analysis

#### i. Standardized Workflow
1. Dataset integration  
2. Clustering and quality control  
3. Cell type annotation (automatic and manual)

#### ii. Output
- Cell type information

---

### b. Barcode Matching Analysis

#### i. Customized Workflow

1. **Cell Ranger processing**  
   Raw sequencing files (`.fastq`) are transformed into BAM files.  
   - mRNA features and Pre/PostRNA features come from separate FASTQ files.  
   - UMIs that differ within **Hamming distance = 1** are automatically collapsed by Cell Ranger.

2. **Custom Python collapsing of Pre/PostRNAs**  
   Pre/PostRNAs with the same cell barcode (CB) that differ by **Hamming distance = 1** are further collapsed.

3. **(Configurable) Disambiguation of Pre/PostRNA assignment**  
   If a Pre/PostRNA is linked to more than two CBs, it is assigned to the CB with the highest Pre/PostRNA UMI count after passing quality checks.  
   
   Current criteria:
   - CB #1 PreRNA > 5 × CB #2 PreRNA  
   - CB #1 PostRNA > 2 × CB #2 PostRNA  
   - CBs ranked by Pre/PostRNA UMI count

4. **Matching between nuclei and synaptosome Pre/PostRNAs**  
   Matching is performed at a specified Hamming distance threshold.

---

## 2. Demo Generation

### a. Demo Dataset
- Dataset used: **cseq25**

### b. Removed Steps in Demo
- Integration, clustering, and manual annotation are **not included** in the demo.

### c. Input and Output Data

#### i. Input
Sequencing results in FASTQ format. Four sets of sequences:
- Pons
- Cerebellum
- Pre-synaptosome
- Post-synaptosome

**Path to raw files**
```
/home/zhouwan2/expansion_link/Cseq1_data/GEOfiles/geo_submission_Cseq_120325
```
(All `.fastq.gz` files start with `cseq25`)

#### ii. Output
A connectome table between **Pons** and **Cerebellum** with **Allen Institute map_my_cell cell type annotations**.

The connectome table includes:
- Matching synaptosome CB  
- Matching Pons nucleus CB  
- Matching Cerebellum nucleus CB  
- Pons nucleus cell type  
- Cerebellum nucleus cell type

---

## 3. Demo Steps

### Step 1. `cellranger count`
Run one-line commands to quantify features from raw sequencing data.

#### Working Commands

```bash
cellranger count   --id=cseq25-PN-new   --transcriptome=/home/boxuan/data/hdd2/mouse_2022   --libraries=library_PN.csv   --feature-ref=feature_ref_2.csv   --force-cells=21000   --localmem 128   --localcores=32   --check-library-compatibility=false
```

```bash
cellranger count   --id=cseq25-CN2   --transcriptome=/home/boxuan/data/hdd2/mouse_cseq6   --libraries=library_CN.csv   --feature-ref=feature_ref_2.csv   --force-cells=4000   --localmem 128   --localcores=32
```

```bash
cellranger count   --id=cseq25-CS-pre2   --transcriptome=/home/boxuan/data/hdd2/mouse_cseq6   --libraries=library_CSpre2.csv   --feature-ref=feature_ref_pre.csv   --expect-cells=8000   --localmem 128   --localcores=32
```

```bash
cellranger count   --id=cseq25-CS-post2   --transcriptome=/home/boxuan/data/hdd2/mouse_cseq6   --libraries=library_CSpost2.csv   --feature-ref=feature_ref_post.csv   --expect-cells=8000   --localmem 128   --localcores=32
```

#### Demo Command Template

```bash
cellranger count   --id=cseq25-PN   --transcriptome=/path/to/mouse_transcriptome   --libraries=library_PN.csv   --feature-ref=feature_ref_2.csv   --force-cells=21000   --localmem 128   --localcores=32
```

Notes:
- Change `id`, `libraries`, and `feature-ref` as needed.
- Use `--force-cells` for **nucleus libraries**.
- Use `--expect-cells` for **synaptosome libraries**.

---

### Step 2. `cellranger subset-bam_linux`

#### Working Commands

```bash
gunzip barcodes.tsv.gz
```

```bash
~/cellranger/subset-bam_linux   --bam ../../cseq1-PN2/outs/possorted_genome_bam.bam   --cell-barcodes barcodes.tsv   --cores 32   --out-bam pons.pre.barcodes.bam
```

#### Demo Instructions

1. Navigate to the `outs/` directory of each `cellranger count` result.
2. Unzip barcode files:
   ```bash
   gunzip barcodes.tsv.gz
   ```

3. Create a new folder for each sample.
4. Copy:
   - **Filtered** `barcodes.tsv` from nucleus samples
   - **Unfiltered (raw)** `barcodes.tsv` from synaptosome samples

5. Extract reads and collapse barcodes.

#### Example Workflow

```bash
gunzip barcodes.tsv.gz
```

```bash
~/cellranger/subset-bam_linux   --bam ../../cseq1-PN2/outs/possorted_genome_bam.bam   --cell-barcodes barcodes.tsv   --cores 32   --out-bam pons.pre.barcodes.bam
```

```bash
../connectome.py pons.pre.barcodes.bam pons.pre.barcodes.hamm1.txt 1
```

- The last argument (`1`) specifies **Hamming distance = 1** for barcode collapsing.
- Current implementation collapses **CB–UMI–N30**.
- Future update: collapse **N30 only**, not CB–UMI–N30.

---

## Notes

This demo pipeline captures the full computational logic of the connectome-seq analysis while remaining lightweight and easy to run.  
It is intended for demonstration, validation, and extension purposes.


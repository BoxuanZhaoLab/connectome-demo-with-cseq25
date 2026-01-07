# Connectome-seq Demo (cseq25)

This repository provides **demonstration code and example workflows** for the *connectome-seq* analysis pipeline, using **cseq25** as a reference dataset.  
The purpose of this repository is to illustrate the **barcode-based connectome construction workflow**, rather than to provide a fully automated end-to-end pipeline.

---

## Overview

The connectome-seq workflow consists of **two major components**:

### 1. Single-cell mRNA sequencing analysis
This component follows a **standard single-cell RNA-seq workflow** and is *not fully reproduced* in this demo.

**Standard workflow:**
1. Dataset integration  
2. Clustering and quality control  
3. Cell type annotation (automatic and manual)

**Output:**
- Cell type annotations for each nucleus, used downstream for connectome construction

> **Note:** In this demo repository, the integration, clustering, and manual annotation steps are omitted. Pre-annotated cell types are used.

---

### 2. Barcode matching analysis (core of connectome-seq)
This component implements a **customized barcode-based workflow** to link nuclei and synaptosome compartments via Pre/PostRNA barcodes.

**Workflow summary:**
1. **Cell Ranger processing**  
   - Converts raw sequencing files (`.fastq`) into BAM files  
   - mRNA features and Pre/PostRNA features are processed from **separate FASTQ libraries**  
   - UMIs differing by **Hamming distance ≤ 1** are automatically collapsed by Cell Ranger  

2. **Custom Python collapsing**  
   - Pre/PostRNAs sharing the same cell barcode (CB) and differing by **Hamming distance = 1** are further collapsed  

3. **Disambiguation of Pre/PostRNA assignments** *(current implementation)*  
   - If a Pre/PostRNA is linked to more than one CB, it is assigned to the dominant CB after quality control  
   - Current criteria:
     - CB #1 PreRNA > 5 × CB #2 PreRNA  
     - CB #1 PostRNA > 2 × CB #2 PostRNA  
     - CBs ranked by UMI count  

4. **Matching nuclei and synaptosome Pre/PostRNAs**  
   - Matching is performed at a user-defined Hamming distance  

---

## Demo dataset: cseq25

### Input data
The demo uses **four sequencing libraries**:
- Pons nuclei  
- Cerebellum nuclei  
- Pre-synaptosome  
- Post-synaptosome  

**Example location of raw FASTQs:**

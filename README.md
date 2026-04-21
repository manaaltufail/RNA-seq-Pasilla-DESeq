 Reference-based RNA-Seq Data Analysis (Galaxy)

A hands-on transcriptomics project following the [Galaxy Training Network (GTN)](https://training.galaxyproject.org/training-material/topics/transcriptomics/tutorials/ref-based/tutorial.html) tutorial for reference-based RNA-Seq data analysis using the pasilla dataset in *Drosophila melanogaster*.

> **Galaxy History:** [View Public History](https://usegalaxy.org/u/wshma/h/copy-of-rna-seq-pasilla-analysis)

---

 Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Tools Used](#tools-used)
- [Workflow Summary](#workflow-summary)
- [Key Results](#key-results)
- [Repository Structure](#repository-structure)
- [How to Reproduce](#how-to-reproduce)
- [References](#references)

---

Overview

This project performs a **reference-based RNA-Seq differential expression analysis** using the Galaxy web platform. The goal is to identify genes differentially expressed between **pasilla gene knockdown (treated)** and **untreated** samples in *Drosophila melanogaster*.

The full pipeline goes from raw FASTQ reads → quality control → trimming → alignment → counting → differential expression → functional enrichment.

| | |
|---|---|
| **Organism** | *Drosophila melanogaster* |
| **Reference Genome** | dm6 (BDGP6) |
| **Annotation** | Ensembl BDGP6.32.109 |
| **Platform** | [Galaxy](https://usegalaxy.org) |
| **Tutorial** | [GTN – Reference-based RNA-Seq](https://training.galaxyproject.org/training-material/topics/transcriptomics/tutorials/ref-based/tutorial.html) |

---

 Dataset

Data comes from a study on the **pasilla gene** — the *Drosophila* homolog of human splicing regulators NOVA1 and NOVA2. Seven samples across two conditions:

| Sample | Condition | Layout |
|---|---|---|
| GSM461176 | Untreated | Single-end |
| GSM461177 | Untreated | Paired-end |
| GSM461178 | Untreated | Paired-end |
| GSM461182 | Untreated | Single-end |
| GSM461179 | Treated (pasilla KD) | Single-end |
| GSM461180 | Treated (pasilla KD) | Paired-end |
| GSM461181 | Treated (pasilla KD) | Paired-end |

**Raw data source:** [Zenodo – pasilla dataset](https://zenodo.org/record/1185122)  
**Reference annotation:** `Drosophila_melanogaster.BDGP6.32.109_UCSC.gtf.gz`

---

Tools Used

| Step | Tool | Purpose |
|---|---|---|
| Quality Control | **Falco** | Check raw read quality |
| QC Aggregation | **MultiQC** | Aggregate and summarize QC reports |
| Adapter Trimming | **Cutadapt** | Remove adapters & low-quality bases |
| Read Mapping | **RNA STAR** | Splice-aware alignment to dm6 genome |
| Duplicate Marking | **Picard MarkDuplicates** | Flag PCR duplicates |
| Mapping QC | **RSeQC** | Gene body coverage, read distribution, strandedness |
| Read Counting | **featureCounts** | Count reads per annotated gene |
| Differential Expression | **DESeq2** | Statistical DE testing (treated vs. untreated) |
| Functional Enrichment | **goseq** | GO/KEGG pathway enrichment of DE genes |

---

Workflow Summary

```
Raw FASTQ reads (7 samples: 4 untreated, 3 treated)
        │
        ▼
 Quality Control ── Falco → MultiQC report
        │
        ▼
 Adapter Trimming ── Cutadapt → MultiQC report
        │
        ▼
 Read Mapping ── RNA STAR → BAM files → MultiQC report
        │
        ▼
 Post-mapping QC
        ├── Picard MarkDuplicates  → duplicate stats
        ├── RSeQC Infer Experiment → strandedness
        ├── RSeQC Gene Body Coverage
        └── RSeQC Read Distribution
        │
        ▼
 Read Counting ── featureCounts → count matrix
        │
        ▼
 Differential Expression ── DESeq2
        ├── PCA plot           (PC1: 48%, PC2: 33% variance)
        ├── Sample distance heatmap
        ├── Dispersion estimates
        ├── MA-plot
        └── p-value histogram
        │
        ▼
 Functional Enrichment ── goseq → GO/KEGG terms
```

---

 Key Results

 featureCounts — Read Assignment

| Sample | Condition | Assignment Rate | % Assigned to Genes |
|---|---|---|---|
| GSM461177_untreat_paired | Untreated | 82.5% | 63.02% |
| GSM461180_treat_paired | Treated | 84.3% | 63.00% |

> ~63% of reads assigned to annotated genes — consistent with expected RNA-Seq results for *D. melanogaster*.

---

 PCA — Sample Separation

PCA was performed on normalized count data across all 7 samples:

- **PC1 captures 48% of variance** — cleanly separates **treated vs. untreated**
- **PC2 captures 33% of variance** — separates **paired-end vs. single-end** library types
- Replicates within each condition cluster tightly, confirming good reproducibility

---

 Differential Expression — DESeq2

Comparison: **Treated (pasilla knockdown) vs. Untreated**

| Metric | Value |
|---|---|
| Total genes tested | 23,932 |
| Significantly DE (padj < 0.05) | **966 genes** |
| Upregulated (\|log2FC\| > 1, padj < 0.05) | **50 genes** |
| Downregulated (\|log2FC\| > 1, padj < 0.05) | **63 genes** |

 Top 5 Upregulated Genes (Treated vs. Untreated)

| Gene ID | log2 Fold Change | Adjusted p-value |
|---|---|---|
| FBgn0025111 | +2.68 | 2.23e-157 |
| FBgn0035189 | +2.36 | 1.59e-35 |
| FBgn0000071 | +2.34 | 3.50e-56 |
| FBgn0261673 | +2.25 | 1.66e-23 |
| FBgn0037290 | +2.12 | 2.30e-21 |

# Top 5 Downregulated Genes (Treated vs. Untreated)

| Gene ID | log2 Fold Change | Adjusted p-value |
|---|---|---|
| FBgn0039155 | -4.08 | 2.81e-171 |
| FBgn0039827 | -3.37 | 1.45e-84 |
| FBgn0003360 | -3.00 | 2.81e-171 |
| FBgn0034736 | -2.93 | 1.48e-62 |
| FBgn0085359 | -2.61 | 5.07e-28 |

---

 Functional Enrichment — goseq (KEGG Pathways)

Top enriched KEGG pathways among DE genes:

| KEGG ID | Pathway |
|---|---|
| 00010 | Glycolysis / Gluconeogenesis |
| 01100 | Metabolic pathways |
| 00030 | Pentose phosphate pathway |
| 00480 | Glutathione metabolism |
| 00280 | Valine, leucine and isoleucine degradation |
| 00071 | Fatty acid degradation |
| 04512 | ECM–receptor interaction |
| 00531 | Glycosaminoglycan degradation |
| 00982 | Drug metabolism |
| 00051 | Fructose and mannose metabolism |

> Enrichment highlights **metabolic reprogramming** and **extracellular matrix changes** associated with pasilla gene knockdown.

---

 Repository Structure

```
RNA-Seq-Reference-Based-Galaxy/
│
├── README.md
│
├── results/
│   ├── multiqc_featurecounts_stats.tabular   ← featureCounts assignment stats
│   ├── multiqc_webpage/                      ← Full MultiQC HTML report (folder)
│   ├── deseq2_results.tabular                ← Full DESeq2 DE table (23,932 genes)
│   ├── deseq2_plots.pdf                      ← PCA, heatmap, MA-plot, dispersion
│   └── goseq_results.tabular                 ← GO/KEGG enrichment results
│
└── workflow/
    └── ref_rnaseq_workflow.ga                ← Exported Galaxy workflow (optional)
```



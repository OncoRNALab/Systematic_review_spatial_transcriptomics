# Systematic_review_spatial_transcriptomics
This repository contains the used scripts for the analyses performed in the systematic review of sequencing-based spatial transcriptomics platforms.

## Part 1: Calculation & data-analysis of the normalized capture performance metric.

## Part 2: Calculation & data-analysis of the captured genes.

# Spatial Transcriptomics Benchmark Scripts Summary

This document provides a comprehensive overview of the four primary R scripts and their companion Slurm job submission wrappers developed for the spatial transcriptomics benchmark pipeline (`review_ST/Gene_metric`).

---

*1. Pipeline Overview & Script Roles*

```mermaid
flowchart TD
    Raw["Raw Count Tables<br/>(Stats_underTissue_all/<br/>*_gene_summary.csv)"] --> GMM["1. gene_metric_matrix.R<br/>Pairwise expression correlation<br/>& log2FC computation"]
    Raw --> GCU["3. gene_content_upset.R<br/>Gene detection overlap,<br/>UpSet plots & Jaccard matrices"]
    Raw --> CC["4. cumulative_contribution.R<br/>Cumulative UMI contribution<br/>& library complexity curves"]
    
    GMM -->|"pairwise_stats_<tissue>.csv"| HGM["2. heatmap_gene_metric.R<br/>Platform & sample heatmaps,<br/>strip plots & matrix exports"]

    subgraph Outputs ["Key Visual & Tabular Deliverables"]
        HGM --> Out1["Correlation & log2FC Heatmaps<br/>(Median, IQR, k comparisons)"]
        GCU --> Out2["UpSet Plots & Platform<br/>Jaccard/Overlap Heatmaps"]
        CC --> Out3["Cumulative UMI Curves & Inset Bars<br/>(Bold median + dashed replicates + IQR)"]
    end
```

---

*2. Master Summary Table*

| Script | Purpose & Scientific Question | Key CLI Arguments | Input Requirements | Main Output Figures (.png / .pdf) | Main Output Tables (.csv) | Companion Slurm Wrapper |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`gene_metric_matrix.R`** | Computes pairwise gene-by-gene expression correlation (Pearson raw, Pearson $\log_{10}$, Spearman) and fold changes ($\log_2\text{FC}$ median and IQR) across all sample pairs. Evaluates cross-platform reproducibility and concordance. | `--tissue_type`<br/>`--cor_scale` (`log10`, `spearman`, `raw`)<br/>`--common` (`pair`, `all`)<br/>`--scatter_mode` (`representative`, `overlay`, `none`)<br/>`--auto_heatmap` (`TRUE`/`FALSE`)<br/>`--subdivide_spots` (`auto`, `all`, `none`) | • `Stats_underTissue_all/*_gene_summary.csv`<br/>• `annotation_gene_metric.xlsx` | • `representative_scatter_<tissue>`<br/>• `representative_MA_<tissue>`<br/>• `gene_matrix_<tissue>` *(legacy overlay)* | • `pairwise_stats_<tissue>.csv`<br/>• `sample_summary_<tissue>.csv` | `submit_gene_metric.sh` |
| **`heatmap_gene_metric.R`** | Aggregates and visualizes pairwise correlation and log2FC matrices at platform and sample levels. Displays within-platform replicate concordance on the diagonal ($NA$ when $n=1$) and cross-platform comparisons off-diagonal. | `--input` (path to pairwise stats)<br/>`--tissue`<br/>`--cor_metric` (`log10`, `spearman`, `both`, `raw`)<br/>`--level` (`both`, `platform`, `sample`)<br/>`--cluster` (`TRUE`/`FALSE`) | • Output of `gene_metric_matrix.R` (`pairwise_stats_<tissue>.csv`)<br/>• Optional: `sample_summary_<tissue>.csv` | • `platform_heatmap_<cor_metric>_<tissue>`<br/>• `platform_heatmap_log2FC_<tissue>`<br/>• `strip_plot_<cor_metric>_<tissue>`<br/>• `sample_heatmap_<cor_metric>_<tissue>` | • `platform_matrix_<metric>_median_<tissue>.csv`<br/>• `platform_matrix_median_log2FC_<tissue>.csv`<br/>• `platform_matrix_IQR_log2FC_<tissue>.csv` | `submit_heatmap_gene_metric.sh` |
| **`gene_content_upset.R`** | Evaluates qualitative gene detection overlap and intersection size (recreation of Supplementary Fig S14). Features top $N$ / top $X\%$ expression ranking, UpSet intersection diagrams, and sample- and platform-level Jaccard and Overlap coefficient heatmaps. | `--tissue_type`<br/>`--top_n` *(e.g. 100, 200)*<br/>`--top_pct` *(e.g. 5, 0.1)*<br/>`--nintersects` *(default 5)*<br/>`--min_umis` *(default 1)*<br/>`--min_spots` *(default 1)*<br/>`--subdivide_spots` (`auto`) | • `Stats_underTissue_all/*_gene_summary.csv`<br/>• `annotation_gene_metric.xlsx` | • `upset_<tissue>[_filter]`<br/>• `jaccard_heatmap_<tissue>[_filter]`<br/>• `overlap_heatmap_<tissue>[_filter]`<br/>• `jaccard_platform_heatmap_<tissue>[_filter]`<br/>• `overlap_platform_heatmap_<tissue>[_filter]` | • `pairwise_set_similarity_<tissue>[_filter].csv`<br/>• `jaccard_platform_summary_<tissue>[_filter].csv` | `submit_gene_content.sh` |
| **`cumulative_contribution.R`** | Analyzes the cumulative contribution of genes to total UMI recovery (recreation of Supplementary Fig S15). Measures dynamic range, library composition, and capture complexity across Panel A (all genes), Panel B (common genes), and Panel C (non-common genes). | `--tissue_type`<br/>`--common_fraction` *(default 1.0; 0.8 recommended)*<br/>`--min_umis`<br/>`--min_spots`<br/>`--subdivide_spots` (`auto`)<br/>`--fig_width` *(default 12)*<br/>`--fig_height` *(default 5.0)* | • `Stats_underTissue_all/*_gene_summary.csv`<br/>• `annotation_gene_metric.xlsx` | • `cumulative_contribution_<tissue>`<br/>*(Bold platform median curve + faint dashed replicate lines + inset bars showing median detected genes with IQR whiskers)* | • `platform_gene_set_summary_<tissue>.csv`<br/>• `gene_set_summary_<tissue>.csv`<br/>• `common_genes_<tissue>.csv` | `submit_cumulative_contribution.sh` |

---


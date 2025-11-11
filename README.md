# Probe Annotation and Gene Mapping for GEO Datasets

## Overview

This R pipeline performs probe-to-gene annotation and mapping for multiple GEO microarray datasets with different chip platforms (GPL8300, GPL6102). Only successfully annotated probes with valid Gene Symbols are retained for downstream analysis.

## Features

- ✓ Automated probe-to-gene symbol mapping
- ✓ Multi-platform support (GPL8300, GPL6102)
- ✓ Quality control and verification checks
- ✓ Comprehensive annotation statistics
- ✓ Multiple output formats (CSV, Excel, RData)
- ✓ Ready for DEG, WGCNA, GO/KEGG analysis

## Requirements

```r
# Install required packages
install.packages(c("dplyr", "tidyr", "openxlsx", "ggplot2"))

# For GEO data download (if needed)
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("GEOquery")
```

## File Structure

Your data directory should be organized as follows:

```
D:/a/
├── probe_annotation_mapping.R          # Main annotation script
├── verify_annotation_results.R          # Verification script
├── GPL8300.txt                          # Platform annotation file
├── GPL6102.txt                          # Platform annotation file
├── GSE7669_expression_matrix.txt        # Expression data
├── GSE10500_expression_matrix.txt       # Expression data
├── GSE15573_expression_matrix.txt       # Expression data
└── annotated_data/                      # Output directory (auto-created)
```

## Preparing Your Data

### 1. Expression Matrix Format

Your expression data files should be tab-delimited text files with:
- Row names: Probe IDs
- Columns: Sample IDs
- Values: Expression values (can be raw, normalized, or log-transformed)

Example:
```
        Sample1   Sample2   Sample3   Sample4
ProbeA  7.234     7.891     6.543     8.012
ProbeB  5.678     5.234     6.012     5.890
ProbeC  9.123     8.765     9.456     8.901
```

### 2. Platform Annotation Files

Platform files can be obtained from:
- **Option A**: Download from GEO (https://www.ncbi.nlm.nih.gov/geo/)
  - Go to your platform page (e.g., GPL8300)
  - Click "Download full table" at the bottom
  - Save as `GPL8300.txt`

- **Option B**: Let the script download automatically (requires GEOquery)

## Usage

### Step 1: Run Main Annotation Script

```r
# Set working directory
setwd("D:/a")

# Source the main script
source("probe_annotation_mapping.R")
```

**What it does:**
1. Loads platform annotation files (GPL8300, GPL6102)
2. Extracts probe-to-gene mappings
3. Loads expression matrices for each dataset
4. Filters probes without gene annotations
5. Exports annotated data in multiple formats
6. Generates summary statistics

### Step 2: Verify Results

```r
# Run verification script
source("verify_annotation_results.R")
```

**What it does:**
1. Checks for missing values
2. Analyzes gene symbol distribution
3. Identifies genes with multiple probes
4. Checks for duplicate probe IDs
5. Validates expression value ranges
6. Creates visualization plots
7. Generates quality report

## Output Files

### annotated_data/ Directory

**Individual Dataset Files (CSV):**
- `GSE7669_annotated.csv` - Annotated expression matrix
- `GSE10500_annotated.csv` - Annotated expression matrix
- `GSE15573_annotated.csv` - Annotated expression matrix

**Summary Files:**
- `annotation_summary.csv` - Per-dataset annotation statistics
- `overall_statistics.csv` - Overall summary metrics
- `platform_statistics.csv` - Platform-wise statistics
- `probe_annotation_complete.xlsx` - All data in one Excel workbook
- `annotation_results.RData` - R objects for quick loading

**Verification Files:**
- `verification_report.xlsx` - Quality check results
- `plots/annotation_retention_rates.png` - Retention rate visualization
- `plots/unique_genes_by_dataset.png` - Unique genes comparison
- `plots/probes_per_gene_*.png` - Distribution plots per dataset

## Output Data Format

Each annotated CSV file has the following structure:

| Probe_ID | Gene_Symbol | Sample1 | Sample2 | Sample3 | ... |
|----------|-------------|---------|---------|---------|-----|
| 1007_s_at | DDR1 | 7.234 | 7.891 | 6.543 | ... |
| 1053_at | RFC2 | 5.678 | 5.234 | 6.012 | ... |

## Annotation Summary Example

The `annotation_summary.csv` provides key metrics:

| Dataset | Platform | Total_Probes | Annotated_Probes | Removed_Probes | Retention_Rate | Unique_Genes | Samples |
|---------|----------|--------------|------------------|----------------|----------------|--------------|---------|
| GSE7669 | GPL8300 | 54675 | 19735 | 34940 | 86.12 | 12563 | 24 |
| GSE10500 | GPL6102 | 22283 | 19568 | 2715 | 87.82 | 11234 | 36 |

## Customization

### Modify Platform-Dataset Mapping

In `probe_annotation_mapping.R`, edit this section:

```r
dataset_platform_map <- list(
  "GSE7669" = gpl8300_mapping,
  "GSE10500" = gpl6102_mapping,
  "GSE15573" = gpl8300_mapping  # Change platform as needed
)
```

### Change File Paths

Update file paths if your data is named differently:

```r
expr_file <- paste0(gse_id, "_expression_matrix.txt")
# Change to: expr_file <- paste0(gse_id, ".txt")
```

### Handle Multiple Probes per Gene

If you want to collapse multiple probes to one value per gene, add after annotation:

```r
# Option 1: Average expression across probes
expr_collapsed <- expr_filtered %>%
  group_by(Gene_Symbol) %>%
  summarise(across(where(is.numeric), mean, na.rm = TRUE))

# Option 2: Select probe with highest variance
expr_collapsed <- expr_filtered %>%
  group_by(Gene_Symbol) %>%
  slice_max(order_by = apply(select(., where(is.numeric)), 1, var),
            n = 1, with_ties = FALSE)
```

## Downstream Analysis Examples

### Load Annotated Data

```r
# Load from RData
load("annotated_data/annotation_results.RData")

# Access specific dataset
gse7669_data <- results_list[["GSE7669"]]

# Or load from CSV
gse7669_data <- read.csv("annotated_data/GSE7669_annotated.csv")
```

### Prepare for DESeq2

```r
# Extract expression matrix
expr_matrix <- gse7669_data %>%
  select(-Probe_ID) %>%
  column_to_rownames("Gene_Symbol")

# Create DESeq2 object
# (Add your phenotype data and design formula)
```

### Prepare for WGCNA

```r
# Transpose for WGCNA (samples as rows, genes as columns)
wgcna_input <- t(expr_matrix)
```

### Prepare for GO/KEGG Analysis

```r
# Get unique gene list
gene_list <- unique(gse7669_data$Gene_Symbol)

# Use with clusterProfiler or other enrichment tools
```

## Troubleshooting

### Issue: Cannot find annotation columns

**Solution:** Check your platform file column names:
```r
names(gpl8300_annot)
```
Then update the column matching in `extract_probe_gene_mapping()` function.

### Issue: Low annotation rate (<70%)

**Possible causes:**
- Wrong platform file for the dataset
- Platform file is incomplete or corrupted
- Expression data uses different probe IDs

**Solution:** Verify dataset-platform pairing on GEO website.

### Issue: Expression files not found

**Solution:** Ensure files are named correctly:
- `GSE7669_expression_matrix.txt`, or
- `GSE7669.txt`

Or modify the file matching pattern in the script.

### Issue: Multiple probes per gene

This is **normal** for microarray data. Each probe represents a different region of the gene.

**Options:**
- Keep all probes (default) - most conservative
- Average expression values - reduces noise
- Select highest variance probe - keeps most informative

## Contact & Support

For issues or questions:
1. Check the verification report for data quality issues
2. Review the annotation summary statistics
3. Examine the generated plots in `annotated_data/plots/`

## Citation

If you use this pipeline, please cite the relevant GEO datasets and platforms used in your analysis.

## License

This script is provided as-is for research purposes.

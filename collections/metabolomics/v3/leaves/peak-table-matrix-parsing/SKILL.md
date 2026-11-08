---
name: peak-table-matrix-parsing
description: Use when input is a spatial metabolomics peak table matrix (e.g. the
  AFADESI mouse brain table with 53,812 pixels x 2,654 ion peaks) that must be loaded
  and split into pixel coordinates and ion intensity columns before SMQVP analysis.
when_to_use_negative:
- Input is raw instrument data (e.g. imaging MS raw files) rather than an already-constructed
  peak table matrix — SMQVP's input requirement is specifically a peak table with
  X/Y coordinates plus m/z intensity columns
- The table lacks the first-two-column X/Y coordinate layout, e.g. it is a plain feature-by-sample
  intensity table without spatial pixel indices
- The goal is metabolite annotation or identification rather than quality visualization
  and preprocessing of pixel-level spatial data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SMQVP v1.0
  role: Consumes the parsed peak table matrix as its Upload Data input for quality
    visualization and preprocessing
  repo: github.com/mzlab-research/SMQVP
- name: R
  role: Environment (with Shiny) in which the peak table is loaded and its coordinate/ion
    columns are split before analysis
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_008
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:932a6543d5280290c090ddf23605ac3b64ee150bc38ceba56ef56d4398ba1b8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# peak-table-matrix-parsing

## Summary

Parse spatial metabolomics peak table matrices in the SMQVP-defined layout — first two columns are pixel X/Y coordinates and remaining columns are m/z ion intensities — so downstream quality visualization and preprocessing modules receive a correctly shaped pixel-by-ion matrix.

## When to use

Use when input is a spatial metabolomics peak table matrix (e.g. the AFADESI mouse brain table with 53,812 pixels x 2,654 ion peaks) that must be loaded and split into pixel coordinates and ion intensity columns before SMQVP analysis.

## When NOT to use

- Input is raw instrument data (e.g. imaging MS raw files) rather than an already-constructed peak table matrix — SMQVP's input requirement is specifically a peak table with X/Y coordinates plus m/z intensity columns
- The table lacks the first-two-column X/Y coordinate layout, e.g. it is a plain feature-by-sample intensity table without spatial pixel indices
- The goal is metabolite annotation or identification rather than quality visualization and preprocessing of pixel-level spatial data

## Inputs

- Peak table matrix with first two columns = pixel X/Y coordinates and subsequent columns = m/z ion intensities (e.g. 53,812 pixels x 2,654 ion peaks from the AFADESI mouse brain, OMIX009541)
- SMQVP-preprocessed peak table matrices retaining the same first-two-column X/Y coordinate layout

## Outputs

- Pixel-coordinate matrix (n_pixels x 2)
- Ion intensity matrix (n_pixels x n_ions) with pixels as observations and ions as features
- Optional coordinate-keyed intensity table ready for SMQVP upload or Louvain clustering

## How to apply

Load the peak table (e.g. from the OMIX009541 deposit at https://ngdc.cncb.ac.cn/omix/release/OMIX009541) with R or pandas, treating the first two columns as pixel X and Y coordinates and all remaining columns as m/z values with intensities at each pixel. Validate the matrix shape against expectations — the demonstrated AFADESI mouse brain example is 53,812 pixels x 2,654 ion peaks. Separate the coordinate block from the ion-intensity block so that pixels are treated as observations and ions as features for quality visualization points such as background consistency (QVP2, Pearson correlation > 0.9 threshold), noise ion detection, and missing value distribution. If the table was produced by a prior SMQVP preprocessing step, apply the same parsing convention since the preprocessed table retains pixel X and Y coordinates in the first two columns.

## Related tools

- **SMQVP v1.0** (Consumes the parsed peak table matrix as its Upload Data input for quality visualization and preprocessing) — github.com/mzlab-research/SMQVP
- **R** (Environment (with Shiny) in which the peak table is loaded and its coordinate/ion columns are split before analysis)

## Examples

```
import pandas as pd; df = pd.read_csv('OMIX009541_peak_table.tsv', sep='\t'); coords = df.iloc[:, :2]; ions = df.iloc[:, 2:]
```

## Evaluation signals

- Parsed matrix dimensions match the expected pixel x ion shape (53,812 x 2,654 for the AFADESI mouse brain example)
- First two parsed columns contain integer pixel X/Y coordinates; all remaining columns are numeric m/z intensities
- Downstream QVP modules run without format errors (e.g. QVP2 background correlations computable and exceeding 0.9; QVP6 missing ratios mostly below 5%)
- Pixel counts after parsing match the number of rows mapped back to spatial coordinates for cluster-map rendering

## Limitations

- Detailed input data requirements are documented only in the tutorial panel within SMQVP, so edge-case format deviations may not be covered by the README
- No changelog or software version is stated for the SMQVP repository, so format expectations could drift across releases

## Evidence

- [readme] The input for SMQVP is a peak table matrix with the following structure: First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each pixel: "The input for SMQVP is a peak table matrix with the following structure:
- First two columns: X and Y coordinates of the pixels
- Subsequent columns: m/z values with corresponding intensities at each"
- [readme] The data includes 53,812 pixels and 2,654 ion peaks available via OMIX009541: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541."
- [intro] This peak table and raw data are available in the public repository: "This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541"
- [intro] Upload your peak table matrix to SMQVP: "Upload your peak table matrix to SMQVP."

---
name: lc-ms-feature-table-generation
description: Use when raw LC–MS metabolomics data (netCDF/mzXML vendor files) must
  be converted into a sample-by-feature table via XCMS peak identification, RT correction,
  and grouping, prior to peak-quality labeling or MetaClean classification.
when_to_use_negative:
- Input is already a pre-processed XCMS object or feature table — do not re-run peak-picking/RT-correction/grouping.
- Data comes from a vendor-processed, uniquely formatted output that cannot be converted
  into an XCMS object — MetaClean downstream requires an XCMS object input.
- Targeted proteomics/peptide data — MetaClean and this pipeline were designed for
  untargeted metabolomics, not targeted MS assays.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: XCMS
  role: Performs peak-picking, retention time correction, and grouping of peaks across
    all samples and QCs to produce the feature table / XCMS object
  repo: https://CRAN.R-project.org/package=MetaClean
- name: IPO
  role: Optimizes XCMS parameters using isotope data prior to peak detection
- name: MetaClean
  role: Consumes the generated XCMS object downstream for peak-quality metric computation
    and classification
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:a23eb5688801fd7ac3703ea0a5ff3517de08921f2836eeee44244bc6904cc0bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  - scmamp
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ms-feature-table-generation

## Summary

Generates a feature table from raw LC–MS metabolomics data by performing peak-picking, retention-time correction, and grouping of similar peaks across samples and pooled QCs using XCMS, with IPO used to optimize XCMS parameters prior to feature detection. This is the upstream pre-processing step required before any peak-quality assessment or downstream statistical analysis.

## When to use

Use when raw LC–MS metabolomics data (netCDF/mzXML vendor files) must be converted into a sample-by-feature table via XCMS peak identification, RT correction, and grouping, prior to peak-quality labeling or MetaClean classification.

## When NOT to use

- Input is already a pre-processed XCMS object or feature table — do not re-run peak-picking/RT-correction/grouping.
- Data comes from a vendor-processed, uniquely formatted output that cannot be converted into an XCMS object — MetaClean downstream requires an XCMS object input.
- Targeted proteomics/peptide data — MetaClean and this pipeline were designed for untargeted metabolomics, not targeted MS assays.

## Inputs

- Raw LC–MS metabolomics data files (e.g. netCDF/mzXML) from untargeted metabolomics studies
- Pooled QC sample injections acquired within the same run as the biological samples
- XCMS peak-picking parameters (optionally IPO-optimized using isotope data)

## Outputs

- XCMS object containing identified, RT-corrected, and grouped peaks across all samples and QCs
- Sample-by-feature intensity table suitable for downstream filtering and statistical analysis
- Per-dataset IPO peak width parameter and related tuning diagnostics

## How to apply

Run XCMS on each dataset to perform peak-picking, retention-time correction, and grouping of peaks across all samples and routinely injected pooled QC samples, yielding a feature table. Optimize XCMS peak-picking parameters with IPO, which tunes parameters using isotope data — the article specifically reports the IPO peak width parameter per dataset as a dataset-characteristic descriptor (e.g. a smaller IPO peak width parameter indicated Test 2 was more dissimilar from other datasets). The output XCMS object serves as the required input format for downstream MetaClean peak-quality metric computation. Judge success by checking that peaks from all samples and QCs have been identified, RT-corrected, and grouped, and that the resulting XCMS object can be consumed by getPeakQualityMetrics() without format errors.

## Related tools

- **XCMS** (Performs peak-picking, retention time correction, and grouping of peaks across all samples and QCs to produce the feature table / XCMS object) — https://CRAN.R-project.org/package=MetaClean
- **IPO** (Optimizes XCMS parameters using isotope data prior to peak detection)
- **MetaClean** (Consumes the generated XCMS object downstream for peak-quality metric computation and classification) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
Rscript xcms_pipeline.R  # IPO-optimized XCMS: peak-picking, RT correction, grouping across samples and pooled QCs to produce the XCMS object consumed by MetaClean
```

## Evaluation signals

- Confirm all biological samples and pooled QC injections were included in the XCMS run before RT correction and grouping
- Verify the IPO-optimized peak width parameter is recorded per dataset and is consistent with peak-shape expectations for that platform
- Check that the resulting XCMS object retains the structure needed by MetaClean (getPeakQualityMetrics expects an XCMS object input)
- Compare peak counts across datasets; a substantially smaller peak count passing RSD<30% or a smaller IPO peak width parameter indicates the dataset is dissimilar from others

## Limitations

- MetaClean downstream requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs
- The quality of downstream peak-quality classification depends on upstream XCMS parameters; the article reports IPO-optimized peak width per dataset, and datasets with different peak width settings were harder to classify
- Characteristics of the LC–MS run (ionization mode, analytical platform/method, biological matrix) affect peak shape and size and therefore affect what constitutes a good integration

## Evidence

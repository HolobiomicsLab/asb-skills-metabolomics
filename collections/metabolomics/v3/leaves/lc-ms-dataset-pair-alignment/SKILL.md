---
name: lc-ms-dataset-pair-alignment
description: Use when two peak-picked, conventionally aligned untargeted LC-MS data
  sets were acquired under different conditions (e.g. different gradient elution)
  and you need to determine overlapping features and concatenate abundances.
when_to_use_negative:
- The datasets were acquired from excessively disparate chromatographic techniques,
  such as between an RPLC dataset and a HILIC dataset
- Input datasets were acquired in different ionization modes (positive vs negative),
  or samples lack biological similarity and metabolomic overlap
- Abundance values have already been normalized or transformed in a way that removes
  ranked abundance order information
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: metabCombiner
  role: R package implementing the full five-step paired alignment workflow
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment for the metabCombiner package
- name: mgcv
  role: Underlying package for fit_gam's spline-fitting of the RT mapping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:f00b11d1d28ba6352caa242f06288a281d94c8e122170dda8b91d8baaa195574
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hhabra/metabCombiner
  repo_url: https://github.com/hhabra/metabCombiner
  tools:
  - GAM
  - R
  - fit_gam
  - metabCombiner
  - mgcv
  - selectAnchors
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# lc-ms-dataset-pair-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align a pair of disparately acquired untargeted LC-MS metabolomics data sets by matching known and unknown <m/z, retention time> features through metabCombiner's five-step workflow, producing a combined table of intersecting feature measurements with a reported weighted per-compound matching accuracy of approximately 90% on plasma data.

## When to use

Use when two peak-picked, conventionally aligned untargeted LC-MS data sets were acquired under different conditions (e.g. different gradient elution) and you need to determine overlapping features and concatenate abundances.

## When NOT to use

- The datasets were acquired from excessively disparate chromatographic techniques, such as between an RPLC dataset and a HILIC dataset
- Input datasets were acquired in different ionization modes (positive vs negative), or samples lack biological similarity and metabolomic overlap
- Abundance values have already been normalized or transformed in a way that removes ranked abundance order information

## Inputs

- Peak-picked and conventionally aligned untargeted LC-MS metabolomics feature table (e.g. plasma20)
- Peak-picked and conventionally aligned untargeted LC-MS metabolomics feature table (e.g. plasma30)
- Optional list of known shared compounds for anchor selection and RT mapping

## Outputs

- Combined table of intersecting feature measurements from both datasets
- Feature pair alignment list ranked by m/z, mapped retention time and relative abundance similarity
- RT mapping spline predictions for overlapping compounds

## How to apply

Run the metabCombiner five-step pipeline on both input tables. First, format and filter features using the retention time range (rtmin/rtmax), missingness (misspc) and duplicate filters (e.g. 0.0025 Da & 0.05 min). Second, group features by m/z to generate a search space of possible feature pair alignments. Third, use selectAnchors to pick retention time ordered pairs from known shared metabolites and fit_gam to fit a spline mapping retention times between the two datasets (mgcv, scaled t-family). Fourth, rank candidate alignments by m/z, mapped retention time and relative abundance similarity. Fifth, reduce and concatenate abundances into a combined table. The paper reports a mean absolute retention time prediction error of roughly 0.06 min on the plasma20/plasma30 pair with more than 50 of 68 test compounds predicted within 0.1 min, and a weighted per-compound matching accuracy of approximately 90%.

## Related tools

- **metabCombiner** (R package implementing the full five-step paired alignment workflow) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment for the metabCombiner package)
- **mgcv** (Underlying package for fit_gam's spline-fitting of the RT mapping)

## Examples

```
library(metabCombiner); d1 <- readData("plasma20.csv"); d2 <- readData("plasma30.csv"); anchors <- selectAnchors(d1, d2); fit <- fit_gam(anchors); result <- calcScores(fit)
```

## Evaluation signals

- Mean absolute retention time prediction error on held-out known compounds should be around 0.06 min (paper's plasma benchmark)
- More than 50 of 68 test compounds in the plasma pair should be predicted within 0.1 min
- Weighted per-compound matching accuracy should approach approximately 90% on the plasma data set pair
- Output combined table should contain only intersecting features from both input tables, with abundances concatenated
- Anchor-identity outliers should be left unfiltered during selectAnchors per the paper's protocol

## Limitations

- Data sets acquired from HILIC methods are more difficult to align than those by RPLC methods
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into alignment
- The preprocessing method and choice of parameters should be considered carefully for each data set
- The output must be corrected for batch effects before carrying out further statistical and bioinformatics analyses

## Evidence

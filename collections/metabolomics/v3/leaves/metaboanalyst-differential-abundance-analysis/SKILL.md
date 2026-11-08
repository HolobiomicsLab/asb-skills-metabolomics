---
name: metaboanalyst-differential-abundance-analysis
description: Use when comparing a MetaboAnalyst-based NMR peak-grouping and differential
  analysis pipeline against an alternative peak-based workflow (e.g. speaq 2.0) on
  the same peak list, or when a lower-resolution grouped feature table is acceptable
  for between-group testing.
when_to_use_negative:
- The input is already a processed feature table with metabolite-level annotations,
  since grouping would only further reduce resolution
- Multiplet information must be preserved, since MetaboAnalyst's grouping sums close
  peaks together and removes multiplet structure
- A high-resolution feature matrix (e.g. speaq 2.0's 677 features rather than 245)
  is required for the downstream statistical analysis
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: MetaboAnalyst
  role: Performs the moving-window peak grouping (0.03 ppm window, 0.015 ppm steps)
    and the FDR-corrected differential analysis on the onion peak list
- name: speaq 2.0
  role: Provides the reference peak-based grouping and differential analysis workflow
    used as the higher-resolution (677-feature) comparator
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributes speaq 2.0 functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix)
    used to build the comparator feature matrix
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Execution environment for loading the onion peak data, running the speaq 2.0
    workflow, and compiling the comparison table
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:e20f7bf4929bf2689552762c332beb5d228af6c732b5215b6ce25341f47fb036
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - MetaboAnalyst
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metaboanalyst-differential-abundance-analysis

## Summary

Applies MetaboAnalyst's moving-window peak grouping (0.03 ppm window, 0.015 ppm steps) to NMR peak lists and runs Benjamini-Hochberg FDR-corrected differential abundance analysis, producing a lower-resolution feature matrix that can be cross-validated against speaq 2.0 grouping results.

## When to use

Use when comparing a MetaboAnalyst-based NMR peak-grouping and differential analysis pipeline against an alternative peak-based workflow (e.g. speaq 2.0) on the same peak list, or when a lower-resolution grouped feature table is acceptable for between-group testing.

## When NOT to use

- The input is already a processed feature table with metabolite-level annotations, since grouping would only further reduce resolution
- Multiplet information must be preserved, since MetaboAnalyst's grouping sums close peaks together and removes multiplet structure
- A high-resolution feature matrix (e.g. speaq 2.0's 677 features rather than 245) is required for the downstream statistical analysis

## Inputs

- NMR peak list (peak/sample/value data) from a dietary-intervention study, e.g. the publicly available onion intake in mice NMR dataset
- Group labels (dietary groups) for the differential analysis
- speaq 2.0 grouped feature matrix (677 features) for cross-mapping comparison

## Outputs

- MetaboAnalyst moving-window grouped feature matrix (245 features; 0.03 ppm window, 0.015 ppm steps)
- Benjamini-Hochberg FDR-corrected differential abundance results flagging highly significant features between groups
- Cross-mapping comparison table (e.g. Table 2) recording coverage of MetaboAnalyst features by speaq 2.0 counterparts and the 677 vs 245 resolution difference

## How to apply

Process the same NMR peak list (e.g. the onion intake in mice data) in MetaboAnalyst using its moving-window grouping with a 0.03 ppm window stepped in 0.015 ppm increments, which yields a coarser feature matrix (245 features vs speaq 2.0's 677) because MetaboAnalyst sums close peaks together, removing multiplet information. Run the Benjamini-Hochberg FDR-corrected differential abundance analysis on both the MetaboAnalyst and speaq 2.0 feature matrices to flag highly significant features between dietary groups. Cross-map each highly significant MetaboAnalyst feature onto highly significant speaq 2.0 features to verify coverage and quantify the resolution difference. Rationale: speaq 2.0's wavelet peak-picking plus grouping avoids binning and was designed to complement established tools like MetaboAnalyst by improving peak-list quality and reproducibility.

## Related tools

- **MetaboAnalyst** (Performs the moving-window peak grouping (0.03 ppm window, 0.015 ppm steps) and the FDR-corrected differential analysis on the onion peak list)
- **speaq 2.0** (Provides the reference peak-based grouping and differential analysis workflow used as the higher-resolution (677-feature) comparator) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributes speaq 2.0 functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) used to build the comparator feature matrix) — https://cran.r-project.org/package=speaq
- **R** (Execution environment for loading the onion peak data, running the speaq 2.0 workflow, and compiling the comparison table) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- Each highly significant MetaboAnalyst feature must match at least one highly significant speaq 2.0 feature in the cross-mapping table
- The MetaboAnalyst feature matrix must contain 245 features versus 677 in the speaq 2.0 matrix when both use 0.03 ppm windows with 0.015 ppm steps
- Both pipelines must apply Benjamini-Hochberg FDR correction before flagging highly significant features
- The comparison table must explicitly record the resolution difference attributed to MetaboAnalyst summing close peaks together

## Limitations

- MetaboAnalyst's moving-window grouping sums close peaks together, giving lower resolution (245 vs 677 features) and removing multiplet information
- The absence of standardized and automated NMR metabolomics workflows means substantial user intervention can introduce bias and lower reproducibility
- Binning/bucketing-style summarization suffers from difficult bin boundary definition in crowded spectra and intrinsic information loss

## Evidence

- [results] MetaboAnalyst lower resolution compared to speaq 2.0: [evidence span withheld — anchor status: not_found]
- [intro] speaq 2.0 designed to complement MetaboAnalyst: "to construct a method that can complement established tools for NMR data analysis like MetaboAnalyst [12], by improving performance, analysis quality and reproducibility"
- [abstract] speaq 2.0 workflow avoids binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] speaq package provides differential analysis functions: [evidence span withheld — anchor status: not_found]
- [intro] binning information loss: "Loss of information is intrinsically linked to the binning approach as entire bins are simply summed together."

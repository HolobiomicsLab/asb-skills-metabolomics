---
name: msms-neutral-loss-annotation
description: Use when a filtered feature table contains a candidate metabolite feature
  (e.g. m/z 185.1183) whose identity must be supported by MS/MS evidence such as a
  diagnostic neutral loss before accepting it in the final metabolite set.
when_to_use_negative:
- Input is unprocessed LC/MS raw data requiring xcms feature detection and CAMERA
  grouping first — use the Metaboseek feature-detection workflow instead
- The candidate feature has no associated MS2 scans (MS1-only acquisition), in which
  case neutral-loss confirmation is impossible and annotation remains at lower confidence
  (Level 3 at best)
- The goal is de novo molecular formula prediction rather than fragment-based annotation
  (use SIRIUS-based formula prediction with the Seven Golden Rules filters instead)
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metaboseek
  role: Provides the MS2 Browser module to locate MS2 scans by parent-mass (ppm) and
    retention-time tolerance and inspect neutral-loss fragments; SaveTableModule stores
    results as .mskFT
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Used via Metaboseek's graphical user interface to detect and align molecular
    features that feed the MS2 annotation step
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used together with xcms to load results into Metaboseek as a Feature Table
    for fragmentation analysis
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:9034e22486ca4b5409b8d2de7c7c712fce79b86e864c1bf1aaf5e200a39d92f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msms-neutral-loss-annotation

## Summary

Annotates CA-derived metabolite features by locating their MS2 scans in the Metaboseek MS2 Browser (parent-mass ppm and retention-time tolerance search) and confirming characteristic neutral-loss fragments, e.g. the formic-acid loss identifying the alpha-hydroxylated citronellic acid derivative at m/z 185.1183.

## When to use

Use when a filtered feature table contains a candidate metabolite feature (e.g. m/z 185.1183) whose identity must be supported by MS/MS evidence such as a diagnostic neutral loss before accepting it in the final metabolite set.

## When NOT to use

- Input is unprocessed LC/MS raw data requiring xcms feature detection and CAMERA grouping first — use the Metaboseek feature-detection workflow instead
- The candidate feature has no associated MS2 scans (MS1-only acquisition), in which case neutral-loss confirmation is impossible and annotation remains at lower confidence (Level 3 at best)
- The goal is de novo molecular formula prediction rather than fragment-based annotation (use SIRIUS-based formula prediction with the Seven Golden Rules filters instead)

## Inputs

- Filtered feature table (.mskFT preferred, or .csv with 'mz' and 'rt' columns)
- MS/MS raw data files (e.g. mzML) associated with the feature table
- Parent m/z (e.g. 185.1183) and retention time of the candidate feature

## Outputs

- Annotated feature with confirmed MS2 diagnostic neutral loss (formic-acid loss for the alpha-hydroxylated citronellic acid derivative)
- Updated feature table saved in .mskFT format with complete processing history

## How to apply

Load the filtered feature table (with 'mz' and 'rt' columns; .mskFT preferred) into Metaboseek, then in the MS2 Browser locate the MS2 scans associated with the feature by searching on its parent mass within a ppm tolerance and a retention-time tolerance around the feature's 'rt' value. Examine the fragmentation spectrum for the expected neutral loss — here the loss of formic acid (46 Da) that is characteristic of the alpha-hydroxylated citronellic acid derivative (compound 7). Optionally apply the MS2 peak-quality filters exposed by Metaboseek, such as 'Noise level in %' (excluding peaks below a set percentage of the maximum peak intensity in a scan), 'min. peaks', 'ignore small fragments' (m/z < 100), and a 'Cosine threshold' for spectral comparisons. Retain the feature as an annotated compound only when the diagnostic neutral loss is present in its MS/MS spectrum, and save the result via SaveTableModule in .mskFT format to preserve the processing history.

## Related tools

- **Metaboseek** (Provides the MS2 Browser module to locate MS2 scans by parent-mass (ppm) and retention-time tolerance and inspect neutral-loss fragments; SaveTableModule stores results as .mskFT) — https://github.com/mjhelf/Metaboseek
- **xcms** (Used via Metaboseek's graphical user interface to detect and align molecular features that feed the MS2 annotation step) — https://github.com/sneumann/xcms
- **MSnbase** (Used together with xcms to load results into Metaboseek as a Feature Table for fragmentation analysis) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- The MS/MS spectrum of the m/z 185.1183 feature shows the characteristic neutral loss of formic acid expected for the alpha-hydroxylated citronellic acid derivative (7)
- MS2 scans are retrieved within the specified parent-mass ppm and retention-time tolerances around the feature's 'rt' and 'mz' values
- The saved .mskFT file preserves the complete processing history including the annotation step
- Cross-check against Supplementary Fig. 3 confirms the feature is part of the reported set of 32 CA-derived metabolites

## Limitations

- Metabolite annotations are limited to confidence Level 3 (tentative structure or compound class assignments only)
- 'ignore small fragments' is described as an experimental feature that excludes peaks with m/z < 100 m/z from spectra
- Many detected features remain unidentified; annotation depends on the availability of matching MS2 scans in the raw data

## Evidence

- [abstract] Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis: "Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis"
- [methods] Load feature table via xcms/MSnbase: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [methods] Noise level intensity filter for MS2 peaks: "peaks at an intensity below a set percentage of the maximum peak 
intensity in a scan (`Noise level in %`) will be excluded"
- [methods] Small-fragment exclusion: "You can also
`ignore small fragments`, an experimental feature that will exclude peaks with 
m/z < 100 m/z from the spectra"
- [readme] Metaboseek as interactive MS data browser: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."

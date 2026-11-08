---
name: mz-rt-targeted-feature-verification
description: Use when a comparative statistical result flags specific m/z values (e.g.
  hacl-1-enriched features) and you must confirm the raw chromatographic evidence
  — presence/absence and intensity in each genotype/diet group — by direct EIC inspection
  at known m/z and RT.
when_to_use_negative:
- Input is raw, unprocessed LC/MS data with no xcms feature detection or grouping
  performed yet — run feature detection first
- The goal is unbiased discovery of new features rather than verification of specific
  candidate m/z/RT values
- The data are not comparative LC/MS metabolomics samples (e.g. GC-MS or spectra without
  retention-time chromatography)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Metaboseek
  role: Web-based GUI used to load the xcms feature table, regroup samples, and perform
    RT-window/ppm-based grouped EIC inspection of target features
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Underlying package for feature detection and alignment of molecular features
    from LC/MS data before EIC verification
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Package used together with xcms to load xcms results into Metaboseek as a
    Feature Table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:d628d502018b237f9209831644dcc8752c0194436cda009418a8348b1e9d8989
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - R
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mz-rt-targeted-feature-verification

## Summary

Targeted verification of specific metabolomic features at candidate m/z and retention-time values using Metaboseek's RT-window and ppm-based extracted ion chromatogram (EIC) inspection, e.g. confirming that cyclopropane-containing N-acyl GPEs (m/z 496.2317, GPE 13:1; m/z 570.2674, cyclopropane GLEA) are abolished in worms fed a cyclopropane-deficient bacterial diet.

## When to use

Use when a comparative statistical result flags specific m/z values (e.g. hacl-1-enriched features) and you must confirm the raw chromatographic evidence — presence/absence and intensity in each genotype/diet group — by direct EIC inspection at known m/z and RT.

## When NOT to use

- Input is raw, unprocessed LC/MS data with no xcms feature detection or grouping performed yet — run feature detection first
- The goal is unbiased discovery of new features rather than verification of specific candidate m/z/RT values
- The data are not comparative LC/MS metabolomics samples (e.g. GC-MS or spectra without retention-time chromatography)

## Inputs

- Centroided LC/MS data files (mzML/mzXML) from all biological groups (e.g. WT and hacl-1 worms fed OP50 or JW1653-1 E. coli)
- xcms project folder with sample grouping from the feature detection run
- Feature table in .mskT format with processing history, loaded into Metaboseek
- Target list of m/z values and approximate retention times (e.g. 496.2317, 570.2674)

## Outputs

- Visually inspected grouped EICs showing target features present or absent per sample group
- Exported filtered feature table (.csv) with m/z, rt, group intensities, and fold-changes for diet-dependent features
- Confirmed presence/absence calls for targeted features supporting the reported comparative result

## How to apply

Load the centroided LC/MS files (mzML/mzXML) from all experimental groups into Metaboseek by importing the xcms project folder so sample grouping from feature detection is retained, then load the .mskFT feature table and assign group labels (e.g. genotype WT vs hacl-1, diet OP50 vs JW1653-1) in the Regroup Table tab. Filter the feature table to features reproducibly detected across biological replicates and absent from blanks. From the group-wise intensity comparison, identify candidate diet- or genotype-dependent features and their m/z and RT values. For each target (e.g. m/z 496.2317 and m/z 570.2674), open Grouped EICs and inspect the EIC across a narrow RT window and ppm mass tolerance, visually confirming the peak is present in OP50-fed hacl-1 samples and absent or strongly reduced in JW1653-1-fed samples. Correct application means the visual EIC evidence matches the statistical fold-change table, reproducing the paper's Fig. 5d-f result.

## Related tools

- **Metaboseek** (Web-based GUI used to load the xcms feature table, regroup samples, and perform RT-window/ppm-based grouped EIC inspection of target features) — https://github.com/mjhelf/Metaboseek
- **xcms** (Underlying package for feature detection and alignment of molecular features from LC/MS data before EIC verification) — https://github.com/sneumann/xcms
- **MSnbase** (Package used together with xcms to load xcms results into Metaboseek as a Feature Table)

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- The exported CSV contains rows for the target m/z values (496.2317, 570.2674) with near-zero or absent mean intensities in the JW1653-1-fed hacl-1 group, matching the reported Fig. 5d-f result
- Grouped EICs within the RT window and ppm tolerance show a clear chromatographic peak in OP50-fed samples that is absent or strongly reduced in JW1653-1-fed samples
- The feature table retains group labels (genotype and diet) so group-wise intensities can be compared directly
- Features retained in the filtered table are reproducibly detected across biological replicates and absent from blank samples
- Visual EIC intensity ranking is consistent with the statistical group-wise fold-change computed from the feature table

## Limitations

- Metaboseek annotates metabolites only up to confidence Level 3 (tentative structure or compound-class assignments), so EIC verification supports tentative annotation, not definitive structural confirmation
- EIC inspection depends on prior xcms feature detection and grouping quality; misalignment or missed features will propagate
- Many detected features remain unidentified, so absence of a feature in a diet group cannot alone identify the molecular species

## Evidence

- [methods] graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [methods] load xcms results into the app as a Feature Table: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [intro] Processing of metabolomics data involves three major steps: "Processing of metabolomics data involves three major steps – feature detection, comparative statistical analysis, and structural characterization"
- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data.: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."

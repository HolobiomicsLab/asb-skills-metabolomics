---
name: ms2-spectral-quality-assessment
description: Use when you need to quantify how 'good' tandem MS spectra are—e.g. comparing
  DDA vs DIA(SWATH) vs DaDIA MS2 quality—by scoring each spectrum against a standard
  spectral library.
when_to_use_negative:
- When the input is a raw mzXML without prior peak picking — XCMS/DaDIA.R must first
  extract features and assemble MS2 spectra before dot products can be computed.
- When no reference spectral library is available or the compound classes fall outside
  MoNA coverage; dot-product grading is meaningless without a matched standard spectrum.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DaDIA.R
  role: Extracts metabolic features and assembles MS2 spectra from the combined DDA/DIA
    data; the resulting spectra are the input to dot-product grading.
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: centWave-based peak picking on the mzXML files before MS2 spectra are scored;
    version 3.11.4+ is required to run DaDIA.R.
- name: R
  role: Runtime environment for DaDIA.R and for computing/bucketing the dot-product
    scores.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:87a37730766c0994736706d6b42466625cbcab005f96ae5c77de475e2ee5e14f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ms2-spectral-quality-assessment

## Summary

Grades experimental MS2 spectra by computing dot-product scores against the MoNA standard MS2 spectral library (91,819 spectra) and binning them into high/low quality classes with a 0.7 dot-product cutoff, as used in the DaDIA urine and leukemia metabolomics workflows.

## When to use

Use when you need to quantify how 'good' tandem MS spectra are—e.g. comparing DDA vs DIA(SWATH) vs DaDIA MS2 quality—by scoring each spectrum against a standard spectral library.

## When NOT to use

- When the input is a raw mzXML without prior peak picking — XCMS/DaDIA.R must first extract features and assemble MS2 spectra before dot products can be computed.
- When no reference spectral library is available or the compound classes fall outside MoNA coverage; dot-product grading is meaningless without a matched standard spectrum.

## Inputs

- Experimental MS2 spectra (from DaDIA.R/xcms processing of DDA + DIA mzXML files)
- MoNA MS spectral library (91,819 standard MS2 spectra, downloaded 2020-09-20)

## Outputs

- Per-spectrum dot-product scores against MoNA
- High/low MS2 quality fractions per acquisition mode (e.g. ms2_quality_distribution.csv with DDA vs DIA provenance split)

## How to apply

Load the experimental MS2 spectra (extracted via DaDIA.R / XCMS ≥3.11.4 from the combined DDA and DIA mzXML files) and match each one against the MoNA standard MS2 library of 91,819 spectra. Compute the dot product between each experimental spectrum and its best library match, then classify spectra using the dot-product threshold of 0.7 that the paper uses for annotation. Binning the scores (e.g. 90–100 vs lower ranges) lets you compare the fraction of high-quality MS2 spectra across acquisition modes (13% for DaDIA vs 7% for DIA(SWATH) in the urine study). The same scores also serve as the annotation gate: only metabolites with a dot product score higher than 0.7 are kept. Run the grading inside R after XCMS peak picking so the scores tie back to the extracted metabolic features.

## Related tools

- **DaDIA.R** (Extracts metabolic features and assembles MS2 spectra from the combined DDA/DIA data; the resulting spectra are the input to dot-product grading.) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (centWave-based peak picking on the mzXML files before MS2 spectra are scored; version 3.11.4+ is required to run DaDIA.R.)
- **R** (Runtime environment for DaDIA.R and for computing/bucketing the dot-product scores.)

## Examples

```
Rscript -e 'source("DaDIA.R"); graded <- gradeMS2(extracted_ms2, moNA_lib); write.csv(graded, "ms2_quality_distribution.csv", row.names=FALSE)'
```

## Evaluation signals

- Reproduce the urine-study distribution: 13% of DaDIA MS2 spectra in the 90–100 dot-product range vs 7% for DIA(SWATH).
- Confirm the low-quality bucket matches the paper: 25% (DaDIA) vs 43% (DIA(SWATH)).
- Verify the DDA/DIA provenance split of DaDIA MS2 spectra: 63% DDA-derived vs 37% DIA-derived.
- Check that the 0.7 dot-product cutoff is consistently applied as the annotation gate before flagging any spectrum as high quality.
- Sanity-check that the MS2 feature counts (956 DDA, 4024 DIA, 4901 DaDIA) are consistent with the number of scored spectra.

## Limitations

- Requires XCMS ≥3.11.4 and metaMS 1.25.1; older versions will fail to run DaDIA.R.
- The MoNA library used (91,819 spectra, downloaded September 20, 2020) sets an upper bound on scoreable compounds — unlisted metabolites cannot be graded.
- Dot-product grading only works after manual deconvolution of MS2 spectra (see supplementary Text S-8); mis-deconvoluted spectra inflate scores.
- Rescued DDA features have a 17% false-positive rate per manual EIC inspection, so some scored spectra may not correspond to a real chromatographic peak.

## Evidence

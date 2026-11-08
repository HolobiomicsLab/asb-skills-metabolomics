---
name: savitzky-golay-smoothing-of-chromatographic-signals
description: Use when EICs extracted from mzML/mzXML LC-MS standard-mixture runs contain
  baseline drift or jagged peaks that must be corrected and smoothed before computing
  per-peak quality metrics like MCQ, max intensity, or asymmetry factor.
when_to_use_negative:
- Input is already a table of derived peak metrics (MCQ, intensity, Std/Blk) — smoothing
  applies to raw EIC intensity traces, not to downstream metric tables
- Goal is mobile-phase comparison/selection from metric summaries rather than per-peak
  signal preprocessing of individual chromatograms
- Data are direct-infusion MS or non-chromatographic spectra where no retention-time
  axis exists to define an EIC
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3361
tools:
- name: MetaMOPE
  role: Containerized web tool that generates standard analytes' EICs from mzXML/mzML
    and applies IRLS baseline correction and Savitzky-Golay smoothing before computing
    the eight per-peak metrics
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Source code of functions for the multiple per-peak metrics (MCQ, intensity,
    asymmetry factor, etc.) computed on the smoothed EICs is provided in R
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:0c988e9ace923047a22509dcba475b4b9f996b444b5c8143dbb3d4a9c084ba86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - MetaMOPE
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# savitzky-golay-smoothing-of-chromatographic-signals

## Summary

Preprocesses extracted ion chromatograms (EICs) from mzML/mzXML LC-MS metabolomics runs by IRLS baseline correction followed by Savitzky-Golay smoothing, as implemented in the MetaMOPE workflow, so that per-peak metrics (MCQ, maximum intensity, asymmetry factor, FWHM, Std/Blk ratio) are computed on a clean signal.

## When to use

Use when EICs extracted from mzML/mzXML LC-MS standard-mixture runs contain baseline drift or jagged peaks that must be corrected and smoothed before computing per-peak quality metrics like MCQ, max intensity, or asymmetry factor.

## When NOT to use

- Input is already a table of derived peak metrics (MCQ, intensity, Std/Blk) — smoothing applies to raw EIC intensity traces, not to downstream metric tables
- Goal is mobile-phase comparison/selection from metric summaries rather than per-peak signal preprocessing of individual chromatograms
- Data are direct-infusion MS or non-chromatographic spectra where no retention-time axis exists to define an EIC

## Inputs

- mzML or mzXML files from LC-MS runs of chemical standard mixtures (including three-replicate runs)
- Extracted ion chromatograms (EICs) for target standard analytes derived from the input files

## Outputs

- Baseline-corrected, Savitzky-Golay-smoothed EIC traces per analyte
- Eight per-peak metrics computed on the smoothed EICs: MCQ, maximum intensity, RSD of retention time, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, and modality

## How to apply

Read the sample's input mzXML/mzML file and generate standard analytes' EICs first. Apply IRLS (Iteratively Reweighted Least Squares) baseline correction to remove the chromatographic baseline, then apply the Savitzky-Golay filter to smooth the EIC trace; in MetaMOPE these two steps are always applied in this order before any metric calculation. Only after correction and smoothing compute the eight per-peak metrics: (i) MCQ, (ii) maximum intensity, (iii) RSD of retention time, (iv) Std/Blk intensity ratio, (v) jaggedness, (vi) asymmetry factor, (vii) FWHM, and (viii) modality. The rationale is that baseline offset and high-frequency noise distort peak-apex detection and the MCQ/asymmetry values, so smoothing must precede metric computation for reproducible three-replicate evaluation (e.g. the 20-chemical-standard validation with thresholds r=6, c=15%, I=1000).

## Related tools

- **MetaMOPE** (Containerized web tool that generates standard analytes' EICs from mzXML/mzML and applies IRLS baseline correction and Savitzky-Golay smoothing before computing the eight per-peak metrics) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Source code of functions for the multiple per-peak metrics (MCQ, intensity, asymmetry factor, etc.) computed on the smoothed EICs is provided in R) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git; cp .env.example .env; docker-compose up --build  # then access http://localhost:13002 to run the EIC smoothing/evaluation workflow
```

## Evaluation signals

- Smoothed EICs should preserve peak apex positions and retention times relative to the raw trace (retention time is a reported metric and feeds the RSD-of-retention-time threshold c=15%)
- Per-peak metrics (MCQ, max intensity, asymmetry factor) must be computed only after IRLS baseline correction and Savitzky-Golay smoothing, not on the raw EIC
- On the 20-chemical-standard three-replicate benchmark, downstream validation flags should reproduce taurocholate, l-aspartate, and sphingosine as validated, with estradiol (Std/Blk 4.719 < 6) and l-cystine (intensity 915.368 < 1000) excluded
- All eight metrics (MCQ, max intensity, RSD of RT, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) should be present for every evaluated peak

## Limitations

- Thresholds r (Std/Blk), c (RSD of retention time), and I (peak intensity) are user-specified, so different settings change which metabolites are validated even after identical smoothing
- The article provides no discussion/changelog content, so smoothing-specific failure modes or edge-case limitations cannot be enumerated from the source text

## Evidence

- [abstract] IRLS baseline correction and Savitzky-Golay smoothing precede metric computation: "We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing"
- [abstract] EIC retrieval followed by IRLS correction and Savitzky-Golay filtering: [evidence span withheld — anchor status: not_found]
- [abstract] Eight metrics generated per peak after preprocessing: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity"
- [readme] MetaMOPE is the containerized tool providing the chromatography peak evaluation workflow: "We developed a containerized web tool providing a workflow to quickly determine the optimized mobile
phase by batch evaluating chromatography peaks for metabolomics LC-MS studies."

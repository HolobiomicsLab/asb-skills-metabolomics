---
name: ground-truth-referenced-spectral-similarity-evaluation
description: Use when benchmarking MS/MS denoising or spectral-matching methods on
  dilution-series data where clean high-concentration reference spectra of the same
  compounds serve as ground truth.
when_to_use_negative:
- No ground-truth or high-concentration reference spectra exist for the compounds,
  so similarity cannot be anchored to known-clean spectra
- The goal is de novo annotation of unknowns rather than method benchmarking — use
  Denoising Search against a reference library instead
- Compounds fail the fragmentation prerequisite (fewer than two fragment ions, spectral
  entropy <0.5) and cannot be scored meaningfully
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spectral_denoising
  role: Python package providing sd.read_msp for loading spectra, sanitize_spectrum,
    spectral_entropy and entropy_similarity used to score denoised dilution spectra
    against ground truth
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: Spectral Denoising
  role: 'Primary denoising method being benchmarked: electronic_denoising followed
    by formula_denoising before similarity scoring'
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: MS Reduce
  role: Alternative denoising baseline compared against Spectral Denoising in the
    ground-truth similarity evaluation
- name: DNL denoising
  role: Alternative dynamic-noise-level denoising baseline included in the benchmark
    comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:1e479620ecd9bf5d75838421222d29b600c14f56abcae34e29d4d4e48e26e640
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - ClassyFire
  - DNL denoising
  - MS Reduce
  - Spectral Denoising
  - spectral_denoising
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# ground-truth-referenced-spectral-similarity-evaluation

## Summary

Evaluate MS/MS denoising quality by computing entropy similarity between processed low-concentration spectra and 500-pmol high-concentration ground-truth spectra of the same compounds, then determining the lowest injected quantity reaching entropy similarity >0.75 per compound.

## When to use

Use when benchmarking MS/MS denoising or spectral-matching methods on dilution-series data where clean high-concentration reference spectra of the same compounds serve as ground truth.

## When NOT to use

- No ground-truth or high-concentration reference spectra exist for the compounds, so similarity cannot be anchored to known-clean spectra
- The goal is de novo annotation of unknowns rather than method benchmarking — use Denoising Search against a reference library instead
- Compounds fail the fragmentation prerequisite (fewer than two fragment ions, spectral entropy <0.5) and cannot be scored meaningfully

## Inputs

- Dilution-series MS/MS spectra (13 serial dilutions) of 240 reference metabolite standards in .msp format
- 500-pmol ground-truth reference spectra for the same compounds (.msp)
- Compound metadata: SMILES or molecular formula, adduct, precursor m/z
- Denoised spectra from each compared method (Spectral Denoising, MS Reduce, 1% base-peak thresholding, DNL)

## Outputs

- Per-compound, per-dilution-stage entropy similarity scores against ground-truth spectra
- Lowest injected quantity reaching entropy similarity >0.75 per compound per method
- Aggregated explained ion intensity and similarity summaries per method and dilution stage
- benchmark_results.csv

## How to apply

Load the dilution-series MS/MS spectra and the 500-pmol ground-truth spectra (e.g. with sd.read_msp), then sanitize each spectrum by removing zero-intensity ions and sorting by mass; exclude compounds with fewer than two fragment ions (spectral entropy <0.5). For each dilution-stage spectrum, compare it against its corresponding ground-truth spectrum using entropy similarity, explicitly ignoring the remaining precursor ion intensity so scoring focuses entirely on genuine fragment spectra. Aggregate explained ion intensity and similarity scores per method and dilution stage, and record the lowest injected quantity at which each compound reaches entropy similarity >0.75. Validate by checking that denoised spectra reproduce reported benchmarks (e.g. an average 35-fold improvement in the lowest quantity reaching entropy similarity >0.75 versus MS Reduce, 1% base-peak thresholding and DNL), and write per-compound, per-stage results to a CSV such as benchmark_results.csv.

## Related tools

- **spectral_denoising** (Python package providing sd.read_msp for loading spectra, sanitize_spectrum, spectral_entropy and entropy_similarity used to score denoised dilution spectra against ground truth) — https://github.com/FanzhouKong/spectral_denoising
- **Spectral Denoising** (Primary denoising method being benchmarked: electronic_denoising followed by formula_denoising before similarity scoring) — https://github.com/FanzhouKong/spectral_denoising
- **MS Reduce** (Alternative denoising baseline compared against Spectral Denoising in the ground-truth similarity evaluation)
- **DNL denoising** (Alternative dynamic-noise-level denoising baseline included in the benchmark comparison)

## Examples

```
import spectral_denoising as sd; peaks = sd.read_msp('dilution_spectra.msp'); sim = sd.entropy_similarity(peak_denoised, peak_ground_truth, pmz=pmz)
```

## Evaluation signals

- Average similarity of denoised spectra reproduces the published benchmark (e.g. average 35-fold lower lowest quantity reaching entropy similarity >0.75 versus MS Reduce, 1% base-peak thresholding and DNL)
- Denoised spectra lift previously unmatchable dilution spectra above entropy similarity >0.75 (reported: >700 spectra) with ~20% higher compound annotation rates
- Counts of false negatives from alternative methods match expectations (MS Reduce: 107, DNL: 84 compounds losing true fragment ions)
- Spectral entropy drops after denoising (example: S = 1.9 denoised vs S = 3.3 contaminated)
- Precursor ion intensity is excluded from all similarity calculations; compounds with spectral entropy <0.5 are excluded before scoring

## Limitations

- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Hard thresholding based exclusively on spectral similarity is suboptimal for compound annotation; supplement with retention time matching, molecular cross-section comparisons and biological metadata screening
- Spectral matching alone cannot differentiate isomeric compounds and in-source fragments
- Method parameters are optimized for Orbitrap instruments; other mass analyzers require retuning and validation
- Requires Python >=3.8 and <3.13 (RDKit has no Python 3.13 distribution)

## Evidence

- [results] ignore precursor ion intensity: "the remaining abundance of the precursor ion intensities is ignored to focus entirely on genuine fragment spectra"
- [results] exclusion criterion: "A total of 28 compounds were excluded due to insufficient fragmentation with fewer than two fragment ions (spectral entropy <0.5)"
- [abstract] benchmark magnitude: "It improved high confident compound identifications at an average 35-fold lower concentrations than previously achievable"
- [results] false negatives of baselines: [evidence span withheld — anchor status: not_found]
- [discussion] similarity scoring limitation: "Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives."
- [readme] similarity function usage: [evidence span withheld — anchor status: external_doc]

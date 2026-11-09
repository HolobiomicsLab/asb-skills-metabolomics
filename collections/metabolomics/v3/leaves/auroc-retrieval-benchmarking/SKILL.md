---
name: auroc-retrieval-benchmarking
description: Use when evaluating whether a preprocessing step (centroiding, denoising,
  ion filtering) helps or hurts spectral-similarity retrieval, by measuring AUROC
  of cosine scores against a same/different compound label matrix.
when_to_use_negative:
- The goal is spectral library search or metabolite annotation rather than benchmarking
  the effect of preprocessing steps on similarity-based retrieval
- Spectra lack InChIKey annotations or precursor m/z metadata, so the same-compound
  label matrix or the 10 ppm candidate-pair filter cannot be constructed
- You are comparing similarity metrics against each other (e.g. cosine vs. spectral
  entropy) rather than measuring how preprocessing variants change cosine-based discrimination
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Parsing MGF MS/MS spectra and computing pairwise spectral comparisons
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Source of the cosine-scoring code used to compute pairwise similarity matrices
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: MSEntropy
  role: MSEntropy-derived cosine function evaluated at several m/z tolerances for
    the centroided variant
  repo: https://github.com/YuanyueLi/MSEntropy
- name: spectral_denoising
  role: Electronic denoising of MS/MS spectra to generate the denoised spectrum variant
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: spectraverse-analysis
  role: Repository containing the preprocessing and analysis code implementing this
    benchmark
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:62588bba99f7d71df4fd551371069b9746fbbb9c18937c2bf99cc1fdecf8a657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MSEntropy
  - Python
  - SpectralEntropy
  - matchms
  - spectral_denoising
  - spectraverse-analysis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# auroc-retrieval-benchmarking

## Summary

Benchmark the effect of MS/MS preprocessing variants (centroiding, electronic denoising, top-n low-intensity ion removal) by quantifying how well pairwise cosine similarity separates same-compound from different-compound spectra pairs via AUROC. Used in Spectraverse to justify excluding centroiding and denoising from the preprocessing pipeline.

## When to use

Use when evaluating whether a preprocessing step (centroiding, denoising, ion filtering) helps or hurts spectral-similarity retrieval, by measuring AUROC of cosine scores against a same/different compound label matrix.

## When NOT to use

- The goal is spectral library search or metabolite annotation rather than benchmarking the effect of preprocessing steps on similarity-based retrieval
- Spectra lack InChIKey annotations or precursor m/z metadata, so the same-compound label matrix or the 10 ppm candidate-pair filter cannot be constructed
- You are comparing similarity metrics against each other (e.g. cosine vs. spectral entropy) rather than measuring how preprocessing variants change cosine-based discrimination

## Inputs

- MS/MS spectra in MGF format (raw and preprocessed variants) from the Spectraverse deposit, Zenodo DOI 10.5281/zenodo.17870921
- Per-spectrum precursor m/z metadata for candidate-pair filtering (10 ppm tolerance)
- Per-spectrum InChIKey annotations for building the same-compound label matrix (first 14 characters, connectivity block)

## Outputs

- Pairwise cosine similarity matrices per spectrum variant, saved in numpy (.npy) format
- Binary same-compound label matrix derived from InChIKey connectivity blocks
- Per-variant AUROC values assembled into a comparison table
- Figure S3-style plot comparing AUROC across preprocessing variants

## How to apply

Load raw and variant MS/MS spectra (MGF) from the Spectraverse Zenodo deposit (10.5281/zenodo.17870921) and parse them with matchms. Identify candidate spectrum pairs whose precursor m/z values match within a 10 ppm tolerance, then construct a binary same-compound label matrix by comparing the first 14 characters (connectivity block) of the InChIKeys of paired spectra. For each spectrum variant — raw, centroided (custom MSEntropy-derived cosine function evaluated at several m/z tolerances), electronically denoised via the FanzhouKong/spectral_denoising package, and top-n ion retention — compute pairwise cosine similarity matrices using the SpectralEntropy/MSEntropy-derived scoring code and save them in numpy format. Rank cosine scores against the label matrix to compute a per-variant AUROC, then assemble the AUROC values into a comparison table and plot the Figure S3 comparison. Decision rule from the paper: if a preprocessing step (centroiding or denoising) consistently decreases AUROC, it is excluded from the preprocessing pipeline; for top-n ion retention, check whether the AUROC effect has saturated (it had largely saturated when retaining the top-4096 highest intensity fragment ions).

## Related tools

- **matchms** (Parsing MGF MS/MS spectra and computing pairwise spectral comparisons) — https://github.com/matchms/matchms
- **SpectralEntropy** (Source of the cosine-scoring code used to compute pairwise similarity matrices) — https://github.com/YuanyueLi/SpectralEntropy
- **MSEntropy** (MSEntropy-derived cosine function evaluated at several m/z tolerances for the centroided variant) — https://github.com/YuanyueLi/MSEntropy
- **spectral_denoising** (Electronic denoising of MS/MS spectra to generate the denoised spectrum variant) — https://github.com/FanzhouKong/spectral_denoising
- **spectraverse-analysis** (Repository containing the preprocessing and analysis code implementing this benchmark) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
conda env create -f environment.yml; conda activate spectraverse
```

## Evaluation signals

- AUROC for centroided and electronically denoised variants is lower than the raw-spectrum AUROC, matching the reported modest but consistent decrease
- The AUROC decrease from removing low-intensity ions has largely saturated when retaining the top-4096 highest intensity fragment ions
- Same-compound label matrix matches InChIKey comparisons on exactly the first 14 characters (connectivity block) of paired spectra
- Candidate spectrum pairs are restricted to precursor m/z matches within 10 ppm
- Similarity matrices are saved in numpy format and AUROC values assemble into a comparison table reproducing Figure S3

## Limitations

- Both centroiding and electronic denoising modestly but consistently decreased the AUROC, so neither function was included in the Spectraverse preprocessing pipeline — applying either step may reduce downstream similarity-discrimination performance
- The Python package spectral_denoising requires Python between 3.8 and 3.12 (RDkit has no compatible distribution for Python 3.13)
- AUROC here is measured only for cosine similarity against a 10 ppm precursor-m/z-filtered candidate pair set; results may not transfer to other similarity measures or pair definitions

## Evidence

- [results] Both centroiding and electronic denoising modestly but consistently decreased the AUROC, so neither function was included in the preprocessing pipeline: [evidence span withheld — anchor status: not_found]
- [results] removing low-intensity ions also modestly decreased the AUROC, an effect that had largely saturated when retaining the top-4096 highest intensity fragment ions: [evidence span withheld — anchor status: not_found]
- [supplementary] Construct a binary same-compound label matrix by comparing the first 14 characters (connectivity block) of the InChIKeys of paired spectra: [evidence span withheld — anchor status: not_found]
- [supplementary] Identify candidate spectrum pairs whose precursor m/z values match within a 10 ppm tolerance: [evidence span withheld — anchor status: not_found]
- [readme] A key feature of matchms is its ability to apply various pairwise similarity measures for comparing extensive amounts of spectra: [evidence span withheld — anchor status: external_doc]
- [methods] Code used to calculate cosine score: "Code used to calculate cosine score"

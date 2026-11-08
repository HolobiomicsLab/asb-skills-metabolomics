---
name: electronic-msms-spectrum-denoising
description: Use when evaluating whether electronic noise-ion removal improves or
  degrades spectral-similarity discrimination (e.g., AUROC of cosine similarity for
  same vs. different compound pairs), or when deciding whether to include denoising
  in an MS/MS library preprocessing pipeline.
when_to_use_negative:
- The goal is chemical-noise removal rather than electronic-noise removal — the package
  targets both noise categories but the AUROC evidence here covers electronic denoising
  specifically
- The spectra are already cleaned by matchms filters (e.g., library_cleaning yaml
  steps, low-resolution or zero/uniform-intensity removal) and the question is redundancy
  removal, not noise-ion removal
tools:
- name: spectral_denoising
  role: Removes electronic noise ions from MS/MS spectra to produce the denoised spectrum
    variant
  repo: FanzhouKong/spectral_denoising
- name: matchms
  role: Parses and preprocesses MGF MS/MS spectra before and after denoising
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides the cosine similarity scoring used to compute pairwise similarity
    matrices for the denoised variant
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: MSEntropy
  role: MSEntropy-derived cosine scoring code used in the AUROC evaluation
  repo: YuanyueLi/MSEntropy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:37f3a7e23d574603b4b1a4f26bcf3d82e654c4bd6e7ac41858735720c8474bce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - MSEntropy
  - Python
  - SpectralEntropy
  - matchms
  - spectral_denoising
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# electronic-msms-spectrum-denoising

## Summary

Applies electronic-noise removal to MS/MS spectra using the FanzhouKong/spectral_denoising package, as one variant in the Spectraverse preprocessing evaluation. In this study, electronic denoising modestly but consistently decreased the AUROC of cosine-similarity same/different-compound discrimination, so it was excluded from the final preprocessing pipeline.

## When to use

Use when evaluating whether electronic noise-ion removal improves or degrades spectral-similarity discrimination (e.g., AUROC of cosine similarity for same vs. different compound pairs), or when deciding whether to include denoising in an MS/MS library preprocessing pipeline.

## When NOT to use

- The goal is chemical-noise removal rather than electronic-noise removal — the package targets both noise categories but the AUROC evidence here covers electronic denoising specifically
- The spectra are already cleaned by matchms filters (e.g., library_cleaning yaml steps, low-resolution or zero/uniform-intensity removal) and the question is redundancy removal, not noise-ion removal

## Inputs

- Raw MS/MS spectra in MGF format (from the Spectraverse v1.0.1 Zenodo deposit, doi 10.5281/zenodo.17870921)
- Spectrum metadata including precursor m/z and InChIKeys for same-compound labeling

## Outputs

- Electronically denoised MS/MS spectra (noise ions removed)
- Pairwise cosine similarity matrices (numpy format) for denoised vs. raw variants
- Per-variant AUROC values comparing cosine-similarity discrimination of same vs. different compounds

## How to apply

Load raw MS/MS spectra in MGF format from the Spectraverse Zenodo deposit (10.5281/zenodo.17870921) and parse them with matchms. Denoise one spectrum variant with the FanzhouKong/spectral_denoising package (Python 3.8–3.12 required, due to RDKit compatibility) alongside a raw, un-denoised control. For both variants, compute pairwise cosine similarity matrices (SpectralEntropy/MSEntropy-derived scoring) over candidate pairs matched within a 10 ppm precursor m/z tolerance, labeling pairs as same-compound when the first 14 InChIKey characters (connectivity block) match. Rank cosine scores against the binary label matrix to compute AUROC per variant; if denoising lowers AUROC relative to raw spectra, exclude it from the pipeline, as the authors did.

## Related tools

- **spectral_denoising** (Removes electronic noise ions from MS/MS spectra to produce the denoised spectrum variant) — FanzhouKong/spectral_denoising
- **matchms** (Parses and preprocesses MGF MS/MS spectra before and after denoising) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides the cosine similarity scoring used to compute pairwise similarity matrices for the denoised variant) — https://github.com/YuanyueLi/SpectralEntropy
- **MSEntropy** (MSEntropy-derived cosine scoring code used in the AUROC evaluation) — YuanyueLi/MSEntropy

## Evaluation signals

- AUROC for the denoised variant should be computed against the same binary same-compound label matrix (InChIKey first-14-character connectivity block) as the raw variant, so the delta AUROC is directly attributable to denoising
- Expected qualitative result: electronic denoising modestly but consistently decreases AUROC relative to raw spectra (which is why the authors excluded it from the pipeline)
- Denoised spectra must still be parsable by matchms and retain valid precursor m/z so that the 10 ppm precursor-matching step applies
- Pairwise similarity matrices should be saved in numpy format for all variants to allow direct comparison and reproduction of the Figure S3-style plot

## Limitations

- Denoising degraded the outcome: both centroiding and electronic denoising modestly but consistently decreased the AUROC, so neither was included in the preprocessing pipeline
- The spectral_denoising package requires Python between 3.8 and 3.12 (RDKit has no distribution compatible with Python 3.13)
- RDKit dependency limits which environments the denoising step can run in; use a dedicated environment when combining with other tools

## Evidence

- [readme] Noise ions in MS/MS spectra are largely categorized as 1. electronic noises and 2. chemical noises.: [evidence span withheld — anchor status: external_doc]
- [other] Both centroiding and electronic denoising modestly but consistently decreased the AUROC, so neither function was included in the preprocessing pipeline: [evidence span withheld — anchor status: not_found]
- [other] electronically denoised via the FanzhouKong/spectral_denoising package: [evidence span withheld — anchor status: not_found]
- [readme] Please use Python version between 3.8 to 3.12 for this package to work.: [evidence span withheld — anchor status: external_doc]

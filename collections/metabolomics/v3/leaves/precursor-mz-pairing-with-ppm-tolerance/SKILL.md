---
name: precursor-mz-pairing-with-ppm-tolerance
description: Use when you need candidate spectrum pairs from an MS/MS library whose
  precursor m/z agree within ~10 ppm, to build same-vs-different-compound labels for
  AUROC evaluation of spectral similarity scoring.
when_to_use_negative:
- Spectra lacking reliable PRECURSOR_MZ or InChIKey metadata — precursor pairing and
  connectivity-block labeling cannot be trusted
- Peak-list data outside small-molecule MS/MS library comparison (e.g., peptide/proteomic
  spectra, where the precursor-ppm pairing and InChIKey labeling scheme does not apply)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: matchms
  role: Parsing MGF files and handling precursor m/z metadata for candidate pair enumeration
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides cosine similarity scoring code evaluated on the paired spectra
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: MSEntropy
  role: Source of the custom MSEntropy-derived cosine function used for pairwise scoring
    of paired variants
  repo: https://github.com/YuanyueLi/MSEntropy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:b6b510575a33e66f49746efc31eb1c363e99440edd8e6f9d50afb53901543704
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MSEntropy
  - Python
  - SpectralEntropy
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precursor-mz-pairing-with-ppm-tolerance

## Summary

Enumerates candidate MS/MS spectrum pairs by matching precursor m/z values within a 10 ppm tolerance, then labels pairs as same-compound via the first 14 characters (connectivity block) of their InChIKeys. This yields the binary same-vs-different-compound label matrix required to evaluate spectral similarity (cosine AUROC) in the Spectraverse pipeline.

## When to use

Use when you need candidate spectrum pairs from an MS/MS library whose precursor m/z agree within ~10 ppm, to build same-vs-different-compound labels for AUROC evaluation of spectral similarity scoring.

## When NOT to use

- Spectra lacking reliable PRECURSOR_MZ or InChIKey metadata — precursor pairing and connectivity-block labeling cannot be trusted
- Peak-list data outside small-molecule MS/MS library comparison (e.g., peptide/proteomic spectra, where the precursor-ppm pairing and InChIKey labeling scheme does not apply)

## Inputs

- Raw MS/MS spectra in MGF format from the Spectraverse Zenodo deposit (10.5281/zenodo.17870921)
- Spectra with precursor m/z (PRECURSOR_MZ) metadata
- Spectra annotated with InChIKeys for connectivity-block comparison
- Spectrum variants to be paired: raw, centroided, electronically denoised, and top-n-ion-retained versions

## Outputs

- Candidate pair list of spectra whose precursor m/z match within 10 ppm
- Binary same-compound label matrix (InChIKey 14-character connectivity block comparison)
- Per-variant AUROC values comparing cosine-score discrimination of same vs. different compounds

## How to apply

Load raw MS/MS spectra in MGF format (from the Spectraverse Zenodo deposit, 10.5281/zenodo.17870921) and parse them with matchms, extracting PRECURSOR_MZ metadata. Identify candidate pairs whose precursor m/z values match within a 10 ppm tolerance — this prunes the 488,630-spectrum library to plausible same-compound candidates without computing all pairwise similarities. For each candidate pair, construct the binary same-compound label by comparing the first 14 characters (connectivity block) of the two InChIKeys. Save the label matrix alongside the per-variant cosine similarity matrices (computed with the SpectralEntropy/MSEntropy-derived scoring code) and rank cosine scores against these labels to obtain AUROC for raw, centroided, denoised, and top-n-ion-retained variants.

## Related tools

- **matchms** (Parsing MGF files and handling precursor m/z metadata for candidate pair enumeration) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides cosine similarity scoring code evaluated on the paired spectra) — https://github.com/YuanyueLi/SpectralEntropy
- **MSEntropy** (Source of the custom MSEntropy-derived cosine function used for pairwise scoring of paired variants) — https://github.com/YuanyueLi/MSEntropy

## Examples

```
from matchms.importing import load_from_mgf; spectra = list(load_from_mgf('data/spectraverse_raw.mgf'))
```

## Evaluation signals

- All enumerated pairs have precursor m/z differences within the 10 ppm tolerance
- Same-compound labels exactly reproduce comparison of the first 14 InChIKey characters (connectivity block) of the paired spectra
- Cosine similarity matrices per variant are saved in numpy format and align with the label matrix dimensions
- AUROC per variant reproduces the paper's direction of effect: centroiding and electronic denoising modestly but consistently decrease AUROC relative to raw spectra

## Limitations

- The AUROC decreases from centroiding and denoising are modest and the low-intensity-ion removal effect saturates by the top-4096 fragment ions, so small AUROC differences may be sensitive to pipeline details
- Labeling depends on correct InChIKey annotations; the article notes public libraries contain spectra with 'missing critical metadata, or which have chemically incoherent annotations'
- Curation relied on manual review of only several thousand spectra, so annotation pitfalls affecting pair labels may remain

## Evidence

- [other] workflow: [evidence span withheld — anchor status: not_found]
- [methods] methods: "Place raw data files (MGF format) in the `data/` directory."
- [intro] intro: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [discussion] discussion: "our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis), accompanied by extensive documentation"

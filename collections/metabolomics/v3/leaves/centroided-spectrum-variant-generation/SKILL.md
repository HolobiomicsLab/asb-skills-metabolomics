---
name: centroided-spectrum-variant-generation
description: Use when you have raw profile-mode MGF spectra from the Spectraverse
  deposit and need to quantify, via AUROC of cosine similarity, whether centroiding
  degrades same-vs-different compound discrimination before deciding whether to include
  a centroiding filter in a preprocessing pipeline.
when_to_use_negative:
- The spectra are already centroided and your goal is only to compare similarity scoring
  functions rather than evaluate whether to add a centroiding filter
- You need electronic or chemical noise removal rather than peak-shape binning; use
  the spectral_denoising variant instead
- The analysis targets instrument-specific peak-picking fidelity rather than the downstream
  effect on spectral-similarity discrimination measured by AUROC
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Parsing MGF spectra and running the filtering/preprocessing steps from which
    the centroided variant is derived
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Source of the cosine similarity scoring code used to compute pairwise similarity
    matrices on centroided spectra
  repo: https://github.com/YuanyueLi/SpectralEntropy.git
- name: MSEntropy
  role: Provides the cosine function from which the custom centroiding/scoring code
    was derived and evaluated at several m/z tolerances
  repo: https://github.com/YuanyueLi/MSEntropy
- name: spectraverse-analysis
  role: Hosts the analysis scripts (matchms zip override, MGF data placement, per-variant
    AUROC computation) implementing this skill
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:76bc0edad39e3cd6b1ba7c5f59f0216215d927d59f874a1be6bfa5359c713ccd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MSEntropy
  - Python
  - SpectralEntropy
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Centroided-spectrum-variant-generation

## Summary

Generates centroided variants of raw profile MS/MS spectra using a matchms/SpectralEntropy-derived cosine scoring framework, to test whether centroiding changes the AUROC with which cosine similarity separates same-compound from different-compound spectrum pairs.

## When to use

Use when you have raw profile-mode MGF spectra from the Spectraverse deposit and need to quantify, via AUROC of cosine similarity, whether centroiding degrades same-vs-different compound discrimination before deciding whether to include a centroiding filter in a preprocessing pipeline.

## When NOT to use

- The spectra are already centroided and your goal is only to compare similarity scoring functions rather than evaluate whether to add a centroiding filter
- You need electronic or chemical noise removal rather than peak-shape binning; use the spectral_denoising variant instead
- The analysis targets instrument-specific peak-picking fidelity rather than the downstream effect on spectral-similarity discrimination measured by AUROC

## Inputs

- Raw profile-mode MGF MS/MS spectra from the Spectraverse Zenodo deposit (10.5281/zenodo.17870921)
- Precursor m/z metadata per spectrum
- InChIKey strings for each spectrum's associated compound

## Outputs

- Centroided spectrum variant (MGF or in-memory matchms Spectrum objects)
- Pairwise cosine similarity matrices for the centroided variant (numpy format)
- AUROC value for the centroided variant, comparable with the raw-spectrum AUROC

## How to apply

Load raw and variant MGF spectra from the Spectraverse Zenodo deposit (10.5281/zenodo.17870921) and parse them with matchms. For the centroided variant, apply the custom MSEntropy-derived cosine function (evaluated across several m/z tolerances) to produce centroided spectra, then compute pairwise cosine similarity matrices, saving them in numpy format. Pair candidate spectra whose precursor m/z values match within 10 ppm and label pairs as same-compound when the first 14 characters (connectivity block) of their InChIKeys agree. Rank cosine scores against this binary label matrix to compute AUROC, and compare against the raw-spectrum AUROC. The article's finding is that centroiding modestly but consistently decreased the AUROC, so it was excluded from the final Spectraverse preprocessing pipeline.

## Related tools

- **matchms** (Parsing MGF spectra and running the filtering/preprocessing steps from which the centroided variant is derived) — https://github.com/matchms/matchms
- **SpectralEntropy** (Source of the cosine similarity scoring code used to compute pairwise similarity matrices on centroided spectra) — https://github.com/YuanyueLi/SpectralEntropy.git
- **MSEntropy** (Provides the cosine function from which the custom centroiding/scoring code was derived and evaluated at several m/z tolerances) — https://github.com/YuanyueLi/MSEntropy
- **spectraverse-analysis** (Hosts the analysis scripts (matchms zip override, MGF data placement, per-variant AUROC computation) implementing this skill) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
python run_matching.py --input data/raw.mgf --variant centroided --mz-tolerance 0.02 --output out/centroided_cosine.npy
```

## Evaluation signals

- The centroided variant's AUROC should be modestly but consistently lower than the raw-spectrum AUROC at the chosen m/z tolerance, matching Figure S3's comparison
- Pairwise cosine similarity matrices for the centroided variant must be saved in numpy format and be symmetric with the same shape as the precursor-matched candidate-pair matrix
- Same-compound labels must derive from the first 14 InChIKey characters; verify that no pair labeled same-compound has differing connectivity blocks
- Precursor matching tolerance must be exactly 10 ppm; any candidate pair outside this window should be excluded from the AUROC computation

## Limitations

- Centroiding is only evaluated in terms of its effect on cosine-similarity AUROC; other quality metrics (e.g. peak-picking accuracy) are not assessed
- The custom cosine function was evaluated at several m/z tolerances, but the article does not report which tolerance was optimal or how sensitive the AUROC decrease is to that choice
- Curation relied on manual review of only several thousand spectra, so pitfalls in centroiding behavior for unreviewed spectra may remain

## Evidence

- [results] Both centroiding and electronic denoising modestly but consistently decreased the AUROC, so neither function was included in the preprocessing pipeline: [evidence span withheld — anchor status: not_found]
- [methods] Custom MSEntropy-derived cosine function evaluated at several m/z tolerances: [evidence span withheld — anchor status: not_found]
- [methods] Precursor m/z matching within 10 ppm and InChIKey connectivity-block labeling: [evidence span withheld — anchor status: not_found]
- [readme] README instruction to replace matchms libraries with the modified zip for filtering: "use the zip file at matchms/matchms.zip and replace it original matchms libraries"

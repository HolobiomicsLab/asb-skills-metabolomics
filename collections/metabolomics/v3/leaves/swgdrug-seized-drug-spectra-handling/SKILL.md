---
name: swgdrug-seized-drug-spectra-handling
description: Use when EI mass spectra peak lists (m/z, intensity) from the SWGDRUG
  library or real seized-drug specimens need to be converted into DeepEI-predicted
  0/1 structural fingerprints as input to the PS2MS integrated similarity ranking.
when_to_use_negative:
- The analyte's chemical structure is already known and a reference spectrum match
  exists — in that case conventional library screening against NIST/Wiley/SWGDRUG
  reference libraries suffices and DeepEI fingerprint prediction is unnecessary
- You need to predict mass spectra of enumerated database compounds from structures
  — that is the NEIMS step of PS2MS, not the DeepEI fingerprint-prediction step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DeepEI
  role: Deep learning model that converts the analyte's EI mass spectrum into its
    corresponding structural fingerprint without requiring the analyte's structure
  repo: https://github.com/jhhung/PS2MS
- name: PS2MS
  role: Pipeline that consumes the predicted fingerprints in the SMSF integrated similarity
    metric and drug-detection step
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime required to run the DeepEI deep learning tools
  repo: https://www.python.org/downloads/
- name: rdkit
  role: Cheminformatics dependency of the PS2MS pipeline; installed from conda alongside
    a source build of the C++ code
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:8d9e9820f9730c53ce9ab3b46d6dca0826be352d3b96eb26c038996be50cb16f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - PS2MS
  - python
  - python 3.6.9
  - rdkit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# swgdrug-seized-drug-spectra-handling

## Summary

Parses EI mass spectra (m/z, intensity peak lists) from the SWGDRUG seized-drug mass spectral library into the input format expected by the DeepEI deep learning model, so that spectra from seized-drug casework can be converted into structural fingerprints for NPS identification with PS2MS.

## When to use

Use when EI mass spectra peak lists (m/z, intensity) from the SWGDRUG library or real seized-drug specimens need to be converted into DeepEI-predicted 0/1 structural fingerprints as input to the PS2MS integrated similarity ranking.

## When NOT to use

- The analyte's chemical structure is already known and a reference spectrum match exists — in that case conventional library screening against NIST/Wiley/SWGDRUG reference libraries suffices and DeepEI fingerprint prediction is unnecessary
- You need to predict mass spectra of enumerated database compounds from structures — that is the NEIMS step of PS2MS, not the DeepEI fingerprint-prediction step

## Inputs

- EI mass spectrum peak lists (m/z, intensity pairs) from the SWGDRUG mass spectral library
- GC/EI-MS spectra from real seized-drug evidence specimens (70 eV electron energy)
- PS2MS repository DeepEI subdirectory (pretrained DeepEI model and fingerprint prediction code)

## Outputs

- predicted_fingerprints.npy — NumPy array of binary 0/1 fingerprint vectors, one per analyte spectrum
- An .msp file with predicted fingerprints merged into the spectral records (as the PS2MS README describes)

## How to apply

Parse each EI spectrum (70 eV GC/EI-MS peak list of m/z and intensity pairs) from the downloaded SWGDRUG mass spectral library into the input format expected by DeepEI, running under Python 3.6.9 or higher with rdkit installed from conda. Run the pretrained DeepEI model on each spectrum to predict its structural fingerprint without needing the analyte's chemical structure — this is the key advantage over NEIMS, which requires a known structure. Collect the predicted binary 0/1 fingerprint vectors into a NumPy array saved as predicted_fingerprints.npy, and optionally merge the fingerprints into the .msp file as the PS2MS README describes. The resulting fingerprints feed the SMSF integrated similarity metric used by PS2MS's drug-detection step to rank candidate identities against the synthetic NPS database.

## Related tools

- **DeepEI** (Deep learning model that converts the analyte's EI mass spectrum into its corresponding structural fingerprint without requiring the analyte's structure) — https://github.com/jhhung/PS2MS
- **PS2MS** (Pipeline that consumes the predicted fingerprints in the SMSF integrated similarity metric and drug-detection step) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime required to run the DeepEI deep learning tools) — https://www.python.org/downloads/
- **rdkit** (Cheminformatics dependency of the PS2MS pipeline; installed from conda alongside a source build of the C++ code) — https://www.rdkit.org/docs/Install.html

## Examples

```
import numpy as np; fps = DeepEI.predict(swgdrug_spectra); np.save('predicted_fingerprints.npy', np.asarray(fps, dtype=int))
```

## Evaluation signals

- Each saved fingerprint in predicted_fingerprints.npy is a well-formed binary vector of the DeepEI output dimensionality (all values exactly 0 or 1)
- Per the paper, fingerprint-based identification using the SMSF metric ranks over 90% of test substances within the top 20 predictions on the NIST test set and over 85% on the SWGDRUG test set
- Incorporating the fingerprint (Jaccard) similarity raises the proportion of correct predictions at rank 1 from below 40% to more than 50%
- The .msp merge step completes without errors and fingerprints are attached to the correct spectral records
- Downstream drug-detection step outputs a ranked list of the hundred most similar compounds per analyte

## Limitations

- DeepEI-predicted fingerprints feed a similarity ranking, not a definitive structure determination; additional postprocessing may be needed to determine precise identities
- PS2MS was designed for NPS whose mass spectra do not match any established reference spectra; using it when reference spectra exist bypasses standard confirmatory workflows (retention time and reference-standard validation)
- No version numbers are stated for DeepEI itself, and the fingerprint prediction depends on a Python 3.6.9+ environment with rdkit built from source plus the conda package

## Evidence

- [methods] DeepEI fingerprint prediction from SWGDRUG spectra: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [methods] SMSF ranking performance on NIST and SWGDRUG: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [methods] Jaccard/fingerprint similarity improves top-1 accuracy: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [readme] README: DeepEI role and msp merge: "PS<sup>2</sup>MS employs DeepEI to predict the fingerprint of the unknown analyte. The system also use the function of DeepEI to calculate the fingerprint of compounds of synthetic database. The"
- [readme] README: Python and rdkit requirements: "[python 3.6.9](https://www.python.org/downloads/) or higher to the run deep learing tools"
- [intro] Conventional library screening context: "The spectra are initially compared with reference spectrum libraries such as the National Institute of Standards and Technology (NIST),6 Wiley,7 and SWGDRUG8 MS libraries"

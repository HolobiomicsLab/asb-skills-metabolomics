---
name: spectrum-to-fingerprint-deep-learning-inference
description: Use when an unknown analyte's GC/EI-MS spectrum fails to match any reference
  library (NIST, Wiley, SWGDRUG) and a structural representation is needed from the
  spectrum alone.
when_to_use_negative:
- The analyte's mass spectrum already matches an established reference spectrum (NIST/Wiley/SWGDRUG),
  so conventional library identification with retention-time confirmation suffices
- You have the analyte's chemical structure and need spectrum prediction rather than
  fingerprint inference — use NEIMS in the forward direction instead
- Direct structure prediction is required rather than a ranked candidate list — DeepEI
  only yields a fingerprint representation, not a structure
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepEI
  role: Pretrained deep learning model that converts the analyte's EI mass spectrum
    into its corresponding structural fingerprint (binary 0/1 vector) without requiring
    the analyte's structure; also used to compute fingerprints of compounds in the
    synthetic NPS database
  repo: https://github.com/jhhung/PS2MS
- name: PS2MS
  role: Pipeline that orchestrates DeepEI fingerprint prediction, merges fingerprints
    into the msp file, and feeds them to the SMSF integrated similarity metric for
    candidate ranking
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime required to run the DeepEI deep learning tool
  repo: https://www.python.org/downloads/
- name: rdkit
  role: Chemistry toolkit dependency of the PS2MS pipeline (install python package
    from conda and build C++ code from source)
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:6444842e0dfd431aece79985dd706bd9ac1f3592bdaac9f23481a30443d22244
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectrum-to-fingerprint-deep-learning-inference

## Summary

Converts an EI mass spectrum (m/z, intensity peak list) into a predicted binary molecular fingerprint using the pretrained DeepEI deep learning model, without requiring knowledge of the analyte's chemical structure. In PS2MS this fingerprint enables structure-aware similarity ranking of candidate NPS identities when no reference spectrum match exists.

## When to use

Use when an unknown analyte's GC/EI-MS spectrum fails to match any reference library (NIST, Wiley, SWGDRUG) and a structural representation is needed from the spectrum alone.

## When NOT to use

- The analyte's mass spectrum already matches an established reference spectrum (NIST/Wiley/SWGDRUG), so conventional library identification with retention-time confirmation suffices
- You have the analyte's chemical structure and need spectrum prediction rather than fingerprint inference — use NEIMS in the forward direction instead
- Direct structure prediction is required rather than a ranked candidate list — DeepEI only yields a fingerprint representation, not a structure

## Inputs

- EI mass spectrum of the unknown analyte as an m/z, intensity peak list (e.g. from GC/EI-MS at 70 eV)
- Mass spectral peak lists parsed from the SWGDRUG mass spectral library
- DeepEI input-format spectrum files processed with Python 3.6.9+

## Outputs

- Predicted binary 0/1 fingerprint vectors per spectrum, of the DeepEI output dimensionality
- NumPy array of predicted fingerprints saved as predicted_fingerprints.npy
- Fingerprints merged into an .msp file for downstream drug-detection comparison

## How to apply

Parse the EI mass spectra (m/z, intensity peak lists), e.g. from the SWGDRUG mass spectral library, into the input format expected by DeepEI using Python 3.6.9 or higher. Run the pretrained DeepEI model on each spectrum to convert it into its corresponding structural fingerprint, then collect the predicted binary 0/1 fingerprint vectors into a NumPy array (e.g. saved as predicted_fingerprints.npy), optionally merging the fingerprints into an .msp file as the PS2MS README describes. The rationale is that existing spectrum predictors like NEIMS require a known chemical structure, whereas DeepEI works in the reverse direction — spectrum to fingerprint — which is what makes identification of unidentified analytes possible. The resulting fingerprint is then fed into the integrated SMSF similarity metric together with the NEIMS-predicted spectra of enumerated derivatives, for which over 90% (NIST) and 85% (SWGDRUG) of test substances ranked within the top 20 predictions.

## Related tools

- **DeepEI** (Pretrained deep learning model that converts the analyte's EI mass spectrum into its corresponding structural fingerprint (binary 0/1 vector) without requiring the analyte's structure; also used to compute fingerprints of compounds in the synthetic NPS database) — https://github.com/jhhung/PS2MS
- **PS2MS** (Pipeline that orchestrates DeepEI fingerprint prediction, merges fingerprints into the msp file, and feeds them to the SMSF integrated similarity metric for candidate ranking) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime required to run the DeepEI deep learning tool) — https://www.python.org/downloads/
- **rdkit** (Chemistry toolkit dependency of the PS2MS pipeline (install python package from conda and build C++ code from source)) — https://www.rdkit.org/docs/Install.html

## Examples

```
python -c "import numpy as np; fps = deepei.predict(spectra); np.save('predicted_fingerprints.npy', np.asarray(fps, dtype=int))"
```

## Evaluation signals

- Each saved fingerprint is a well-formed binary vector of the DeepEI output dimensionality (all values 0/1)
- The array predicted_fingerprints.npy has one fingerprint row per input spectrum with no missing entries
- Merged .msp file contains fingerprint entries consistent with the input peak lists
- Downstream check: on NIST and SWGDRUG test sets, over 90% and 85% of test substances respectively rank within the top 20 predictions under the SMSF metric
- Adding Jaccard similarity on fingerprints raises the top-1 correct-prediction proportion from below 40% to more than 50%

## Limitations

- Fingerprint-based identification is only meaningful for substances in the enumerated synthetic NPS database; the SMSF top-20 recall on SWGDRUG (85%) is lower than on NIST (90%)
- PS2MS was validated on cathinone-like derivatives and real evidence specimens; performance for other NPS classes depends on the chosen core template
- No version numbers are stated for DeepEI in the paper, and no explicit release details beyond the PS2MS repository are given
- The fingerprint is a structural representation only — additional postprocessing is needed to determine precise identities

## Evidence

- [methods] DeepEI converts spectrum to fingerprint: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [readme] README: DeepEI role and msp merge: "PS<sup>2</sup>MS employs DeepEI to predict the fingerprint of the unknown analyte."
- [readme] README: fingerprints merged into msp: "The system merges the fingerprint into the msp file."
- [methods] Top-20 ranking performance: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [methods] Jaccard similarity improves top-1: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."

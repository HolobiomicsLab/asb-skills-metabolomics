---
name: binary-fingerprint-output-validation
description: Use when DeepEI (or a similar fingerprint predictor) has converted EI
  mass spectra into structural fingerprints and you must verify each vector is binary
  and correctly dimensioned before merging into .msp or saving as .npy.
when_to_use_negative:
- The fingerprint predictor is being applied to structures rather than spectra — NEIMS
  handles spectrum prediction from known structures; this validation only covers DeepEI's
  spectrum-to-fingerprint direction where the analyte's structure is unknown.
- Downstream steps only need ranked similarity scores and the fingerprints have already
  been merged into a validated .msp file — re-validating at that stage adds no information.
- The spectra have not yet been run through DeepEI; validation of predicted fingerprints
  requires the model output to exist first.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: DeepEI
  role: Deep learning model whose spectrum-to-fingerprint predictions are validated;
    converts the analyte's EI mass spectrum into its corresponding binary structural
    fingerprint without requiring the analyte's structure.
  repo: https://github.com/jhhung/PS2MS
- name: PS2MS
  role: Pipeline into which validated fingerprints are merged (into .msp files) and
    consumed by the SMSF integrated similarity metric for candidate ranking.
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime (3.6.9 or higher) required to run the DeepEI deep learning tool and
    perform the NumPy-based validation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:d684111b356d731d588db64914d62f49a8094bed763b8ae6fd3d69f1601f4742
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - PS2MS
  - python
  - python 3.6.9
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Binary fingerprint output validation

## Summary

Validates that fingerprints predicted by DeepEI from EI mass spectra are well-formed binary 0/1 vectors of the expected DeepEI output dimensionality before they are stored (e.g., predicted_fingerprints.npy) or merged into an .msp file for downstream PS2MS SMSF similarity ranking.

## When to use

Use when DeepEI (or a similar fingerprint predictor) has converted EI mass spectra into structural fingerprints and you must verify each vector is binary and correctly dimensioned before merging into .msp or saving as .npy.

## When NOT to use

- The fingerprint predictor is being applied to structures rather than spectra — NEIMS handles spectrum prediction from known structures; this validation only covers DeepEI's spectrum-to-fingerprint direction where the analyte's structure is unknown.
- Downstream steps only need ranked similarity scores and the fingerprints have already been merged into a validated .msp file — re-validating at that stage adds no information.
- The spectra have not yet been run through DeepEI; validation of predicted fingerprints requires the model output to exist first.

## Inputs

- Predicted fingerprint vectors from DeepEI (one per analyte EI mass spectrum)
- EI mass spectrum peak lists (m/z, intensity) from the SWGDRUG/NIST mass spectral library
- Expected DeepEI output dimensionality (reference vector length)

## Outputs

- Validated binary 0/1 fingerprint vectors stored in a NumPy array saved as predicted_fingerprints.npy
- Optionally, fingerprints merged into an .msp file (as PS2MS does for the synthetic NPS database)

## How to apply

After parsing EI peak lists (m/z, intensity) and running the pretrained DeepEI model on each spectrum in Python 3.6.9+, collect the predicted vectors and check that every entry is a valid 0/1 binary value (no intermediate floats, NaNs, or missing entries) and that the vector length matches DeepEI's fixed output dimensionality. Store the validated vectors in a NumPy array saved as predicted_fingerprints.npy, optionally merging them into an .msp file as the PS2MS README describes. This validation matters because these fingerprints feed the integrated SMSF metric that ranks enumerated NPS candidates, so malformed vectors would silently corrupt similarity scores. Use per the paper's benchmark as a sanity anchor: fingerprint-based identification should place over 90% and 85% of test substances within the top 20 predictions on the NIST and SWGDRUG test sets, respectively.

## Related tools

- **DeepEI** (Deep learning model whose spectrum-to-fingerprint predictions are validated; converts the analyte's EI mass spectrum into its corresponding binary structural fingerprint without requiring the analyte's structure.) — https://github.com/jhhung/PS2MS
- **PS2MS** (Pipeline into which validated fingerprints are merged (into .msp files) and consumed by the SMSF integrated similarity metric for candidate ranking.) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime (3.6.9 or higher) required to run the DeepEI deep learning tool and perform the NumPy-based validation.)

## Examples

```
import numpy as np; fps = np.load('predicted_fingerprints.npy'); assert fps.shape[1] == DEEPEI_OUTPUT_DIM and np.isin(fps, [0, 1]).all()
```

## Evaluation signals

- Every saved fingerprint is a well-formed binary vector: all values are exactly 0 or 1 with no NaNs or intermediate floats.
- Each fingerprint's length equals the DeepEI output dimensionality; shape of the NumPy array matches (n_spectra, fingerprint_dim).
- Number of validated fingerprints equals the number of parsed EI spectra fed into DeepEI (no dropped or duplicated rows).
- predicted_fingerprints.npy loads cleanly with numpy.load and is dtype-compatible with downstream SMSF computation.
- End-to-end benchmark: fingerprint-based identification on the NIST and SWGDRUG test sets should place over 90% and 85% of test substances, respectively, within the top 20 SMSF-ranked predictions.

## Limitations

- No version numbers are stated for DeepEI in the article, so the exact expected fingerprint dimensionality must be inferred from the pretrained model checkpoint.
- The article reports aggregate benchmark performance (top-20 rates on NIST/SWGDRUG); per-spectrum correctness of individual fingerprint bits cannot be verified without reference structures.
- Fingerprint predictions are only as good as the pretrained DeepEI model; the paper notes existing in silico methods are limited by training data (NEIMS trained solely on NIST library spectra).

## Evidence

- [methods] DeepEI spectrum-to-fingerprint conversion: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [methods] Benchmark performance anchor: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [readme] DeepEI fingerprint of unknown analyte (README): "PS<sup>2</sup>MS employs DeepEI to predict the fingerprint of the unknown analyte."
- [readme] Fingerprint merge into msp file (README): "The system merges the fingerprint into the msp file."
- [readme] Python runtime requirement (README): "[python 3.6.9](https://www.python.org/downloads/) or higher to the run deep learing tools"

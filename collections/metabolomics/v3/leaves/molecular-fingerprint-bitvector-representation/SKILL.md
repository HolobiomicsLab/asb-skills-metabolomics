---
name: molecular-fingerprint-bitvector-representation
description: Use when an unidentified analyte's GC/EI-MS spectrum matches no known
  reference (NIST/Wiley/SWGDRUG) and a structural representation must be derived from
  the spectrum alone to rank enumerated derivative candidates.
when_to_use_negative:
- The analyte's mass spectrum already matches an established reference spectrum in
  the NIST, Wiley, or SWGDRUG libraries — conventional library screening suffices
  and the fingerprint prediction step is unnecessary.
- The chemical structure of the target compound is already known — in that case NEIMS-style
  spectrum prediction from structure (ECFP input) applies, not spectrum-to-fingerprint
  inference, since existing prediction methods 'are limited to cases where the chemical
  structure of NPS is already known.'
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: DeepEI
  role: Pretrained deep learning model that converts the analyte's EI mass spectrum
    into its corresponding structural fingerprint (and computes fingerprints of synthetic
    database compounds merged into the .msp file)
- name: PS2MS
  role: Pipeline that orchestrates fingerprint prediction and feeds the predicted
    fingerprint into the integrated SMSF similarity ranking
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime environment (or higher) required to run the DeepEI fingerprint prediction
  repo: https://www.python.org/downloads/
- name: rdkit
  role: Chemoinformatics library used in the PS2MS environment for handling structures/fingerprints
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:e197bb4e479cfd72aa11ae7049a3d068130f0c03e0c35d5101d3c3dbeddc0d14
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# molecular-fingerprint-bitvector-representation

## Summary

Convert an analyte's EI mass spectrum into its structural molecular fingerprint (binary 0/1 bit vector) using the DeepEI deep learning model, without needing to know the analyte's structure. This predicted fingerprint is the structural half of PS2MS's integrated SMSF similarity metric for ranking candidate NPS identities.

## When to use

Use when an unidentified analyte's GC/EI-MS spectrum matches no known reference (NIST/Wiley/SWGDRUG) and a structural representation must be derived from the spectrum alone to rank enumerated derivative candidates.

## When NOT to use

- The analyte's mass spectrum already matches an established reference spectrum in the NIST, Wiley, or SWGDRUG libraries — conventional library screening suffices and the fingerprint prediction step is unnecessary.
- The chemical structure of the target compound is already known — in that case NEIMS-style spectrum prediction from structure (ECFP input) applies, not spectrum-to-fingerprint inference, since existing prediction methods 'are limited to cases where the chemical structure of NPS is already known.'

## Inputs

- EI mass spectrum of the unknown analyte as a peak list of m/z and intensity pairs
- SWGDRUG or NIST mass spectral library entries (peak lists) for validation
- SMILES structures of enumerated synthetic NPS database compounds (for their DeepEI fingerprints, merged into the .msp file)

## Outputs

- Predicted binary 0/1 molecular fingerprint vector of DeepEI's output dimensionality for each analyte spectrum
- NumPy array of predicted fingerprints saved as predicted_fingerprints.npy
- Fingerprints merged into an .msp file alongside the synthetic NPS database spectra

## How to apply

Parse the analyte's EI mass spectrum (m/z, intensity peak lists, e.g. from the SWGDRUG MS library) into the input format expected by DeepEI, running under Python 3.6.9 or higher. Run the pretrained DeepEI model to map the spectrum to its corresponding structural fingerprint without requiring the analyte's structure; DeepEI is also used by PS2MS to compute fingerprints of the enumerated synthetic NPS database compounds, which are merged into the .msp file. Store the predicted binary 0/1 fingerprint vectors in a NumPy array (predicted_fingerprints.npy) for downstream similarity scoring. The predicted fingerprint is then compared against candidate fingerprints via Jaccard similarity as part of the integrated SMSF metric, which ranks the hundred most similar compounds; with the fingerprint component included, over 90% and 85% of test substances ranked within the top 20 predictions on the NIST and SWGDRUG data sets, and adding Jaccard similarity raised top-1 accuracy from below 40% to more than 50%.

## Related tools

- **DeepEI** (Pretrained deep learning model that converts the analyte's EI mass spectrum into its corresponding structural fingerprint (and computes fingerprints of synthetic database compounds merged into the .msp file))
- **PS2MS** (Pipeline that orchestrates fingerprint prediction and feeds the predicted fingerprint into the integrated SMSF similarity ranking) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime environment (or higher) required to run the DeepEI fingerprint prediction) — https://www.python.org/downloads/
- **rdkit** (Chemoinformatics library used in the PS2MS environment for handling structures/fingerprints) — https://www.rdkit.org/docs/Install.html

## Examples

```
import numpy as np; fps = DeepEI.predict(ei_peak_list(mz, intensities)); np.save('predicted_fingerprints.npy', np.array(fps, dtype=int))
```

## Evaluation signals

- Each saved fingerprint must be a well-formed binary vector containing only 0/1 values at DeepEI's fixed output dimensionality.
- predicted_fingerprints.npy loads as a NumPy array whose row count equals the number of input spectra.
- On NIST and SWGDRUG test sets, the SMSF metric incorporating these fingerprints ranks over 90% and 85% of test substances within the top 20 predictions, respectively.
- Adding the Jaccard similarity on fingerprints increases top-1 prediction accuracy from below 40% to more than 50%.
- Fingerprints of synthetic database compounds can be merged into the .msp file without format errors.

## Limitations

- DeepEI inference requires the mass spectrum only, but the overall PS2MS identification assumes a preselected drug core (e.g., cathinone) for derivative enumeration; results depend on that template choice.
- Spectrum-to-fingerprint prediction quality bounds downstream ranking; additional postprocessing may be needed to determine precise identities.
- No version numbers are stated for DeepEI in the paper's discussion, so exact model version provenance must be checked in the PS2MS repository.

## Evidence

- [methods] PS2MS employs the DeepEI deep learning model to convert the analyte's mass spectrum into its corresponding fingerprint: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [intro] Fingerprint prediction without analyte structure: "The system leverages two deep learning models, NEIMS and DeepEI, to predict the mass spectrum of the enumerated substances in the NPS database and the structure representations (i.e., fingerprints)"
- [readme] PS2MS employs DeepEI to predict the fingerprint of the unknown analyte.: "PS<sup>2</sup>MS employs DeepEI to predict the fingerprint of the unknown analyte."
- [readme] The system merges the fingerprint into the msp file.: "The system merges the fingerprint into the msp file."
- [methods] Top-20 ranking performance of SMSF metric: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [methods] Jaccard similarity improves top-1 accuracy: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."

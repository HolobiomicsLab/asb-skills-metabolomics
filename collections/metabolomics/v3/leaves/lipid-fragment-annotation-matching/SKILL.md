---
name: lipid-fragment-annotation-matching
description: Use when you have per-peak attribution scores from a deep learning lipidomics
  model and need to verify that top-attributed m/z values correspond to known, interpretable
  lipid fragments rather than spurious spectral features.
when_to_use_negative:
- Input is raw profile spectra rather than already-computed per-peak attributions
  — smoothing, baseline correction, peak picking, and attribution must happen first
- The goal is lipid identification itself (use LipiDetective prediction on mzML files)
  rather than post-hoc interpretation of existing predictions
- Spectra are from lipid classes outside the model's training distribution of phospholipid
  standard and reference measurements, where fragment annotations may not exist in
  Alex123
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: LipiDetective
  role: Pre-trained transformer model whose predicted lipid tokens and input peak
    attributions are matched to fragment annotations
  repo: https://github.com/LipiTUM/lipidetective
- name: Captum
  role: Library providing the integrated gradients method used to compute the per-peak
    attributions being matched
- name: Alex123
  role: Fragment database used to match peaks in the MS2 spectra to known fragments
- name: PyTorch
  role: Framework in which the LipiDetective transformer is implemented and run in
    inference mode during the interpretability analysis
- name: h5py
  role: Reads the preprocessed HDF5 test spectra fed to the model for attribution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:daf1aa17b545bcdaf7f6220978d1b702d9046547828985882859d4e51e4bd66f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - Alex123
  - Captum
  - LipiDetective
  - PyTorch
  - h5py
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-fragment-annotation-matching

## Summary

Match per-peak model attributions (e.g., integrated gradients values from LipiDetective) against known lipid fragment annotations in the Alex123 database to determine whether the model's predictions are driven by humanly interpretable fragment peaks.

## When to use

Use when you have per-peak attribution scores from a deep learning lipidomics model and need to verify that top-attributed m/z values correspond to known, interpretable lipid fragments rather than spurious spectral features.

## When NOT to use

- Input is raw profile spectra rather than already-computed per-peak attributions — smoothing, baseline correction, peak picking, and attribution must happen first
- The goal is lipid identification itself (use LipiDetective prediction on mzML files) rather than post-hoc interpretation of existing predictions
- Spectra are from lipid classes outside the model's training distribution of phospholipid standard and reference measurements, where fragment annotations may not exist in Alex123

## Inputs

- Per-peak integrated gradients attribution table keyed by m/z and predicted lipid token (from Captum analysis of LipiDetective)
- Alex123 fragment database for matching MS2 peaks to known lipid fragments
- LMSD query results providing the exact mass of the identified lipid
- Preprocessed MS2 spectra in HDF5 format (m/z range 50-1600, base peak normalized)

## Outputs

- peak_attributions.csv with columns for m/z, predicted token, and attribution score
- List of top-attributed peaks flagged as matching known Alex123 fragments vs. unannotated (e.g., 224.0 m/z)

## How to apply

After computing integrated gradients attributions with Captum over input peak embeddings for each predicted lipid token, aggregate the attributions into a per-peak table keyed by m/z and predicted token. Query the Alex123 database to match peaks in the MS2 spectra to known fragments — e.g., the 255.2 m/z peak corresponds to the known 16:0 fatty acid fragment driving prediction of 16:0 as the first fatty acid of PC 16:0_18:1. Compare top-attributed peaks against the Alex123 fragment list and LMSD-derived exact masses, treating agreement as qualitative confirmation of interpretability (no strict numeric threshold is required). Note peaks such as 224.0 m/z that are relevant for prediction but lack any annotation in Alex123 — report these as unannotated rather than discarding them. Save the result as peak_attributions.csv with columns for m/z, predicted token, and attribution score.

## Related tools

- **LipiDetective** (Pre-trained transformer model whose predicted lipid tokens and input peak attributions are matched to fragment annotations) — https://github.com/LipiTUM/lipidetective
- **Captum** (Library providing the integrated gradients method used to compute the per-peak attributions being matched)
- **Alex123** (Fragment database used to match peaks in the MS2 spectra to known fragments)
- **PyTorch** (Framework in which the LipiDetective transformer is implemented and run in inference mode during the interpretability analysis)
- **h5py** (Reads the preprocessed HDF5 test spectra fed to the model for attribution)

## Examples

```
ig = IntegratedGradients(model); attributions = ig.attribute(inputs=peak_embeddings, target=token_id); then join peak m/z values against the Alex123 fragment list and write peak_attributions.csv
```

## Evaluation signals

- The highest-attribution peaks match known lipid fragment m/z values reported as humanly interpretable in the paper's integrated gradients analysis (qualitative consistency, no exact numeric threshold required)
- The known 255.2 m/z peak (16:0 fatty acid fragment) is recovered as a key driver for predicting 16:0 as the first fatty acid in the PC 16:0_18:1 example
- peak_attributions.csv contains the required columns (m/z, predicted token, attribution score) and one row per peak-token pair
- Relevant but unannotated peaks (e.g., 224.0 m/z) are retained and flagged rather than silently dropped

## Limitations

- Some peaks relevant for prediction (e.g., 224.0 m/z) have no annotation in the Alex123 database, so fragment matching cannot fully explain all model attention
- Validation is qualitative — the paper sets no exact numeric threshold beyond consistency with reported interpretability results
- Interpretability analysis requires model weights (.pth), preprocessed HDF5 spectra, and attention/embedding extraction, which may not transfer to models without exposed embeddings

## Evidence

- [methods] Alex123 fragment matching: "the Alex123 database [19] was used to match peaks in the MS2 spectra to known fragments"
- [methods] integrated gradients via Captum: "the integrated gradients [30] method was employed as provided by the Captum [31] library"
- [abstract] humanly interpretable peaks finding: "Analysis of the integrated gradients reveals that LipiDetective focuses on relevant peaks that can be matched to known fragments and are thus humanly interpretable"
- [results] unannotated peaks caveat: [evidence span withheld — anchor status: not_found]
- [methods] LMSD exact mass query: "LMSD was queried to retrieve the exact mass of the identified lipid"

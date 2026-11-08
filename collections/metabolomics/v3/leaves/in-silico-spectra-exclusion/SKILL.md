---
name: in-silico-spectra-exclusion
description: Use when compiling a training set from public libraries (MoNA, GNPS,
  LipidMaps) while a separate annotation reference (NIST 20, LipidBlast) will score
  the evaluation set, to prevent overlap.
when_to_use_negative:
- When the training and evaluation annotation libraries are disjoint (no shared compounds),
  the exclusion step has nothing to filter and is unnecessary
- When the input MSP blocks are already part of the evaluation set (e.g. the experimental
  MS/MS spectra being annotated), removing LipidBlast/NIST 20 coverage would delete
  the data you intend to predict on
- When the goal is in-silico model benchmarking against a single unified library rather
  than the case-study train/evaluate split described here
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IDSL_MINT
  role: Downstream framework that consumes the curated (filtered) training set; the
    exclusion step ensures the IDSL_MINT transformer model is trained only on compounds
    absent from the NIST 20/LipidBlast evaluation annotation libraries
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:b3c8a30ca8e1fe1d0ec6081c6282a5e773bc80abd623ed41474631a5b123d80f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# in-silico-spectra-exclusion

## Summary

Filters a curated training set by removing MSP blocks whose compounds are already covered in independent reference libraries (here LipidBlast in-silico and NIST 20) so that evaluation-set annotations cannot leak into model training.

## When to use

Use when compiling a training set from public libraries (MoNA, GNPS, LipidMaps) while a separate annotation reference (NIST 20, LipidBlast) will score the evaluation set, to prevent overlap.

## When NOT to use

- When the training and evaluation annotation libraries are disjoint (no shared compounds), the exclusion step has nothing to filter and is unnecessary
- When the input MSP blocks are already part of the evaluation set (e.g. the experimental MS/MS spectra being annotated), removing LipidBlast/NIST 20 coverage would delete the data you intend to predict on
- When the goal is in-silico model benchmarking against a single unified library rather than the case-study train/evaluate split described here

## Inputs

- MSP file with spectral blocks from a public library (e.g. lipidmaps_msp.zip from Zenodo entry [27], file#2)
- Compound coverage lists for the annotation libraries (NIST 20 and LipidBlast compounds used for the evaluation set, file#1 csa_spreadsheet.zip)

## Outputs

- Curated training set of experimentally measured MSP blocks (positive and negative modes kept separate)
- Spectrum counts per lipid class used to apply the ≥50-spectra retention threshold

## How to apply

Parse each .msp spectral block from the downloaded training library (e.g. lipidmaps_msp.zip from Zenodo entry [27]) and first drop blocks flagged as in-silico LipidBlast predictions so only experimentally measured spectra remain. Then cross-reference the compound identifiers of the remaining blocks against the compounds covered in the annotation libraries used for the evaluation set — in the ST002044 case study, NIST 20 and LipidBlast — and exclude any block whose compound appears in either library, because those libraries are used to annotate the experimental MS/MS spectra and their compounds must not appear in the training data. Keep positive and negative ion-mode blocks separated throughout the filter. Afterwards count spectra per lipid class and retain only classes with at least 50 spectra (6 of 8 lipid classes passed). The rationale is to ensure a high validity of the resulting model, since the same compounds would otherwise be annotated by library search in the evaluation set while being memorized during training.

## Related tools

- **IDSL_MINT** (Downstream framework that consumes the curated (filtered) training set; the exclusion step ensures the IDSL_MINT transformer model is trained only on compounds absent from the NIST 20/LipidBlast evaluation annotation libraries) — https://github.com/idslme/IDSL_MINT

## Examples

```
python -c "from IDSL_MINT import ms2fp; ms2fp.train("ms2fp_yaml.yaml")"  # after excluding LipidBlast/NIST20-covered blocks from lipidmaps_curated_training.msp
```

## Evaluation signals

- Curated training set should cover 6.96% of all LipidMaps compounds in positive mode
- Curated training set should cover 4.58% of all LipidMaps compounds in negative mode
- Exactly 6 of 8 lipid classes should pass the ≥50-spectra threshold after filtering
- No remaining training MSP block should have a compound identifier present in the NIST 20 or LipidBlast compound lists
- Positive and negative ion-mode blocks must remain separated after the exclusion step

## Limitations

- The filter is only valid for the case-study split where NIST 20 and LipidBlast are the annotation libraries; using different evaluation libraries requires re-deriving the exclusion list
- Excluding classes with fewer than 50 spectra leaves only 6 well-covered lipid classes, reducing class diversity of the trained model
- MSP blocks must include PrecursorMZ row entries and unique headers for IDSL_MINT parsing; blocks missing these fields will fail downstream

## Evidence

- [results] Exclude spectra of compounds covered in LipidBlast and NIST 20 libraries: "We excluded the spectra related to compounds that were covered in the LipidBlast (in-silico) [23] and NIST 20 libraries because we used these two libraries for annotating the experimental MS/MS"
- [results] Case study training set coverage percentages: "Our case study training set included 6.96% and 4.58% of all the compounds in the LipidMaps database in positive and negative modes, respectively"
- [results] Lipid class retention threshold: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [readme] MSP parsing requirements for IDSL_MINT: "MSP blocks must include `PrecursorMZ: ` row entries."
- [results] Train/evaluate separation rationale: "Next, we carefully compiled the training dataset for the case study (ST002044) to ensure a high validity of the resulting model (Fig. 3)."

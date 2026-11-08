---
name: loocv-result-aggregation
description: Use when validating a known-to-unknown metabolite identification method
  by LOOCV plus an external test set, specifically to determine whether identification
  is valid when a structure-related metabolite exists in the database.
when_to_use_negative:
- The input unknown has a direct spectral match in an MS/MS database (e.g., an exact
  spectrum entry), rather than requiring structural-similarity-based inference through
  a related metabolite
- The goal is de novo identification of compounds with no structure-related metabolite
  in any reference database — DeepMASS is explicitly reported valid only when a structure-related
  metabolite is available
- Spectra are not comparable to the training instrument conditions; the bundled DeepMASS
  model was trained for 40V spectra, so a custom model must be trained (via test.py
  on data/spectra) before aggregation is meaningful
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DeepMASS
  role: Deep-learning model that scores structural similarity between unknown metabolites
    and known ones from MS/MS spectra, plus a rank method for picking candidate structures;
    used to generate LOOCV and external-test identifications that are then aggregated
  repo: https://github.com/hcji/DeepMASS
- name: KEGG
  role: Reference database supplying the 662 compounds whose MS/MS spectra form the
    LOOCV evaluation set and the pool of structure-related metabolites
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument used to measure the external-test male-infertility biomarker spectra
    aggregated in the validation table
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument used to measure the external-test male-infertility biomarker spectra
    aggregated in the validation table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:f3093532dd636179188699806995e26aec55589130af8d0698d7082036a28550
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# loocv-result-aggregation

## Summary

Aggregates leave-one-out cross-validation (LOOCV) identification results from DeepMASS on 662 KEGG compound MS/MS spectra together with external-test results for male-infertility biomarkers, and tabulates identification validity against the presence of a structure-related metabolite in the reference database.

## When to use

Use when validating a known-to-unknown metabolite identification method by LOOCV plus an external test set, specifically to determine whether identification is valid when a structure-related metabolite exists in the database.

## When NOT to use

- The input unknown has a direct spectral match in an MS/MS database (e.g., an exact spectrum entry), rather than requiring structural-similarity-based inference through a related metabolite
- The goal is de novo identification of compounds with no structure-related metabolite in any reference database — DeepMASS is explicitly reported valid only when a structure-related metabolite is available
- Spectra are not comparable to the training instrument conditions; the bundled DeepMASS model was trained for 40V spectra, so a custom model must be trained (via test.py on data/spectra) before aggregation is meaningful

## Inputs

- MS/MS spectra of 662 KEGG compounds (LOOCV evaluation set)
- External-test MS/MS spectra of male-infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF
- Reference database of known metabolites (KEGG) annotated with structural relationships
- Spectra files placed in the DeepMASS data/spectra directory

## Outputs

- Per-case LOOCV identification results for the 662 KEGG compounds
- Per-case external-test identification results for the male-infertility biomarkers
- Contingency table of identification validity (valid/invalid) vs structure-related metabolite presence (present/absent)
- Aggregate statement of when identification is valid (valid iff a structure-related metabolite is in the database)

## How to apply

Run DeepMASS (github.com/hcji/DeepMASS) in leave-one-out cross-validation mode over the MS/MS spectra of the 662 KEGG compounds, so each compound is identified against a database excluding itself. Separately run the trained model on external-test spectra of male-infertility biomarkers acquired on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF instruments. For each LOOCV case and each external-test case, check whether a structure-related metabolite of the unknown is present in the reference database, then build a contingency table crossing identification validity (valid/invalid) with structure-related metabolite presence (present/absent). The correct outcome is that 'present & valid' and 'absent & invalid' cells dominate, matching the paper's finding that identification of an unknown compound is valid if its structure-related metabolite is available in the database.

## Related tools

- **DeepMASS** (Deep-learning model that scores structural similarity between unknown metabolites and known ones from MS/MS spectra, plus a rank method for picking candidate structures; used to generate LOOCV and external-test identifications that are then aggregated) — https://github.com/hcji/DeepMASS
- **KEGG** (Reference database supplying the 662 compounds whose MS/MS spectra form the LOOCV evaluation set and the pool of structure-related metabolites)
- **Shimadzu LC-ESI-IT-TOF** (Instrument used to measure the external-test male-infertility biomarker spectra aggregated in the validation table)
- **Bruker Compact LC-ESI-QTOF** (Instrument used to measure the external-test male-infertility biomarker spectra aggregated in the validation table)

## Examples

```
python example.py
```

## Evaluation signals

- Contingency table cells 'present & valid' and 'absent & invalid' dominate, reproducing the paper's reported finding that identification is valid when a structure-related metabolite is available
- LOOCV is a true leave-one-out: each of the 662 KEGG compounds is identified against a database from which it is excluded (no self-match leakage)
- External-test identifications come only from spectra acquired on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF instruments, kept disjoint from the LOOCV training compounds
- Every identification case is classified on both axes (validity and structure-related-metabolite presence) — no unclassified cases remain in the tabulation
- Aggregated results are consistent with the DeepMASS claim that structural-similarity scoring extends the identification range of existing MS/MS databases

## Limitations

- Identification of an unknown compound is only valid if a structure-related metabolite is available in the database; unknowns without a related database entry cannot be reliably identified
- The experimental spectra (MetDNA dataset) were removed from the repository at the owner's request, so the package cannot be run directly; users must train their own model on an in-house database or wait for public-database spectra
- The pre-trained model downloadable from the README is specific to 40V spectra (model/40V directory); other collision energies require retraining
- The discussion section text is absent from the source document, so detailed caveats from the discussion could not be extracted

## Evidence

- [readme] DeepMASS is a known-to-unknown metabolite identification workflow, which includes a deep-learning based model to predict structural similarity: [evidence span withheld — anchor status: external_doc]
- [readme] Since the experimental spectra has been removed, this package cannot be run directly. You can train your own model with your in-house database.: [evidence span withheld — anchor status: external_doc]

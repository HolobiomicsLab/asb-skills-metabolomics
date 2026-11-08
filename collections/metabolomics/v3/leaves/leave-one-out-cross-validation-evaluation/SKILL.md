---
name: leave-one-out-cross-validation-evaluation
description: Use when a database-backed MS/MS identification method needs unbiased
  performance estimates on a closed compound set — e.g. 662 KEGG spectra scored by
  deep neural network structural similarity.
when_to_use_negative:
- The query compounds are external, unlabeled biomarkers (e.g. the male infertility
  LC-ESI-IT-TOF/QTOF biomarkers) rather than a closed database set — that case is
  an external test, not LOOCV evaluation.
- No structure-related metabolite for a query exists in the database, since the paper
  states identification is only valid when one is available.
- The DeepMASS model has not been trained or downloaded (the removed MetDNA dataset
  means the package cannot be run directly out of the box).
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DeepMASS
  role: Provides the deep-learning model and scoring workflow whose structural-similarity
    prediction is evaluated under leave-one-out cross-validation on the 662 KEGG spectra.
  repo: https://github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that generated the experimental MS/MS spectra in the original evaluation
    set.
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument that generated experimental MS/MS spectra alongside the Shimadzu
    LC-ESI-IT-TOF data.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:68aa75c390a5bf52d69d0d608080b2ee88362ac69931a44049016ceea39ce906
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# leave-one-out-cross-validation-evaluation

## Summary

Evaluates a metabolite identification model by leave-one-out cross-validation on MS/MS spectra of 662 KEGG compounds, scoring each query compound against the remaining database entries with the DeepMASS structural-similarity model.

## When to use

Use when a database-backed MS/MS identification method needs unbiased performance estimates on a closed compound set — e.g. 662 KEGG spectra scored by deep neural network structural similarity.

## When NOT to use

- The query compounds are external, unlabeled biomarkers (e.g. the male infertility LC-ESI-IT-TOF/QTOF biomarkers) rather than a closed database set — that case is an external test, not LOOCV evaluation.
- No structure-related metabolite for a query exists in the database, since the paper states identification is only valid when one is available.
- The DeepMASS model has not been trained or downloaded (the removed MetDNA dataset means the package cannot be run directly out of the box).

## Inputs

- MS/MS spectra of 662 KEGG compounds (the DeepMASS evaluation set)
- DeepMASS trained model (e.g. model-40V.zip placed in /model/40V)
- In-house or public-spectra database if retraining the model

## Outputs

- Per-compound table of top-ranked structure-related metabolite and DeepMASS similarity score
- Overall identification accuracy across the 662 KEGG compounds

## How to apply

Load the MS/MS spectra of the 662 KEGG compounds used by DeepMASS and run the deep neural network structural-similarity scoring in leave-one-out mode: for each query compound, exclude its own entry from the database and score it against the remaining 661 spectra. Record the top-ranked structure-related metabolite and its similarity score in a per-compound result table. Aggregate per-compound outcomes into an overall identification accuracy for the 662-compound evaluation. The rationale is that structurally related metabolites have related MS/MS spectra, so a valid identification is one whose structure-related metabolite is available in the database — the LOOCV setting directly tests whether the model can recover that relationship without the query compound being in the training set. Note that the experimental MetDNA spectra used in the original evaluation have been removed from the repository, so reproduction requires either private access to that dataset or training a replacement model on an in-house or public-spectra database.

## Related tools

- **DeepMASS** (Provides the deep-learning model and scoring workflow whose structural-similarity prediction is evaluated under leave-one-out cross-validation on the 662 KEGG spectra.) — https://github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument that generated the experimental MS/MS spectra in the original evaluation set.)
- **Bruker Compact LC-ESI-QTOF** (Instrument that generated experimental MS/MS spectra alongside the Shimadzu LC-ESI-IT-TOF data.)

## Examples

```
python example.py  # spectra of the 662 KEGG compounds in data/spectra; model-40V.zip unzipped into /model/40V
```

## Evaluation signals

- Every one of the 662 query compounds yields a per-compound entry with a top-ranked structure-related metabolite and a similarity score.
- No query compound is scored against a database that still contains its own spectrum (true leave-one-out behavior).
- The aggregated accuracy is computed over the full 662-compound set, matching the article's stated evaluation scope.
- Reproduced top-1 rankings should be consistent with the claim that identification is valid only when a structure-related metabolite is present in the database.
- Model artefacts match the README layout: model zip extracted under /model/40V and spectra placed in data/spectra.

## Limitations

- The MetDNA experimental spectra dataset was removed at the owner's request (2019/04/29), so the package cannot be run directly and the original LOOCV results can only be reproduced by those who already possess the dataset, keeping it private.
- If the original dataset is unavailable, a replacement model must be trained on an in-house database or public spectra, which changes the basis of the evaluation.
- Identification validity is contingent on a structure-related metabolite existing in the database; queries lacking such relatives will not be identified.

## Evidence

- [readme] Since the experimental spectra has been removed, this package cannot be run directly.: [evidence span withheld — anchor status: external_doc]
- [readme] if you want to train a model based on your in-house database, please put your spectra files in to data/spectra directory and run test.py: [evidence span withheld — anchor status: not_found]

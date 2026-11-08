---
name: external-dataset-model-validation
description: Use when a structural-similarity scoring model trained on one MS/MS dataset
  (e.g. MetDNA/DIONysus 40V spectra of KEGG compounds) must be evaluated on independent
  spectra measured on different instruments (e.g. Shimadzu LC-ESI-IT-TOF or Bruker
  Compact LC-ESI-QTOF) to test generalization.
when_to_use_negative:
- Do not use when the question is internal model fit on the training set itself —
  that is the leave-one-out cross-validation evaluation on the 662 KEGG compounds,
  not an external generalization test.
- Do not use to claim identification of an unknown whose structure-related metabolite
  is absent from the reference database — the paper explicitly states identification
  is valid only when that metabolite is available.
- Do not use as a de novo structural elucidation method; DeepMASS only ranks known
  database structures by predicted similarity.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DeepMASS
  role: Deep-neural-network scorer used to predict structural similarity between external
    unknown metabolite MS/MS spectra and the 662-compound KEGG database; inference
    produces the ranked candidate list used for validation.
  repo: github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument used to acquire the external-test MS/MS biomarker spectra (cond_external_shimadzu)
    that the trained model is validated on.
- name: Bruker Compact LC-ESI-QTOF
  role: Second external instrument platform used to acquire held-out biomarker spectra
    for the external generalization test.
- name: KEGG
  role: Reference MS/MS spectral database of 662 compounds defining the fixed search
    space against which external spectra are scored.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:8195d953731deaaacf8fb9a5d8e212b65d4d9a18e3a6298d794bda4fd580103c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# external-dataset-model-validation

## Summary

Validates a trained DeepMS/MS structural-similarity scorer on spectra acquired on instruments and sample cohorts disjoint from the training data, by ranking KEGG database metabolites for each held-out MS/MS spectrum and checking recovery of the structure-related (true) metabolite.

## When to use

Use when a structural-similarity scoring model trained on one MS/MS dataset (e.g. MetDNA/DIONysus 40V spectra of KEGG compounds) must be evaluated on independent spectra measured on different instruments (e.g. Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) to test generalization.

## When NOT to use

- Do not use when the question is internal model fit on the training set itself — that is the leave-one-out cross-validation evaluation on the 662 KEGG compounds, not an external generalization test.
- Do not use to claim identification of an unknown whose structure-related metabolite is absent from the reference database — the paper explicitly states identification is valid only when that metabolite is available.
- Do not use as a de novo structural elucidation method; DeepMASS only ranks known database structures by predicted similarity.

## Inputs

- Trained DeepMASS model checkpoint (model-40V, downloaded and unzipped into /model/40V)
- External-test MS/MS spectra files (.mgf or similar spectral files) placed in data/spectra, e.g. cond_external_shimadzu biomarker spectra from Shimadzu LC-ESI-IT-TOF / Bruker Compact LC-ESI-QTOF
- KEGG reference MS/MS spectral library for the 662-compound database

## Outputs

- Per-biomarker ranked list of database metabolites scored by DeepMASS structural-similarity score
- Identification-success labels (top-hit recovery of the structure-related metabolite) per external spectrum, assembled e.g. as external_shimadzu_identification.csv
- External identification performance metric (metric_external_shimadzu_identification)

## How to apply

Take the frozen DeepMASS deep-neural-network scorer (the model-40V trained on MS/MS spectra of 662 KEGG compounds) and apply it to external-test spectra that were never used in training — here the biomarker spectra from the male infertility study acquired on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF. Load the KEGG reference library of MS/MS spectra for the 662 compounds as the fixed database search space; for each external spectrum run DeepMASS inference to score structural similarity of the unknown against every database metabolite. Rank database metabolites by DeepMASS score per spectrum and label whether the structure-related (true) metabolite is the top hit — an identification counts as valid only if its structure-related metabolite is available in and recovered from the database. Assemble per-biomarker ranked hits and identification-success labels (e.g. external_shimadzu_identification.csv) and compare against the reported external identification metric. Rationale: the paper shows identification of unknown compounds is valid when the structure-related metabolite is in the database, so external success is judged strictly on that recovery condition rather than on any retraining or threshold tuning.

## Related tools

- **DeepMASS** (Deep-neural-network scorer used to predict structural similarity between external unknown metabolite MS/MS spectra and the 662-compound KEGG database; inference produces the ranked candidate list used for validation.) — github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument used to acquire the external-test MS/MS biomarker spectra (cond_external_shimadzu) that the trained model is validated on.)
- **Bruker Compact LC-ESI-QTOF** (Second external instrument platform used to acquire held-out biomarker spectra for the external generalization test.)
- **KEGG** (Reference MS/MS spectral database of 662 compounds defining the fixed search space against which external spectra are scored.)

## Examples

```
python example.py
```

## Evaluation signals

- Every external biomarker spectrum yields a ranked database hit list; the structure-related (true) metabolite is explicitly flagged as present or absent in the KEGG 662-compound library.
- Identification is counted valid only when the structure-related metabolite is available in the database AND recovered as a top hit; success rate computed only over that valid subset.
- Output table (e.g. external_shimadzu_identification.csv) matches the reported external identification metric (metric_external_shimadzu_identification) for the Shimadzu LC-ESI-IT-TOF biomarkers.
- No retraining or parameter re-fitting occurs between training and external evaluation — the same model-40V checkpoint is applied unchanged.
- Scores are reproducible across reruns: running the same external spectra through the same model and database produces the same ranking.

## Limitations

- Identification of an unknown compound is only valid if its structure-related metabolite is available in the database; performance degrades or is undefined outside that condition.
- The original experimental spectra dataset (MetDNA dataset) was removed at the owner's request, so the package cannot be run directly out of the box; users must either train their own model on in-house spectra or wait for the author to include public-database spectra.
- External validity is demonstrated on male-infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF; generalization to other instrument types or cohorts is not established by this test alone.
- Scoring depends on MS/MS spectra acquired at consistent collision energy with the model (40V model checkpoint); spectra at different energies may not be directly comparable.

## Evidence

- [abstract] identification of unknown compound is valid if its structure-related metabolite is available in the database: "Results show that the identification of unknown compound is valid if its structure-related metabolite is available in the database"
- [abstract] external test on the biomarkers from male infertility study: "an external test on the biomarkers from male infertility study"
- [readme] Download the model here, then unzip it into /model/40V directory: [evidence span withheld — anchor status: external_doc]
- [readme] Clone the repo and put spectra files in data/spectra directory: [evidence span withheld — anchor status: external_doc]
- [readme] the dataset can be only used to reproduce the results of DeepMASS paper: [evidence span withheld — anchor status: external_doc]

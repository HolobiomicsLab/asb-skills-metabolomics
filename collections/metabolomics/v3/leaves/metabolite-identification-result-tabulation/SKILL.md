---
name: metabolite-identification-result-tabulation
description: Use when DeepMASS structural-similarity scoring has produced ranked candidate
  metabolites for MS/MS spectra (e.g. 662 KEGG compounds or male-infertility biomarkers)
  and you need per-compound result tables plus aggregate validity counts.
when_to_use_negative:
- When only raw MS/MS spectra are available and DeepMASS structural-similarity scoring
  has not yet produced ranked candidates — the tabulation step assumes scored results
  already exist.
- When the goal is to train a new DeepMASS model from an in-house database rather
  than evaluate identification outcomes of an already-run model.
- When you need spectral-level metrics (e.g. cosine or dot-product spectral similarity)
  instead of compound-level identification validity counts.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: DeepMASS
  role: Produces the ranked structural-similarity scores whose identification outcomes
    are tabulated and aggregated into validity counts.
  repo: https://github.com/hcji/DeepMASS
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:7abfb80b25b09a2f7273dcd80b715492ba801b65b3a7df5a56eeb6c5006f0fc6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolite-identification-result-tabulation

## Summary

Tabulate DeepMASS identification outcomes (top-ranked structure-related metabolite and structural-similarity score) per query compound from leave-one-out and external-test evaluations, then aggregate into validity counts to test whether identification is valid when a structure-related database metabolite exists.

## When to use

Use when DeepMASS structural-similarity scoring has produced ranked candidate metabolites for MS/MS spectra (e.g. 662 KEGG compounds or male-infertility biomarkers) and you need per-compound result tables plus aggregate validity counts.

## When NOT to use

- When only raw MS/MS spectra are available and DeepMASS structural-similarity scoring has not yet produced ranked candidates — the tabulation step assumes scored results already exist.
- When the goal is to train a new DeepMASS model from an in-house database rather than evaluate identification outcomes of an already-run model.
- When you need spectral-level metrics (e.g. cosine or dot-product spectral similarity) instead of compound-level identification validity counts.

## Inputs

- Per-compound DeepMASS identification results from leave-one-out cross-validation on MS/MS spectra of 662 KEGG compounds (top-ranked structure-related metabolite, similarity score)
- External-test identification results for male-infertility biomarker spectra from Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF
- Reference-database annotation of structure-related metabolites for each query compound (KEGG-derived)

## Outputs

- Per-compound result table (query compound, top-ranked structure-related metabolite, DeepMASS similarity score, database-structure-related presence, valid/invalid)
- Contingency table of identification validity vs structure-related database metabolite presence
- Aggregated overall identification accuracy across the 662-compound LOOCV evaluation

## How to apply

Load the per-query output of the DeepMASS neural-network structural-similarity scoring, recording for each query compound the top-ranked structure-related metabolite and its similarity score. For the leave-one-out cross-validation setting, each query spectrum from the 662 KEGG compounds is scored against the remaining 661 database spectra; for the external test, repeat the tabulation for male-infertility biomarker spectra measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF. For every case, annotate whether a structure-related metabolite is present in the reference database (present/absent) and whether identification is judged valid (valid/invalid). Build a contingency table crossing these two annotations, then aggregate per-compound outcomes into an overall identification accuracy across the 662-compound evaluation. The tabulation is the evidence base for the paper's finding that identification is valid if its structure-related metabolite is available in the database.

## Related tools

- **DeepMASS** (Produces the ranked structural-similarity scores whose identification outcomes are tabulated and aggregated into validity counts.) — https://github.com/hcji/DeepMASS

## Evaluation signals

- Every one of the 662 KEGG query compounds appears exactly once in the per-compound result table (LOOCV setting).
- Each row carries a top-ranked structure-related metabolite and a DeepMASS similarity score, not a missing/null candidate.
- The contingency table cells 'present & valid' and 'absent & invalid' together dominate the counts, matching the finding that identification is valid if its structure-related metabolite is available in the database.
- The aggregated overall identification accuracy covers the full 662-compound set with no dropped or double-counted queries.
- External-test rows for male-infertility biomarkers (Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF) are tabulated with the same schema as the LOOCV rows.

## Limitations

- The experimental spectra dataset underlying the results was removed from the repository at the owner's (MetDNA dataset) request, so the tabulation cannot be reproduced directly without obtaining the spectra from the owner.
- The README states the package 'cannot be run directly' since the experimental spectra was removed; the user must train their own model on an in-house database or wait for public database spectra to be included.
- The validity criterion is indirect: DeepMASS extends the identification range by predicting structural similarity, so the tabulation tests structure-related-metabolite availability rather than direct spectral-library matching.

## Evidence

- [readme] DeepMASS scoring basis: [evidence span withheld — anchor status: external_doc]

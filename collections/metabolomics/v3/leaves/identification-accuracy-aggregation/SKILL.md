---
name: identification-accuracy-aggregation
description: Use when you have per-compound top-ranked candidate lists and similarity
  scores from a leave-one-out or database-search run and need to report overall identification
  accuracy on a labeled compound set.
when_to_use_negative:
- The goal is de novo structure elucidation without any database — this aggregation
  presumes a database search where a structure-related metabolite may or may not be
  present.
- The spectra are not MS/MS spectra paired with known structures (e.g., unlabeled
  clinical samples like the male infertility biomarkers used only for external testing),
  since accuracy cannot be computed without ground-truth compound labels.
- The candidate ranking comes from a method other than structural-similarity scoring
  (e.g., direct spectral match), which the paper's 'structure-related metabolite counts
  as correct' criterion was not designed to evaluate.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepMASS
  role: Deep-learning model scoring structural similarity between unknown and known
    metabolites from MS/MS spectra, with a rank method picking candidate structures;
    its LOOCV scoring produces the per-compound results aggregated here
  repo: https://github.com/hcji/DeepMASS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:2f11c6a68c1c3350773a4585b22e9bc7e5af3c76cbfbf8db5027d92e84c023a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - DeepMASS
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# identification-accuracy-aggregation

## Summary

Aggregates per-compound leave-one-out identification outcomes from a structural-similarity scoring run into an overall identification accuracy, as done for DeepMASS on 662 KEGG compounds. It quantifies whether unknown metabolites can be identified when a structure-related metabolite exists in the database.

## When to use

Use when you have per-compound top-ranked candidate lists and similarity scores from a leave-one-out or database-search run and need to report overall identification accuracy on a labeled compound set.

## When NOT to use

- The goal is de novo structure elucidation without any database — this aggregation presumes a database search where a structure-related metabolite may or may not be present.
- The spectra are not MS/MS spectra paired with known structures (e.g., unlabeled clinical samples like the male infertility biomarkers used only for external testing), since accuracy cannot be computed without ground-truth compound labels.
- The candidate ranking comes from a method other than structural-similarity scoring (e.g., direct spectral match), which the paper's 'structure-related metabolite counts as correct' criterion was not designed to evaluate.

## Inputs

- MS/MS spectra of the 662 KEGG compounds used by DeepMASS (spectra files placed in data/spectra)
- Trained DeepMASS structural-similarity model (model/40V) and the in-house/database metabolite structures

## Outputs

- Per-compound result table with query compound, top-ranked structure-related metabolite, and DeepMASS similarity score
- Overall leave-one-out identification accuracy aggregated across the 662-compound evaluation

## How to apply

Load the MS/MS spectra of the 662 KEGG compounds used by DeepMASS, then run the deep neural network structural-similarity scoring in leave-one-out mode so each query compound is scored only against the remaining 661 database spectra, excluding its own spectrum. For each query compound, record the top-ranked structure-related metabolite and the DeepMASS similarity score in a per-compound result table. Score a compound as correctly identified if its top-ranked hit is a structure-related metabolite, reflecting the paper's finding that identification is valid when a structure-related metabolite is available in the database. Finally, aggregate the per-compound outcomes (hits over total queries) into a single identification accuracy figure for the 662-compound evaluation.

## Related tools

- **DeepMASS** (Deep-learning model scoring structural similarity between unknown and known metabolites from MS/MS spectra, with a rank method picking candidate structures; its LOOCV scoring produces the per-compound results aggregated here) — https://github.com/hcji/DeepMASS

## Examples

```
python example.py  # rank candidates for each of the 662 KEGG query spectra, then aggregate top-hit outcomes into overall LOOCV identification accuracy
```

## Evaluation signals

- Each of the 662 query compounds is scored against exactly 661 database spectra (self excluded) — verify no self-match appears in the top-ranked hits.
- The per-compound result table contains a top-ranked metabolite and a DeepMASS similarity score for every query compound (no missing rows).
- Each hit can be classified as a structure-related metabolite or not; the accuracy is the fraction of queries whose top hit is structure-related.
- The aggregate accuracy should be consistent with the paper's claim that identification is valid when a structure-related metabolite is available in the database.

## Limitations

- The experimental spectra dataset (MetDNA dataset) was removed from the repository at the owner's request, so the package cannot be run directly; you must train your own model with an in-house database or use public database spectra.
- The model was trained with a 40V collision-energy model file only; spectra at other collision energies may not be handled.
- The evaluation criterion counts a structure-related metabolite as a valid identification, which is a similarity-based rather than exact-match standard.

## Evidence

- [readme] README: spectra placement and training workflow: [evidence span withheld — anchor status: not_found]
- [readme] README: dataset removal limitation: [evidence span withheld — anchor status: external_doc]

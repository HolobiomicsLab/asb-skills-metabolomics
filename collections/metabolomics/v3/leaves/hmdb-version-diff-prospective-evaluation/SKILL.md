---
name: hmdb-version-diff-prospective-evaluation
description: Use when a structure-generating model predates a database version bump
  (e.g., HMDB v4→v5) and you want to test whether its sampling frequency ranks newly
  added, previously unknown metabolites above other generated molecules.
when_to_use_negative:
- The model was trained on the same database version you are evaluating against —
  any 'prospective' gain would be circular; only metabolites newly added after training
  (HMDB v4→v5 diff) qualify.
- You lack the model's sampling frequencies or generated-structure database — the
  ROC and enrichment analysis cannot be computed from reference lists alone.
- You need to evaluate metabolites from evolutionarily distant organisms (e.g., plant
  or bacterial metabolism), where the article notes a bespoke model trained on human
  metabolite structures is likely required.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: DeepMet (clm)
  role: Chemical language model whose 34.4M generated structures and sampling frequencies
    form the ranked candidate table being evaluated
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Canonicalize withheld test and HMDB v5.0 SMILES and compute SMILES/InChIKey
    matches against the ranked frequency table
- name: Snakemake
  role: Runs the end-end clm workflow (Snakefile) including the write_structural_prior_CV_overall
    evaluation logic being mirrored
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime for the evaluation script computing ROCs, hypergeometric test, and
    output files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:70f633f919e88fcd2336df0ab4d16eb296bc676c1aaa1153e9458003f2e9b565
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - DeepMet (clm)
  - Python
  - Python 3.10
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# HMDB version-diff prospective evaluation

## Summary

Evaluates whether a generative chemical language model's sampling frequencies prospectively anticipate metabolites discovered after model training, by diffing metabolites newly added in a later database release (HMDB v5.0) against the released set of generated structures and computing enrichment/ROC metrics.

## When to use

Use when a structure-generating model predates a database version bump (e.g., HMDB v4→v5) and you want to test whether its sampling frequency ranks newly added, previously unknown metabolites above other generated molecules.

## When NOT to use

- The model was trained on the same database version you are evaluating against — any 'prospective' gain would be circular; only metabolites newly added after training (HMDB v4→v5 diff) qualify.
- You lack the model's sampling frequencies or generated-structure database — the ROC and enrichment analysis cannot be computed from reference lists alone.
- You need to evaluate metabolites from evolutionarily distant organisms (e.g., plant or bacterial metabolism), where the article notes a bespoke model trained on human metabolite structures is likely required.

## Inputs

- DeepMet/clm released resource of 34.4M generated SMILES with per-structure sampling frequencies
- HMDB v5.0 list of 313 newly added metabolite SMILES (absent from the training-release HMDB)
- Withheld cross-validation test SMILES (positive control)
- RDKit for SMILES canonicalization and InChIKey computation

## Outputs

- evaluation_metrics.csv with CV AUC, prospective AUC, generated fraction, top-10,000 hit count, fold enrichment, and hypergeometric p-value
- roc_curves.png plotting both the cross-validation and prospective ROC curves
- Ranked frequency table with per-hit ranks for matched HMDB v5.0 additions

## How to apply

Load the released resource of 34.4M generated SMILES with their sampling frequencies and rank structures by decreasing frequency, mirroring the clm write_structural_prior_CV_overall evaluation logic. Canonicalize withheld cross-validation test SMILES with RDKit, match against the ranked table by SMILES/InChIKey comparison, and derive the CV ROC (target AUC 0.98) as a positive control. Then obtain the 313 SMILES of metabolites newly added in HMDB v5.0 (i.e., absent from the training version), determine the generated fraction by database matching (target 252/313, 81%), and compute a prospective ROC on those 252 hits (target AUC 0.97). Finally, count HMDB v5.0 additions falling within the top-10,000 most frequent structures (target 105/252) and compute enrichment over random expectation (~1,500-fold) with a hypergeometric test (target p = 4.3e-5). Write metrics to evaluation_metrics.csv and plot both ROC curves to roc_curves.png.

## Related tools

- **DeepMet (clm)** (Chemical language model whose 34.4M generated structures and sampling frequencies form the ranked candidate table being evaluated) — https://github.com/skinniderlab/CLM
- **RDKit** (Canonicalize withheld test and HMDB v5.0 SMILES and compute SMILES/InChIKey matches against the ranked frequency table)
- **Snakemake** (Runs the end-end clm workflow (Snakefile) including the write_structural_prior_CV_overall evaluation logic being mirrored) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime for the evaluation script computing ROCs, hypergeometric test, and output files)

## Examples

```
python evaluate_prior.py --frequencies deepmet_34.4M.csv --hmdb_v5_additions hmdb_v5_new_smiles.txt --top_k 10000 --metrics evaluation_metrics.csv --plot roc_curves.png
```

## Evaluation signals

- Cross-validation ROC AUC on withheld test metabolites equals ~0.98
- Generated fraction of the 313 HMDB v5.0 additions equals 252/313 (81%)
- Prospective ROC AUC on the 252 generated HMDB v5.0 additions equals ~0.97
- 105/252 HMDB v5.0 additions fall within the top-10,000 most frequent structures
- Hypergeometric enrichment over random expectation is ~1,500-fold with p = 4.3e-5

## Limitations

- The language model was trained and evaluated only on the structures of human metabolites; evolutionarily distant applications (e.g., plant or bacterial metabolism) will likely require bespoke models.
- The model has inherent limitations anticipating metabolites that originate from divergent, as-of-yet undiscovered biosynthetic routes, which could deflate prospective recovery on future database additions.
- Only 81% of HMDB v5.0 additions appear in the generated set at all — the remaining 61 are counted as misses regardless of rank.

## Evidence

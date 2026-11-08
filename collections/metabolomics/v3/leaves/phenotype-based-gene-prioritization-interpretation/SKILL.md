---
name: phenotype-based-gene-prioritization-interpretation
description: Use when a cohort of patients has both an Exomiser-filtered candidate
  gene list with hiPHIVE phenotype scores and an alternative (e.g., metabolomics-driven)
  per-gene score, and you need to compare causative-gene ranks per patient.
when_to_use_negative:
- The patient has no known/previously diagnosed causative gene — rank comparison against
  ground truth is not possible (the analysis needs a confirmed diagnosis, as in the
  11 TIDEX patients)
- The candidate list lacks Exomiser hiPHIVE phenotype scores (e.g., only variant-effect/frequency-filtered
  genes with no phenotype ranking), so the Exomiser-Phenotype rank cannot be computed
- Feature-to-metabolite mapping is unavailable for the metabolomic system (as for
  the TIDEX metabolomic platform), which limits interpreting the metabolomics-driven
  side of the comparison at the metabolite level
tools:
- name: metPropagate
  role: Supplies the metabolomics-driven per-gene propagated scores (Score.ID) whose
    causative-gene ranks are compared against Exomiser-Phenotype
  repo: https://github.com/emmagraham/metPropagate
- name: Exomiser (hiPHIVE phenotype algorithm)
  role: Provides the phenotype-driven candidate gene ranking (Exomiser-Phenotype score)
    used as the per-patient comparator
- name: DeepVariant
  role: Upstream producer of the singleton VCFs that Exomiser filters into the candidate
    gene lists being ranked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:883e0c6591278a58d83dca503e31f411ec8c8a2b9d36afc7da5bc8c30c6d07b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - DeepVariant
  - Exomiser (hiPHIVE phenotype algorithm)
  - metPropagate
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# phenotype-based-gene-prioritization-interpretation

## Summary

Per-patient, head-to-head interpretation of Exomiser's hiPHIVE phenotype-driven (Exomiser-Phenotype) candidate gene rankings against a metabolomics-driven ranking such as metPropagate's propagated scores, recording the rank of each patient's known causative gene under both schemes and flagging which tool ranked it higher.

## When to use

Use when a cohort of patients has both an Exomiser-filtered candidate gene list with hiPHIVE phenotype scores and an alternative (e.g., metabolomics-driven) per-gene score, and you need to compare causative-gene ranks per patient.

## When NOT to use

- The patient has no known/previously diagnosed causative gene — rank comparison against ground truth is not possible (the analysis needs a confirmed diagnosis, as in the 11 TIDEX patients)
- The candidate list lacks Exomiser hiPHIVE phenotype scores (e.g., only variant-effect/frequency-filtered genes with no phenotype ranking), so the Exomiser-Phenotype rank cannot be computed
- Feature-to-metabolite mapping is unavailable for the metabolomic system (as for the TIDEX metabolomic platform), which limits interpreting the metabolomics-driven side of the comparison at the metabolite level

## Inputs

- metPropagate LPA_output results files per sample (per-gene 'Score.ID' propagated scores and 'Original.Score.ID' original ME scores)
- Exomiser candidate gene lists per patient with hi-PHIVE phenotype scores (e.g., sampleA_candidate_genes.csv), with all inheritance patterns combined and highest-ranked variant retained per gene
- List of each patient's previously diagnosed causative gene (e.g., Supplementary Table 1)
- Exomiser variant-filtered candidate lists derived from DeepVariant singleton VCFs (281–609 genes per TIDEX patient)

## Outputs

- Per-patient rank table comparing causative gene rank under metPropagate (sorted by Score.ID, descending) vs Exomiser-Phenotype (e.g., metpropagate_vs_exomiser_rank_comparison.csv, 11 rows for TIDEX)
- Per-patient flags of which scheme ranked the causative gene more highly (expected: metPropagate higher in 6/11 TIDEX patients)

## How to apply

For each patient, load metPropagate's post-propagation node scores (Score.ID) and original ME scores (Original.Score.ID) from the LPA_output/integration results files, and the patient's Exomiser candidate gene list with hi-PHIVE phenotype scores (e.g., sampleA_candidate_genes.csv). When multiple Exomiser inheritance modes are present, combine gene scores across all inheritance patterns and retain the highest-ranked variant per gene so each gene has a single Exomiser-Phenotype score. Identify the patient's previously diagnosed causative gene (e.g., from Supplementary Table 1), sort the candidate list in descending order by metPropagate's propagated score, and record the causative gene's rank; repeat with the Exomiser-Phenotype score. Compare the two ranks per patient, flag the patients where metPropagate ranked the causative gene more highly, and write the per-patient table (e.g., metpropagate_vs_exomiser_rank_comparison.csv). In the TIDEX benchmark this comparison is the basis of the reported 6/11 result; note Exomiser's candidate lists arise from variant-effect and population-frequency filtering of DeepVariant singleton VCFs (281–609 genes per patient).

## Related tools

- **metPropagate** (Supplies the metabolomics-driven per-gene propagated scores (Score.ID) whose causative-gene ranks are compared against Exomiser-Phenotype) — https://github.com/emmagraham/metPropagate
- **Exomiser (hiPHIVE phenotype algorithm)** (Provides the phenotype-driven candidate gene ranking (Exomiser-Phenotype score) used as the per-patient comparator)
- **DeepVariant** (Upstream producer of the singleton VCFs that Exomiser filters into the candidate gene lists being ranked)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('integration/results/sampleA'); ranked = df.sort_values('Score.ID', ascending=False).reset_index(drop=True); print(ranked.index[ranked['ID'] == causative_gene][0] + 1)"
```

## Evaluation signals

- For each patient, the causative gene appears exactly once in both the metPropagate and Exomiser-Phenotype sorted lists, and both ranks are recorded in the comparison table
- Reproduced comparison is consistent with the reported finding: metPropagate ranked the causative gene more highly than Exomiser-Phenotype in 6/11 TIDEX patients (Exomiser-Phenotype higher in 5/11)
- Exomiser-Phenotype ranking is computed over the Exomiser candidate list (281–609 genes per patient), so each causative gene rank is bounded by that patient's list size
- All Exomiser inheritance patterns were combined with the highest-ranked variant per gene retained — verify no gene carries duplicate/multiple Exomiser scores before ranking

## Limitations

- TIDEX candidate lists came from singleton exomes rather than trio WES; trio parental data would shorten candidate lists and change ranks, and is not reflected in the reported results
- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities could not be determined on the metabolomics-driven side
- The metabolomic system could not measure large glycosaminoglycans (limiting ME analysis for the IDS patient), and no software version was reported for the STRING network or for Exomiser/hiPHIVE

## Evidence

- [abstract] ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients: "ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients"
- [intro] Exomiser’s hiPHIVE algorithm maps human phenotype ontology terms across species: "Exomiser’s hiPHIVE algorithm maps human phenotype ontology terms across species, enabling researchers to leverage databases of well-phenotyped single-gene knock out animal models"
- [results] Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes: "Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes"
- [discussion] exhibits comparable performance to existing phenotype-based prioritization tools such as Exomiser’s hiPHIVE phenotype algorithm: "exhibits comparable performance to existing phenotype-based prioritization tools such as Exomiser’s hiPHIVE phenotype algorithm"
- [readme] File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID".: "File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID"."

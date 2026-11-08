---
name: reaction-activity-score-computation-from-transcriptomics
description: Use when you have a SBML metabolic model with GPR rules and a gene-level
  transcriptomics table (e.g. FPKM values for breast cancer cell lines) and need reaction-level
  expression scores for constraint-based flux analysis or multi-omics flux-regulation
  classification.
when_to_use_negative:
- Input is a pre-computed reaction-level score file (e.g. already-computed RAS floats
  without GPR rules to resolve) — use the normalization step directly instead of re-running
  GPR extraction and score resolution
- Transcriptomics data are not gene-level measurements matched to the model's gene
  identifiers, or the model has no GPR rules to extract
- The goal is metabolomics-derived flux prediction (e.g. mass-action substrate-availability
  scoring) rather than transcriptomics-derived reaction expression scoring
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3922
tools:
- name: INTEGRATE
  role: The pipeline hosting getGPRsFromModel, getRASscore and getNormalizedRAS scripts
    that perform RAS extraction, resolution and normalization
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime used to execute the getGPRsFromModel.py, getRASscore.py and getNormalizedRAS.py
    scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:3b36b6a9987177df963ce83d2de5435d541bb1cdb1d15bda799e990893ed6296
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# reaction-activity-score-computation-from-transcriptomics

## Summary

Computes per-cell-line Reaction Activity Scores (RAS) by resolving the GPR logical rules of a constraint-based metabolic model (e.g. ENGRO2) over transcriptomics data: taking the minimum transcript value across AND-linked genes (enzyme subunits) and the sum across OR-linked genes (isoforms). These RAS values are the transcriptomics-derived input for INTEGRATE's cell-relative model integration and downstream regulation-classification steps.

## When to use

Use when you have a SBML metabolic model with GPR rules and a gene-level transcriptomics table (e.g. FPKM values for breast cancer cell lines) and need reaction-level expression scores for constraint-based flux analysis or multi-omics flux-regulation classification.

## When NOT to use

- Input is a pre-computed reaction-level score file (e.g. already-computed RAS floats without GPR rules to resolve) — use the normalization step directly instead of re-running GPR extraction and score resolution
- Transcriptomics data are not gene-level measurements matched to the model's gene identifiers, or the model has no GPR rules to extract
- The goal is metabolomics-derived flux prediction (e.g. mass-action substrate-availability scoring) rather than transcriptomics-derived reaction expression scoring

## Inputs

- SBML metabolic model with GPR rules (default: ENGRO2 model)
- Gene-level transcriptomics dataset as a tab-separated file with FPKM values per cell line (default: FPKM_Breast_forMarea.tsv, columns for MCF102A, MDAMB231, SKBR3, MCF7, MDAMB361)
- Regex for extracting gene identifiers from GPR rules (default r"([A-Z0-9.]+)")

## Outputs

- modelId_GPR.csv: one row per reaction with columns id and rule
- modelId_RAS.csv: column Rxn plus one RAS column per input cell line
- modelId_wNormalizedRAS.csv: per reaction, mean_XXX and norm_XXX columns for each cell line XXX

## How to apply

First run getGPRsFromModel.py on the input model (default modelId 'ENGRO2') to extract each reaction's GPR rule into a CSV with columns id and rule. Then run getRASscore.py with gprRule pointing to that file, rnaSeqFileName pointing to the transcriptomics table (default 'FPKM_Breast_forMarea.tsv') and regexOrgSpecific set to a regex that captures the gene identifiers inside the GPR rules (default r"([A-Z0-9.]+)", the dot also catching already-computed float scores); the script resolves each rule by assigning the min transcript value to AND-linked genes (shared enzyme subunits) and the sum to OR-linked genes (isoforms), producing modelId_RAS.csv with a Rxn column and one RAS column per sample. Finally run getNormalizedRAS.py on the RAS file to add per-reaction mean_XXX and norm_XXX columns, normalizing each cell line's score on the maximum across all cell lines. The normalized RAS then feeds rasIntegration.py (via rasNormFileName) to impose transcriptomics-derived constraints in the cell-relative SBML models.

## Related tools

- **INTEGRATE** (The pipeline hosting getGPRsFromModel, getRASscore and getNormalizedRAS scripts that perform RAS extraction, resolution and normalization) — https://github.com/qLSLab/integrate
- **Python** (Runtime used to execute the getGPRsFromModel.py, getRASscore.py and getNormalizedRAS.py scripts)

## Examples

```
python pipeline/getGPRsFromModel.py && python pipeline/getRASscore.py && python pipeline/getNormalizedRAS.py
```

## Evaluation signals

- modelId_GPR.csv exists and contains one row per reaction with both id and rule columns
- modelId_RAS.csv contains a Rxn column and exactly one RAS column per input cell line (MCF102A, MDAMB231, SKBR3, MCF7, MDAMB361)
- modelId_wNormalizedRAS.csv contains, for each reaction, a mean_XXX and a norm_XXX column for each of the five input cell lines
- For reactions whose GPR contains AND connectors, the RAS equals the minimum (not the mean) of the linked genes' transcript values; for OR connectors it equals the sum
- Normalized RAS values (norm_XXX) fall within [0,1] after max-normalization across cell lines

## Limitations

- RAS ignores post-transcriptional regulation: the score is a proxy for enzyme-level activity derived only from mRNA abundance resolved through GPR logic
- Reversible reactions in the model must later be split for downstream t-tests (rasTtest step), since the RAS file does not handle directionality itself
- The default regex r"([A-Z0-9.]+)" must match the gene identifiers actually present in the GPR rules; identifiers not captured are silently dropped from scoring

## Evidence

- [other] The RAS resolves GPR logical rules over transcript levels using the min value for AND-linked genes (enzyme subunits) and the sum for OR-linked genes (isoforms): [evidence span withheld — anchor status: not_found]
- [readme] GPR extraction produces id/rule CSV: "File named modelId + '_GPR.csv' containing for each reaction (column *id*) its GPR rule (column *rule*)."
- [readme] RAS output schema: "File named modelId + '_RAS.csv' containing a column including the reactions ID (column *Rxn*), and a column for each cell line in the input transcriptomics dataset"
- [readme] Normalized RAS output schema: "containing for each reaction (column *Rxn*) the mean (*mean_XXX* column) and normalized (*norm_XXX* column) RAS for each cell line"
- [abstract] Purpose: differential reaction expression from transcriptomics: "We compute differential reaction expression from transcriptomic data and use constraint-based modeling to predict if the differential expression of metabolic enzymes directly originates differences"
- [readme] Default transcriptomics file name: "rnaSeqFileName = transcriptomics dataset file name. Default value: 'FPKM_Breast_forMarea.tsv'"

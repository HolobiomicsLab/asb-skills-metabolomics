---
name: reaction-activity-score-computation-from-gpr-rules
description: Use when transcriptomics data (e.g. an FPKM table per cell line) must
  be converted into per-reaction scores via the GPR rules of a specific metabolic
  model (ENGRO2, Recon3D subset) to feed downstream RAS integration, t-tests or concordance
  analysis.
when_to_use_negative:
- Transcriptomics is already a per-reaction score table (e.g. a precomputed RAS file)
  — re-running getGPRsFromModel/getRASscore would duplicate work
- The analysis goal is flux-level or metabolomics-level prediction (randomSampling,
  createMetabolicDataset) rather than transcriptional reaction expression
- The metabolic model and the transcriptomics gene identifiers cannot be matched by
  any regex — the score would be undefined for most rules
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: INTEGRATE
  role: Pipeline whose Steps 1–3 (getGPRsFromModel, getRASscore, getNormalizedRAS)
    implement GPR-based reaction activity score computation
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime environment for the pipeline scripts (release version 3.0 or later)
- name: MATLAB
  role: Used only in later steps (model conversion to mat file and irreversible split);
    not needed for RAS computation itself
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:2089538d3023d557dbe49c5581e0fee62f1e5298bcd6ca8ee94b49706275291f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Reaction Activity Score (RAS) computation from GPR rules

## Summary

Computes per-reaction activity scores for each cell line by propagating gene expression values through the gene-protein-reaction (GPR) rules extracted from a constraint-based metabolic model, as done in the INTEGRATE pipeline using ENGRO2 or Recon3D-derived models and FPKM breast transcriptomics data.

## When to use

Use when transcriptomics data (e.g. an FPKM table per cell line) must be converted into per-reaction scores via the GPR rules of a specific metabolic model (ENGRO2, Recon3D subset) to feed downstream RAS integration, t-tests or concordance analysis.

## When NOT to use

- Transcriptomics is already a per-reaction score table (e.g. a precomputed RAS file) — re-running getGPRsFromModel/getRASscore would duplicate work
- The analysis goal is flux-level or metabolomics-level prediction (randomSampling, createMetabolicDataset) rather than transcriptional reaction expression
- The metabolic model and the transcriptomics gene identifiers cannot be matched by any regex — the score would be undefined for most rules

## Inputs

- SBML metabolic model (e.g. ENGRO2 or a Recon3D-derived model limited to ENGRO2-active exchange reactions)
- Transcriptomics dataset file (FPKM values per gene per cell line, e.g. FPKM_Breast_forMarea.tsv)
- Gene-extraction regular expression for organism-specific gene identifiers (default r"([A-Z0-9.]+)")

## Outputs

- GPR rule table: modelId + '_GPR.csv' with reaction id and GPR rule per reaction
- RAS score table: modelId + '_RAS.csv' with reaction 'Rxn' column and one RAS score per cell line
- (optional) normalized RAS table: modelId + 'wNormalizedRAS.csv' with mean and norm columns per cell line

## How to apply

Run Step 1 (getGPRsFromModel) on the SBML model — modelId default 'ENGRO2' — to produce modelId+'_GPR.csv' listing each reaction id and its rule. Then run Step 2 (getRASscore) with that GPR file and the transcriptomics dataset (default 'FPKM_Breast_forMarea.tsv'), using regexOrgSpecific = r"([A-Z0-9.]+)" to extract gene identifiers (including already-computed float scores) from the rules. The output modelId+'_RAS.csv' contains the reaction ID column 'Rxn' plus one RAS score column per cell line in the transcriptomics table. Optional normalization (Step 3, getNormalizedRAS) yields mean and normalized ('norm_XXX') columns per cell line for subsequent rasIntegration. For a model variant such as the Recon3D subset restricted to ENGRO2-active exchange reactions, set modelId accordingly so the RAS table matches the input model's reaction space.

## Related tools

- **INTEGRATE** (Pipeline whose Steps 1–3 (getGPRsFromModel, getRASscore, getNormalizedRAS) implement GPR-based reaction activity score computation) — https://github.com/qLSLab/integrate
- **Python** (Runtime environment for the pipeline scripts (release version 3.0 or later))
- **MATLAB** (Used only in later steps (model conversion to mat file and irreversible split); not needed for RAS computation itself)

## Examples

```
python pipeline/getGPRsFromModel.py && python pipeline/getRASscore.py && python pipeline/getNormalizedRAS.py
```

## Evaluation signals

- Output file names match the convention modelId+'_GPR.csv' and modelId+'_RAS.csv'
- The RAS table's 'Rxn' column contains exactly the reactions of the input model's GPR file, and there is one score column per cell line present in the transcriptomics input
- For a cell line whose genes are absent from the transcriptomics file, the RAS column is empty/NaN rather than silently zero — check no silent defaulting
- Gene identifiers extracted from GPR rules via regexOrgSpecific cover the gene set used in the transcriptomics table (spot-check a few rules, e.g. ACONT in the article's concordance example)
- When applied to the Recon3D subset (modelId set to the ENGRO2-active exchange-reaction variant), the RPE reaction is scored in the same regulatory categories as with ENGRO2, per the article's sensitivity analysis

## Limitations

- The regex default r"([A-Z0-9.]+)" assumes gene identifiers are alphanumeric with dots; other naming conventions require an adjusted regex or scores will be missing
- RAS scores are only meaningful relative to the specific model — running them on a Recon3D subset (limited to ENGRO2-active exchange reactions) is the only variant the article validated, and concordance scores depend on the model choice
- Scores propagate gene expression through GPR logic and do not account for enzyme kinetics or protein abundance beyond gene expression
- Transcriptomics data are FPKM-like continuous values; the method was demonstrated on breast cell-line data and its behaviour on other tissues or technologies (e.g. raw counts) is not established

## Evidence

- [methods] GPR rule extraction from model: "Step 1: getGPRsFromModel * Aim: Takes input model and returns the GPR rules associated to each reaction"
- [methods] Regex for gene extraction: "regexOrgSpecific: define regex to extract genes from GPR rules. Default value: r"([A-Z0-9.]+)" where dot is needed to also extract already computer scores, which are float."
- [methods] Transcriptomics input file: "rnaSeqFileName = transcriptomics dataset file name. Default value: 'FPKM_Breast_forMarea.tsv'"
- [methods] Output schema: "File named modelId + '_RAS.csv' containing a column including the reactions ID (column *Rxn*), and a column for each cell line in the input transcriptomics dataset corresponding to the computer RAS"
- [results] Model choice sensitivity (Recon3D): [evidence span withheld — anchor status: not_found]
- [readme] Overview of RAS purpose: "We compute differential reaction expression from transcriptomic data and use constraint-based modeling to predict if the differential expression of metabolic enzymes directly originates differences"
- [readme] Usage command: "Usage: `python pipeline/getRASscore.py`"

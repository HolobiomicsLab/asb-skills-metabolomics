---
name: gut-microbial-biotransformation-prediction
description: Use when a curated parent-chemical list (SMILES/structures, ~20,000 exposome
  chemicals) needs microbiome-mediated metabolite prediction and per-module counts
  as part of a biotransformation metabolite database build.
when_to_use_negative:
- Parent chemical has molecular weight >900 Da or a chemical category not matched
  by BioTransformer's supported categories (4,225 such chemicals were excluded in
  the study)
- Goal is risk-based prioritization of parent chemicals using exposure (SEEM) and
  toxicity (TEST/ToxPi) data — this skill predicts metabolites only
- Need for measured (not predicted) metabolite data or for biotransformation half-lives
  (HLBs), which used the Iterative Fragment Selection approach instead
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: BioTransformer
  role: Machine-learning predictor whose human gut microbial module generates microbiome-mediated
    biotransformation metabolites from parent chemical structures
  repo: github.com/FangLabNTU/HExpMetDB
- name: Java
  role: Runtime platform for executing BioTransformer (Oracle Java software basis)
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:90da2ed389b3ad429782d15959f251f48998a7104ddfec630bbd71ff7f72fae3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - Java
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gut-microbial-biotransformation-prediction

## Summary

Runs BioTransformer's human gut microbial module over a compiled human exposome chemical list to predict microbiome-mediated biotransformation metabolites, contributing 6,337 of the 95,976 predicted metabolites in HExpMetDB built for the >20,000-chemical exposome database.

## When to use

Use when a curated parent-chemical list (SMILES/structures, ~20,000 exposome chemicals) needs microbiome-mediated metabolite prediction and per-module counts as part of a biotransformation metabolite database build.

## When NOT to use

- Parent chemical has molecular weight >900 Da or a chemical category not matched by BioTransformer's supported categories (4,225 such chemicals were excluded in the study)
- Goal is risk-based prioritization of parent chemicals using exposure (SEEM) and toxicity (TEST/ToxPi) data — this skill predicts metabolites only
- Need for measured (not predicted) metabolite data or for biotransformation half-lives (HLBs), which used the Iterative Fragment Selection approach instead

## Inputs

- Compiled human exposome chemical list with SMILES/structures (~20,000 chemicals from the HExpMetDB build step)
- Chemical category and molecular weight annotations for filtering (MW >900 Da and nonmatched categories excluded)
- BioTransformer GUI example compound DEHP for validation

## Outputs

- Predicted human gut microbial biotransformation metabolite list (6,337 metabolites from 20,061 parent chemicals)
- Per-module metabolite counts (gut microbial 6,337 within totals of 95,976 total; 19,212 EC-based; 72,193 CYP450; 15,762 phase II)
- Deduplicated metabolite structures ingested into the predicted biotransformation metabolite database (HExpMetDB)

## How to apply

Load the compiled chemical list (SMILES/structures from the HExpMetDB build step) and first apply the exclusion filter: chemicals with molecular weight >900 Da or nonmatched chemical categories were excluded, removing 4,250 chemicals from the original set and leaving 20,061 parents for prediction. Execute BioTransformer's human gut microbial module (Java-based) on the surviving parent structures, recording per-chemical and per-module prediction counts. Aggregate predictions across the four modules (EC-based, CYP450, phase II, and human gut microbial), deduplicate identical structures, and sum parents' and metabolites' predictions — the gut microbial module contributed 6,337 metabolites against totals of 95,976 (19,212 EC-based, 72,193 CYP450, 15,762 phase II). Spot-check the setup by running the same modules on DEHP via the BioTransformer GUI example and comparing the output metabolite list against the published example to confirm expected DEHP metabolites are reproduced.

## Related tools

- **BioTransformer** (Machine-learning predictor whose human gut microbial module generates microbiome-mediated biotransformation metabolites from parent chemical structures) — github.com/FangLabNTU/HExpMetDB
- **Java** (Runtime platform for executing BioTransformer (Oracle Java software basis)) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- Aggregate metabolite counts must equal the reported totals: 95,976 overall with 6,337 human gut microbial metabolites from 20,061 chemicals (alongside 19,212 EC-based, 72,193 CYP450, and 15,762 phase II)
- Exactly 20,061 chemicals pass the filter after excluding those with MW >900 Da or nonmatched chemical categories (4,250 exclusions)
- Deduplicated identical structures across the four modules — no duplicate metabolite structures in the final database
- DEHP spot-check: running the same prediction modules on DEHP reproduces the metabolites reported in the BioTransformer GUI example

## Limitations

- BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals
- Chemicals with molecular weight >900 Da or nonmatched chemical categories cannot be predicted (4,225 chemicals were excluded on this basis)
- Results are in-silico predictions only and require integration with measured exposome data for biomonitoring interpretation

## Evidence

- [abstract] Predicted biotransformation products using BioTransformer: "biotransformation products were predicted using the previously developed BioTransformer machine-learning method"
- [abstract] Provided a predicted biotransformation metabolite database of >95,000 metabolites: "provided a predicted biotransformation metabolite database of >95,000 metabolites"
- [discussion] Prediction inaccuracy caveat: "BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals"
- [abstract] Compiled human exposome chemical input: "We compiled a human exposome database of >20,000 chemicals"

---
name: cyp450-phasei-phaseii-biotransformation-rule-application
description: Use when you have a compiled chemical list with SMILES/structures and
  need to enumerate in-silico biotransformation products across hepatic (phase I/II),
  EC-based, and gut microbial pathways.
when_to_use_negative:
- The parent chemicals contain metal atoms or exceed MW thresholds that BioTransformer
  cannot process — the article notes such chemicals were excluded (4,225 removed)
  rather than predicted
- The goal is experimentally confirmed metabolite identification rather than in-silico
  prediction — BioTransformer predictions may be incomplete or inaccurate
- The chemical list lacks CASRN/structure information needed to run the prediction
  modules
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: BioTransformer
  role: Machine-learning engine whose four modules (EC-based, CYP450, phase II, human
    gut microbial) generate the predicted metabolites for each parent chemical
  repo: github.com/FangLabNTU/HExpMetDB
- name: Java
  role: Runtime/underlying platform (Java software, Oracle) used with BioTransformer
    and the downstream search GUI
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:eebd25cec71075e9b37dad35b34cfa5c2a4c4ffef995fcfa7ab7a4486bdd1d1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - Java
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cyp450-phasei-phaseii-biotransformation-rule-application

## Summary

Applies the BioTransformer machine-learning biotransformation prediction pipeline — EC-based, CYP450 (phase I), phase II, and human gut microbial modules — to a large human exposome chemical list to generate a predicted metabolite database (HExpMetDB, >95,000 metabolites).

## When to use

Use when you have a compiled chemical list with SMILES/structures and need to enumerate in-silico biotransformation products across hepatic (phase I/II), EC-based, and gut microbial pathways.

## When NOT to use

- The parent chemicals contain metal atoms or exceed MW thresholds that BioTransformer cannot process — the article notes such chemicals were excluded (4,225 removed) rather than predicted
- The goal is experimentally confirmed metabolite identification rather than in-silico prediction — BioTransformer predictions may be incomplete or inaccurate
- The chemical list lacks CASRN/structure information needed to run the prediction modules

## Inputs

- SMILES/structure list for ~20,000 human exposome parent chemicals (from the HExpMetDB build step)
- Chemical category and molecular weight annotations used for the MW >900 Da / nonmatched-category exclusion

## Outputs

- Predicted biotransformation metabolite database (HExpMetDB) of >95,000 metabolites linked to parent chemicals
- Per-module metabolite counts (EC-based, CYP450, phase II, human gut microbial)
- Spot-check metabolite list for a reference compound (DEHP) matching the BioTransformer GUI example

## How to apply

Load the compiled chemical structures (SMILES for ~20,000 human exposome chemicals from HExpMetDB) and exclude chemicals with MW >900 Da or nonmatched chemical categories (this removed 4,225 chemicals, leaving 20,061). Run the four BioTransformer modules in sequence: the EC-based module, the CYP450 phase I module, the phase II module on parent chemicals (and, where BioTransformer supports it, on intermediate metabolites), and the human gut microbial module. Aggregate per-module metabolite counts, deduplicate identical structures across modules, and sum parent and metabolite predictions to obtain the total. Validate against expected distributions (here: 19,212 EC-based, 72,193 CYP450, 15,762 phase II, 6,337 gut microbial; 95,976 total) and spot-check the setup by rerunning a known case, e.g. the DEHP GUI example, and comparing the predicted metabolite list.

## Related tools

- **BioTransformer** (Machine-learning engine whose four modules (EC-based, CYP450, phase II, human gut microbial) generate the predicted metabolites for each parent chemical) — github.com/FangLabNTU/HExpMetDB
- **Java** (Runtime/underlying platform (Java software, Oracle) used with BioTransformer and the downstream search GUI) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- Total deduplicated metabolite count matches the reported aggregate (95,976 from 20,061 chemicals)
- Per-module counts reproduce the expected distribution: 19,212 EC-based, 72,193 CYP450, 15,762 phase II, and 6,337 human gut microbial
- Exclusion bookkeeping: the number of chemicals removed for MW >900 Da or nonmatched categories equals the reported 4,225
- DEHP spot-check output metabolite list matches the metabolites reported in the BioTransformer GUI example
- No duplicate metabolite structures remain after aggregation across modules

## Limitations

- BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals
- Chemicals with MW >900 Da or nonmatched chemical categories cannot be predicted and were excluded
- Predicted metabolites are in-silico entries and require experimental/analytical confirmation before use in biomonitoring

## Evidence

- [abstract] BioTransformer modules used on exposome chemicals: "biotransformation products were predicted using the previously developed BioTransformer machine-learning method"
- [abstract] Scale of the predicted metabolite output: "provided a predicted biotransformation metabolite database of >95,000 metabolites"
- [discussion] Known incompleteness and accuracy caveats: "BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals"

---
name: biotransformer-batch-metabolite-prediction
description: Use when a compiled chemical list (SMILES/structures, e.g. the >20,000-chemical
  human exposome set) needs in-silico metabolite prediction across EC-based, CYP450,
  phase II, and gut microbial biotransformation modules.
when_to_use_negative:
- The input chemical has MW >900 Da or belongs to a nonmatched chemical category —
  such chemicals were excluded from prediction (4,225 of the original list)
- The goal is experimentally confirmed metabolites rather than machine-learning-predicted
  biotransformation products
- The chemical contains structural classes outside BioTransformer's trained domain
  — BioTransformer cannot predict all metabolites and results may be inaccurate for
  some chemicals
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BioTransformer
  role: Machine-learning metabolite prediction engine; its four modules (EC-based,
    CYP450, phase II, human gut microbial) generate the predicted biotransformation
    products
- name: Java
  role: Runtime platform for executing BioTransformer (CLI and user-interactive Java
    software search GUI)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:81af2ecb28718b506c45f6c278ac877fb6e39efde9211a50eb22fa5cb576a379
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - Java
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# biotransformer-batch-metabolite-prediction

## Summary

Batch prediction of biotransformation products for a large chemical inventory using BioTransformer's four machine-learning modules (EC-based, CYP450, phase II, human gut microbial), producing a deduplicated predicted metabolite database with per-module counts.

## When to use

Use when a compiled chemical list (SMILES/structures, e.g. the >20,000-chemical human exposome set) needs in-silico metabolite prediction across EC-based, CYP450, phase II, and gut microbial biotransformation modules.

## When NOT to use

- The input chemical has MW >900 Da or belongs to a nonmatched chemical category — such chemicals were excluded from prediction (4,225 of the original list)
- The goal is experimentally confirmed metabolites rather than machine-learning-predicted biotransformation products
- The chemical contains structural classes outside BioTransformer's trained domain — BioTransformer cannot predict all metabolites and results may be inaccurate for some chemicals

## Inputs

- Compiled chemical list with SMILES/structures for ~20,000 human exposome chemicals (HExpMetDB build output)
- Module selection: EC-based, CYP450 (phase I), phase II, human gut microbial
- Reference GUI example compound (DEHP) for spot-check validation

## Outputs

- Predicted biotransformation metabolite database of >95,000 metabolites (95,976 in the reference run) with per-module attribution
- Per-module metabolite counts (EC-based, CYP450, phase II, gut microbial) and total count per parent chemical
- Deduplicated metabolite structure set across all modules

## How to apply

Load the compiled chemical list with SMILES/structures from the HExpMetDB build step (~20,000 human exposome chemicals), then execute BioTransformer's four modules in sequence: EC-based first, then the CYP450 (phase I) and phase II modules on parent chemicals (and intermediates where BioTransformer supports it), then the human gut microbial module. Aggregate per-module and total metabolite counts, deduplicating identical structures across modules and chemicals. Apply the exclusion filter applied in the source study: 4,225 chemicals were dropped for MW >900 Da or nonmatched chemical categories, leaving 20,061 chemicals for prediction. Validate by reproducing the reference totals (95,976 metabolites total; 19,212 EC-based; 72,193 CYP450; 15,762 phase II; 6,337 gut microbial) and by running the BioTransformer GUI example compound DEHP through the same modules to confirm its metabolite list matches the published GUI output.

## Related tools

- **BioTransformer** (Machine-learning metabolite prediction engine; its four modules (EC-based, CYP450, phase II, human gut microbial) generate the predicted biotransformation products)
- **Java** (Runtime platform for executing BioTransformer (CLI and user-interactive Java software search GUI))

## Evaluation signals

- Total predicted metabolites equal 95,976 from 20,061 chemicals
- Per-module counts match the reference values: 19,212 EC-based, 72,193 CYP450, 15,762 phase II, 6,337 human gut microbial
- Excluded chemical count matches 4,225 (MW >900 Da or nonmatched categories)
- Deduplication of identical structures leaves a unique metabolite set consistent with the >95,000-metabolite database total
- DEHP spot-check run reproduces the metabolite list from the published BioTransformer GUI example

## Limitations

- BioTransformer cannot predict all metabolites, and prediction results may be inaccurate for some chemicals
- Chemicals with MW >900 Da or nonmatched chemical categories cannot be processed (4,225 chemicals excluded in the reference run)
- Predicted metabolites are computational products requiring downstream experimental/biomonitoring validation

## Evidence

- [abstract] Prediction method and scale: "biotransformation products were predicted using the previously developed BioTransformer machine-learning method"
- [abstract] Output scale: "provided a predicted biotransformation metabolite database of >95,000 metabolites"
- [discussion] Accuracy caveat: "BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals"

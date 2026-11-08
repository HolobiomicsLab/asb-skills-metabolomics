---
name: metabolite-count-reconciliation
description: Use when a BioTransformer run over a large chemical list must be validated
  against published metabolite totals, per-module counts, or a single-compound reference
  output such as the DEHP GUI example.
when_to_use_negative:
- The chemical set includes metals or compounds with MW >1,000 Da that the upstream
  IFS/HLB pipeline cannot handle and for which the reference counts were not computed
- The goal is experimental metabolite identification rather than reconciling in-silico
  BioTransformer predictions against published counts
- Chemical structures lack SMILES or cannot be categorized, so the MW >900 Da / chemical-category
  exclusion cannot be applied consistently
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0605
tools:
- name: BioTransformer
  role: Generates the biotransformation metabolite predictions (EC-based, CYP450,
    phase II, human gut microbial modules) whose counts are reconciled against the
    published totals and the DEHP GUI example
  repo: github.com/FangLabNTU/HExpMetDB
- name: Java
  role: Runtime environment for executing BioTransformer and the user-interactive
    Java (Oracle)-based search GUI used to access the metabolite database
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:d220cd71cc023675efbdecfbbbae1a3ebb6d5468f135ff924ac1eb4bb7de9dcd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - Java
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolite-count-reconciliation

## Summary

Reconciles BioTransformer biotransformation predictions against published module-level counts, verifying that the total number of predicted metabolites (95,976 from 20,061 chemicals) and the per-module distribution (EC-based, CYP450, phase II, human gut microbial) match, plus a DEHP spot-check against the BioTransformer GUI example.

## When to use

Use when a BioTransformer run over a large chemical list must be validated against published metabolite totals, per-module counts, or a single-compound reference output such as the DEHP GUI example.

## When NOT to use

- The chemical set includes metals or compounds with MW >1,000 Da that the upstream IFS/HLB pipeline cannot handle and for which the reference counts were not computed
- The goal is experimental metabolite identification rather than reconciling in-silico BioTransformer predictions against published counts
- Chemical structures lack SMILES or cannot be categorized, so the MW >900 Da / chemical-category exclusion cannot be applied consistently

## Inputs

- Compiled chemical list with SMILES/structures for ~20,000 human exposome chemicals (from the HExpMetDB build step)
- Published reference counts: 95,976 total; 19,212 EC-based; 72,193 CYP450; 15,762 phase II; 6,337 human gut microbial; 20,061 parent chemicals
- BioTransformer GUI example output for DEHP (reference metabolite list)

## Outputs

- Per-module metabolite counts (EC-based, CYP450, phase II, human gut microbial)
- Deduplicated total metabolite count across modules
- Exclusion report (number of chemicals dropped for MW >900 Da or nonmatched chemical categories)
- DEHP spot-check comparison table (predicted vs. GUI-example metabolites)

## How to apply

Run each BioTransformer module (EC-based, CYP450, phase II, human gut microbial) over the compiled chemical list with SMILES structures, recording per-module prediction counts per chemical. Before counting, apply the same exclusion filters as the reference study: drop chemicals with MW >900 Da or nonmatched chemical categories (4,225 chemicals were excluded, leaving 20,061). Aggregate per-module and total counts, deduplicate identical structures across modules, and compare against the reference totals: 19,212 EC-based, 72,193 CYP450, 15,762 phase II, and 6,337 human gut microbial metabolites (95,976 total). The rationale is that exact-count agreement plus a single-compound spot-check (DEHP against the BioTransformer GUI example) is the practical validation available for large-scale in-silico metabolism predictions, since BioTransformer cannot predict all metabolites and some predictions may be inaccurate.

## Related tools

- **BioTransformer** (Generates the biotransformation metabolite predictions (EC-based, CYP450, phase II, human gut microbial modules) whose counts are reconciled against the published totals and the DEHP GUI example) — github.com/FangLabNTU/HExpMetDB
- **Java** (Runtime environment for executing BioTransformer and the user-interactive Java (Oracle)-based search GUI used to access the metabolite database) — github.com/FangLabNTU/HExpMetDB

## Examples

```
java -jar BioTransformer3.jar -q -t pred -lb 1 -spp 3 -m All -molstruct "DEHP smiles" -molformat smi -o dehp_metabolites; python reconcile_counts.py --modules ec,cyp450,phaseII,micro --ref 95976
```

## Evaluation signals

- Total deduplicated metabolite count equals 95,976 predictions from 20,061 chemicals
- Module-level counts equal 19,212 EC-based, 72,193 CYP450, 15,762 phase II, and 6,337 human gut microbial metabolites
- Number of excluded chemicals (MW >900 Da or nonmatched chemical categories) equals 4,225
- DEHP spot-check reproduces the metabolite list reported in the BioTransformer GUI example
- Per-chemical module counts sum consistently to the aggregate totals after deduplication

## Limitations

- BioTransformer cannot predict all metabolites, and prediction results may be inaccurate for some chemicals
- Chemicals with MW >900 Da or nonmatched chemical categories must be excluded, which reduced the runnable set from the full ~20,000-chemical list to 20,061
- Counts depend on running modules on parents and, where supported, intermediate metabolites, so aggregation/deduplication rules affect totals

## Evidence

- [results] BioTransformer predicted 95,976 metabolites from 20,061 chemicals: [evidence span withheld — anchor status: not_found]
- [results] exclusion of 4,225 chemicals: [evidence span withheld — anchor status: not_found]
- [discussion] BioTransformer limitations: "BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals"
- [abstract] predictions via BioTransformer: "biotransformation products were predicted using the previously developed BioTransformer machine-learning method"
- [methods] DEHP GUI validation: [evidence span withheld — anchor status: not_found]

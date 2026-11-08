---
name: classyfire-chemical-classification-mapping
description: Use when an annotated metabolite feature table (with group/module assignments
  such as WGCNA co-accumulation modules) needs chemical-class labels and per-group
  ClassyFire term enrichment analysis.
when_to_use_negative:
- Metabolite features are not yet annotated (no compound identities) — run MetMiner's
  annotation with the plant-specific MS database first
- There are no predefined groups or modules to test over-representation against
- The dataset is non-plant metabolomics where the plant-specific annotation database
  is inappropriate
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_0154
tools:
- name: MDAtoolkits
  role: Provides the metabolite classification and enrichment analysis functions used
    to map ClassyFire terms and run per-module over-representation tests
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: MetMiner
  role: Produces the annotated metabolite table and module assignments that the ClassyFire
    classifications are mapped onto
  repo: https://github.com/ShawnWx2019/MetMiner
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:8733b0adbeb242763e03a37b1fe8932b9e133fcb51e1502f207d9627dbe9223f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MDAtoolkits
  tools:
  - MDAtoolkits
  - MetMiner
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# classyfire-chemical-classification-mapping

## Summary

Assign ClassyFire chemical classifications to metabolite features and test per-group over-representation of ClassyFire terms using MDAtoolkits, e.g. to interpret WGCNA co-accumulation modules chemically.

## When to use

Use when an annotated metabolite feature table (with group/module assignments such as WGCNA co-accumulation modules) needs chemical-class labels and per-group ClassyFire term enrichment analysis.

## When NOT to use

- Metabolite features are not yet annotated (no compound identities) — run MetMiner's annotation with the plant-specific MS database first
- There are no predefined groups or modules to test over-representation against
- The dataset is non-plant metabolomics where the plant-specific annotation database is inappropriate

## Inputs

- Metabolite feature table with annotations (compound IDs from MetMiner's plant-specific MS database annotation)
- Group/module assignment table (e.g. WGCNA module assignments for the 1,365 metabolites from the 206 Arabidopsis F-box mutant dataset)

## Outputs

- ClassyFire classification labels mapped onto each metabolite feature
- Per-module over-representation test results for ClassyFire terms (enriched classes, significance values)

## How to apply

Take the metabolite annotations produced by MetMiner's plant-specific MS1/MS2 database matching and attach ClassyFire compound classifications to each feature (e.g. the 1,365 metabolites assigned to 12 iterative-WGCNA modules). Using MDAtoolkits' classification and enrichment functions, compute over-representation of ClassyFire terms within each module against the annotated background. Interpret significant classes as chemical signatures of each module (the case study found 7 of 12 modules significantly enriched). Runs whose enrichment counts diverge from the reference outcome should be flagged.

## Related tools

- **MDAtoolkits** (Provides the metabolite classification and enrichment analysis functions used to map ClassyFire terms and run per-module over-representation tests) — https://github.com/ShawnWx2019/MDAtoolkits
- **MetMiner** (Produces the annotated metabolite table and module assignments that the ClassyFire classifications are mapped onto) — https://github.com/ShawnWx2019/MetMiner

## Examples

```
remotes::install_github('ShawnWx2019/MDAtoolkits', ref = 'master')
```

## Evaluation signals

- In the reference case study, 7 of the 12 WGCNA modules showed significant ClassyFire term enrichment — a run diverging from this should be flagged
- Every annotated metabolite in the mapped table carries a ClassyFire classification term
- Enrichment is computed per module against the full annotated feature background using MDAtoolkits classification/enrichment functions
- Enriched ClassyFire terms should be chemically interpretable within modules (e.g. consistent compound classes per co-accumulation module)

## Limitations

- Depends on metabolite annotation quality; suspected in-source fragmentation products may be wrongly annotated as genuine metabolites, and MetMiner currently has no satisfactory solution to detect and remove such redundant annotations
- Over-representation results are only meaningful for modules whose members were successfully annotated with ClassyFire terms

## Evidence

- [abstract] MDAtoolkits classification and enrichment role: "We have also developed MDAtoolkits, which include a complete set of tools for statistical analysis, metabolite classification, and enrichment analysis"
- [full_text] Case-study module classification/enrichment workflow: [evidence span withheld — anchor status: not_found]
- [readme] MDAtoolkits installation from README: "if (!require('MDAtoolkits')) remotes::install_github('ShawnWx2019/MDAtoolkits',ref = 'master');"
- [full_text] Validation criterion of 7 of 12 enriched modules: [evidence span withheld — anchor status: not_found]

---
name: overrepresentation-analysis-single-cell-metabolomics
description: Use when you have MS1-only metabolite annotations (e.g., METASPACE annotations
  at FDR <= 0.1) from a single-cell or spatial metabolomics experiment and need ORA
  that accounts for isomeric/isobaric identification ambiguity.
when_to_use_negative:
- The input already provides resolved molecular structures (e.g., MS2-confirmed identifications),
  so isomeric/isobaric ambiguity does not need to be propagated
- The goal is rank-based metabolite set enrichment analysis (MSEA) with continuous
  fold-change statistics rather than binary overrepresentation of an annotation query
  set
- The background terms of interest (e.g., LION ontology) are not mapped to molecule
  names or sum formulas and thus cannot be analyzed with or without ambiguity consideration
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: S2IsoMEr
  role: R package that performs bootstrapping-based ORA/MSEA, propagating isomeric/isobaric
    ambiguity into enrichment results; provides initEnrichment, Load_background, Run_bootstrap_ORA,
    Run_simple_ORA, and Run_enrichment
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: METASPACE
  role: Source of FDR-scored metabolite annotations used as the ORA query set and
    custom universe
- name: metaspace-converter
  role: Python package to download and convert METASPACE datasets to AnnData objects
    for import into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import AnnData objects exported from METASPACE into R for
    S2IsoMEr
  repo: https://github.com/scverse/anndataR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:8e409cc299dd5fa14199ab4a1f486d20be50c59397691a64b21c9b31853b4374
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - anndataR
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# overrepresentation-analysis-single-cell-metabolomics

## Summary

Perform bootstrapping-based overrepresentation analysis (ORA) on single-cell metabolomics annotations with S2IsoMEr, propagating isomeric/isobaric ambiguity into enrichment results and reporting ambiguity scores, as demonstrated on the SpaceM NASH HepaRG dataset with HMDB v4 annotations.

## When to use

Use when you have MS1-only metabolite annotations (e.g., METASPACE annotations at FDR <= 0.1) from a single-cell or spatial metabolomics experiment and need ORA that accounts for isomeric/isobaric identification ambiguity.

## When NOT to use

- The input already provides resolved molecular structures (e.g., MS2-confirmed identifications), so isomeric/isobaric ambiguity does not need to be propagated
- The goal is rank-based metabolite set enrichment analysis (MSEA) with continuous fold-change statistics rather than binary overrepresentation of an annotation query set
- The background terms of interest (e.g., LION ontology) are not mapped to molecule names or sum formulas and thus cannot be analyzed with or without ambiguity consideration

## Inputs

- Single-cell metabolomics count matrix in RDS format (e.g., NASH_scm_dataset.rds from Zenodo record 13318721)
- METASPACE per-dataset annotation results in RDS format (e.g., SpaceM_metaspace_ds_annotations.rds)
- METASPACE annotations filtered to FDR <= 0.1 against the HMDB v4 database
- Metabolite/lipid background term sets (RAMP-DB 2.0 curated pathways and classes, or LION ontology for lipids)

## Outputs

- Per-term ORA enrichment results (significance/p-values) from bootstrapping-based analysis with reported isomeric/isobaric ambiguity scores
- Standard (no-bootstrapping) ORA results for comparison
- Merged comparison table listing per-term significance under each method plus ambiguity scores, flagging standard-ORA-only significant terms

## How to apply

Download the single-cell metabolomics matrix (NASH_scm_dataset.rds) and METASPACE per-dataset annotations (SpaceM_metaspace_ds_annotations.rds) from Zenodo record 13318721, and filter annotations to those passing FDR <= 0.1 against HMDB v4 for the condition of interest (e.g., U control vs F), using passing annotations as the ORA query and all detected HMDB annotations as a custom universe. Create an S2IsoMEr enrichment object via initEnrichment with the count matrix in scmatrix, and load term-to-molecule backgrounds with Load_background. Run bootstrapping-based ORA via Run_bootstrap_ORA (or the Run_enrichment wrapper) with consider_isomers = TRUE and consider_isobars = TRUE and report_ambiguity_scores = TRUE, so iterative random sampling propagates molecular ambiguity into summarized enrichment results. Optionally run Run_simple_ORA with consider_isomers and consider_isobars both FALSE as a no-bootstrapping baseline, then merge results to flag terms significant only under standard ORA that carry low ambiguity scores (<0.5), which are likely false positives.

## Related tools

- **S2IsoMEr** (R package that performs bootstrapping-based ORA/MSEA, propagating isomeric/isobaric ambiguity into enrichment results; provides initEnrichment, Load_background, Run_bootstrap_ORA, Run_simple_ORA, and Run_enrichment) — https://github.com/alexandrovteam/S2IsoMEr
- **METASPACE** (Source of FDR-scored metabolite annotations used as the ORA query set and custom universe)
- **metaspace-converter** (Python package to download and convert METASPACE datasets to AnnData objects for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import AnnData objects exported from METASPACE into R for S2IsoMEr) — https://github.com/scverse/anndataR

## Examples

```
require(S2IsoMEr); obj <- initEnrichment(scmatrix = scmatrix, query = fdr0.1_hmdb_annotations, universe = all_hmdb_annotations); Run_bootstrap_ORA(obj, consider_isomers = TRUE, consider_isobars = TRUE, report_ambiguity_scores = TRUE)
```

## Evaluation signals

- Terms significant under standard ORA (Run_simple_ORA) but not under bootstrapping ORA (Run_bootstrap_ORA) — e.g., 'Glycerophosphoethanolamines', 'Fatty acyls', 'Linoleic acids and derivatives' — should carry ambiguity scores below 0.5
- The ORA query set must equal METASPACE annotations passing FDR <= 0.1 against HMDB for the compared conditions, with all detected HMDB annotations as the custom universe
- Bootstrapping output should include summarized enrichment results plus reported ambiguity scores (report_ambiguity_scores = TRUE)
- Running with consider_isomers = FALSE and consider_isobars = FALSE must reproduce the no-bootstrapping Run_simple_ORA path
- The comparison table should list per-term significance under both methods side by side with per-metabolite/term ambiguity scores

## Limitations

- Reliance on MS1 data in imaging MS prevents resolving isomeric and isobaric molecules, which is the ambiguity the bootstrapping propagates rather than removes
- Terms with low ambiguity scores (<0.5) that are significant under standard ORA may be false positives missed without bootstrapping
- The LION ontology cannot be used with or without consideration of isomeric/isobaric ambiguity since its terms are not all mapped to molecule names or sum formulas

## Evidence

- [abstract] S2IsoMEr R package and a METASPACE web app use bootstrapping to propagate isomeric/isobaric ambiguities into enrichment analysis: [evidence span withheld — anchor status: not_found]
- [methods] FDR_level = 0.1 DB = ("HMDB", "v4"): "FDR_level = 0.1
DB = ("HMDB", "v4")"
- [readme] We apply iterative random sampling (bootstrapping), perform enrichment analysis for each iteration, and report summarized results: [evidence span withheld — anchor status: external_doc]
- [results] 'Glycerophosphoethanolamines', 'Fatty acyls', and 'Linoleic acids and derivatives' were significantly enriched under standard ORA but not under the bootstrapping-based approach, likely because their ambiguity scores were low (<0.5): [evidence span withheld — anchor status: not_found]
- [readme] Except for the LION ontology, each term is mapped to either molecule names or sum formulas, allowing enrichment analysis to be performed with or without consideration of isomeric/isobaric ambiguity.: [evidence span withheld — anchor status: external_doc]

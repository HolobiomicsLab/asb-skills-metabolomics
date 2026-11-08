---
name: cross-strain-metabolic-distribution-comparison
description: Use when mass spectrometry imaging datasets from multiple mouse strains
  have been annotated per organ and you need to test whether the organ-specific metabolite
  distribution pattern is conserved across strains.
when_to_use_negative:
- When the analysis goal is discovering differential metabolites or pathway perturbations
  rather than verifying spatial distribution conservation — use differential expression/pathway
  skills instead.
- When input data are raw MSI peak lists that have not yet been annotated with organ-specific
  databases via MSIannotator; this skill operates on already-annotated metabolite
  sets.
- When only a single organ or a single mouse strain is available, since the core claim
  (organ-specificity spanning strains) requires multi-organ, multi-strain sets.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0153
tools:
- name: MSIannotator
  role: Annotates MSI features with organ-specific metabolite databases, producing
    the metabolite/adduct lists that feed the cross-strain set comparisons.
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Executes set operations (intersection, difference) on organ metabolite membership
    sets and exports the organ-specific distribution summary table.
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: MATLAB script implementing the annotation step (10 ppm default m/z tolerance)
    whose output Annotator files are the substrate for cross-strain comparison.
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:30d085ce8f2df54329305287bf911f29306e7fcc7db515d4b7cfb541ff95360a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-strain-metabolic-distribution-comparison

## Summary

Verifies that organ-specific metabolite distributions detected by AFADESI-MSI hold across different mouse strains, using set operations over organ-partitioned metabolite databases to compare per-organ metabolite sets between strains.

## When to use

Use when mass spectrometry imaging datasets from multiple mouse strains have been annotated per organ and you need to test whether the organ-specific metabolite distribution pattern is conserved across strains.

## When NOT to use

- When the analysis goal is discovering differential metabolites or pathway perturbations rather than verifying spatial distribution conservation — use differential expression/pathway skills instead.
- When input data are raw MSI peak lists that have not yet been annotated with organ-specific databases via MSIannotator; this skill operates on already-annotated metabolite sets.
- When only a single organ or a single mouse strain is available, since the core claim (organ-specificity spanning strains) requires multi-organ, multi-strain sets.

## Inputs

- Organ specific metabolite database.xlsx with sheets Sheet3–Sheet11 (organ-specific metabolite lists for brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum)
- Annotated MSI feature lists per organ and ionization mode (positive: 1298 metabolites; negative: 515 metabolites)
- Metabolite sets from at least two mouse strains

## Outputs

- Per-organ unique/enriched metabolite sets with adduct ion forms
- Cross-strain comparison confirming conservation of organ-specific distribution
- Summary table (CSV) with organ, mode, metabolite, and adduct columns

## How to apply

Load the organ-partitioned metabolite database sheets (Sheet3–Sheet11 for brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum) from 'Organ specific metabolite database.xlsx' into MATLAB as string arrays, covering 1298 positive-mode and 515 negative-mode metabolites. Build metabolite membership sets per organ and ionization mode from the first-column metabolite names, using MATLAB set operations (setdiff, intersect) to derive per-organ enriched/unique metabolite sets while tracking adduct ion forms per metabolite. Compare the organ-specific metabolite sets derived for the two mouse strains to verify that the organ-specific distribution pattern is conserved; the key finding to reproduce is that metabolites are distributed across the body in an organ-specific manner, which even spanned different mouse strains. Export a summary table with organ, mode, metabolite, and adduct columns (e.g. organ_specific_distribution.csv) for downstream validation.

## Related tools

- **MSIannotator** (Annotates MSI features with organ-specific metabolite databases, producing the metabolite/adduct lists that feed the cross-strain set comparisons.) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Executes set operations (intersection, difference) on organ metabolite membership sets and exports the organ-specific distribution summary table.) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (MATLAB script implementing the annotation step (10 ppm default m/z tolerance) whose output Annotator files are the substrate for cross-strain comparison.) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
MATLAB: organSets = containers.Map(); for s = 3:11, organSets(keys{s}) = setdiff(db{:,1}, union across other organs); end; writetable(summaryTable, 'organ_specific_distribution.csv')
```

## Evaluation signals

- The exported summary table reproduces the finding that 'metabolites were distributed across the body in an organ-specific manner, which even spanned different mouse strains'.
- Per-organ metabolite counts derived from Sheet3–Sheet11 are consistent with the reported totals (1298 positive-mode, 515 negative-mode metabolites across eight organs and serum).
- Every metabolite in the output retains its adduct ion form as recorded in the database sheets, so adduct-specific identity is preserved across strain comparisons.
- The organ-specific sets derived independently for each strain overlap such that the distribution pattern is reproduced in both strains.
- MATLAB set operations yield non-empty unique sets for at least the brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum categories.

## Limitations

- The organ-specific distribution claim rests on the article's own 27,407-adduct-ion database; coverage is bounded by that database's content.
- Annotation tolerates a 10 ppm m/z error by default; tighter or looser tolerances require editing line 25 of MSIannotator.m, which changes the input sets and thus the comparison.
- No changelog is available for the source repository, so version drift in database sheets or script behaviour cannot be tracked.

## Evidence

- [abstract] metabolites were distributed across the body in an organ-specific manner, which even spanned different mouse strains: "metabolites were distributed across the body in an organ-specific manner, which even spanned different mouse strains"
- [readme] Organ specific metabolite database.xlsx - our established database, consisting of 1298 metabolites in positive mode (Sheet1), 515 metabolites in negative mode (Sheet2): "Organ specific metabolite database.xlsx - our established database, consisting of 1298 metabolites in positive mode (Sheet1), 515 metabolites in negative mode (Sheet2)"
- [readme] in the code, the default value of m/z error range is 10 ppm, and if necessary users can reset it by modifying the line 25 code: "in the code, the default value of m/z error range is 10 ppm, and if necessary users can reset it by modifying the line 25 code"
- [readme] Generate a new file (Annotator), consisting of annotated metabolite names and their adduct forms.: "Generate a new file (Annotator), consisting of annotated metabolite names and their adduct forms."
- [discussion] _No changelog found._: "_No changelog found._"

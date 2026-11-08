---
name: jaccard-ordination-health-status
description: Use when MASST matches of synthesized compounds have been joined to ReDU
  sample metadata (biospecimen/tissue, health phenotype) and you need to test whether
  sample groups cluster by health status or tissue based on shared detected conjugates.
when_to_use_negative:
- The matches carry no ReDU metadata (biospecimen/health phenotype), so no phenotype
  grouping can be formed for ordination
- The question requires quantitative abundance differences rather than presence/absence
  of shared conjugates — Jaccard discards intensity information
- Input is raw LC-MS/MS data rather than already-curated MASST match results; run
  MASST matching first
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MASST
  role: Provides the public-dataset spectral matches for synthesized conjugated bile
    acids whose sample metadata (ReDU biospecimen/health phenotype) feed the Jaccard/ordination
    analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:0556a2d2c7762a2e7ac63f3ba056901a3aff8ffb5f963880e398575d59ff971a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# jaccard-ordination-health-status

## Summary

Computes Jaccard distances between samples/groups based on shared bile acid conjugate MASST matches and runs ordination to reveal clustering by health status (e.g. IBD vs healthy), as used to aggregate ReDU-metadata-annotated public metabolomics matches in reverse metabolomics.

## When to use

Use when MASST matches of synthesized compounds have been joined to ReDU sample metadata (biospecimen/tissue, health phenotype) and you need to test whether sample groups cluster by health status or tissue based on shared detected conjugates.

## When NOT to use

- The matches carry no ReDU metadata (biospecimen/health phenotype), so no phenotype grouping can be formed for ordination
- The question requires quantitative abundance differences rather than presence/absence of shared conjugates — Jaccard discards intensity information
- Input is raw LC-MS/MS data rather than already-curated MASST match results; run MASST matching first

## Inputs

- Filtered MASST match results with matched public spectra joined to ReDU sample metadata (biospecimen/tissue and health phenotype per match)
- Bile acid conjugate identity per match (e.g. Glu-, Ile/Leu-, Phe-, Trp-conjugated cholic acid)

## Outputs

- Tissue-by-conjugate and phenotype-by-conjugate spectral-count tables (count_table.csv)
- Jaccard distance matrix between samples/groups based on shared bile acid conjugates
- Ordination plot showing clustering of samples by health status (Fig. 2c,d style tissue-distribution and IBD-vs-healthy patterns)

## How to apply

Load the filtered MASST match results with the ReDU sample metadata (biospecimen/tissue and health phenotype) attached to each matched public spectrum. First tabulate matched spectral counts per tissue-conjugate and phenotype-conjugate pair to build the count tables (Fig. 2c,d of the reverse metabolomics study). Then, treating each sample or metadata group by its set of shared bile acid conjugates (e.g. cholic acid conjugated to Glu, Ile/Leu, Phe, Thr, Trp, Tyr), compute pairwise Jaccard distances — presence/absence of conjugates, not abundance. Run ordination (e.g. PCoA/NMDS) on the Jaccard distance matrix under the health-status grouping and check whether IBD and healthy samples separate. Write the aggregated tables to count_table.csv for validation against the reported patterns.

## Related tools

- **MASST** (Provides the public-dataset spectral matches for synthesized conjugated bile acids whose sample metadata (ReDU biospecimen/health phenotype) feed the Jaccard/ordination analysis)

## Examples

```
from scipy.spatial.distance import pdist; D = squareform(pdist(presence_absence_df.T, metric='jaccard'))  # then PCoA on D grouped by ReDU health phenotype
```

## Evaluation signals

- Aggregated tissue-by-conjugate table shows conjugated bile acids detected mainly in fecal material and intestinal tissue, rarely blood/liver (Fig. 2c pattern)
- Phenotype-by-conjugate table shows Glu, Ile/Leu, Phe and Trp conjugates more frequent in IBD samples than healthy (Fig. 2d pattern)
- Ordination on the Jaccard distance matrix reproduces clustering of samples by health status (IBD vs healthy)
- count_table.csv contains both tissue-conjugate and phenotype-conjugate count tables with non-zero entries for IBD-associated conjugates
- Validation via expert review against the reported Fig. 2c,d spectral-count patterns, including increased cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr in Crohn's disease cohorts

## Limitations

- MASST search coverage (1.2 billion public spectra) does not encompass all molecules, limiting re-analysis of public data in the discovery phase
- Presence/absence Jaccard matching cannot exclude that different isomers are represented by the data without isolation and NMR or X-ray structural analyses
- Associations rely on deposited public data with metadata; increased public data deposition, particularly with metadata, is needed for broader coverage
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds

## Evidence

- [results] Aggregate bile acid MASST matches by ReDU metadata shows conjugated bile acids detected mainly in fecal material and intestinal tissue: [evidence span withheld — anchor status: not_found]
- [methods] Jaccard distances and ordination workflow step: [evidence span withheld — anchor status: not_found]
- [abstract] Disease association finding from abstract: "Using repository-scale analysis 1,2 , we discovered that some conjugated bile acids are associated with inflammatory bowel disease (IBD)."
- [discussion] Limitation on isomer ambiguity: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [abstract] Validation finding from abstract: "Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease."

---
name: gene-set-collection-curation
description: Use when reproducing pathway-level gene-expression analyses (PLAGE, GSEA)
  that depend on a predefined gene-set collection whose memberships must match a historical
  KEGG/Biocarta version.
when_to_use_negative:
- When gene-sets must be defined de novo from custom biology rather than pulled from
  KEGG or Biocarta
- When an already-pinned, article-matched gene-set version is supplied — re-curation
  from live KEGG/Biocarta would break comparability with the reported ES and p-values
- When the analysis targets a single pathway of interest rather than the full ~400-pathway
  collection used for permutation-based significance
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program that archives and distributes the exact KEGG/Biocarta pathway
    versions used in the article for downstream SVD-based pathway analysis
  repo: http://dulci.biostat.edu/pathways
- name: KEGG
  role: Source database for pathway gene memberships in the ~400-pathway collection
- name: Biocarta
  role: Source database for pathway and complex gene memberships in the ~400-pathway
    collection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:e2ada0279a1f4620970ca351de8e66cc537e6c0f8591ccd005c820e30ec74f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biocarta
  - KEGG
  - PLAGE
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-set-collection-curation

## Summary

Assembles and pins the exact gene-set collection used for pathway-level expression analysis — here ~400 pathways and complexes from KEGG and Biocarta — so that downstream PLAGE or GSEA enrichment scores are reproducible despite continuous evolution of those pathway databases.

## When to use

Use when reproducing pathway-level gene-expression analyses (PLAGE, GSEA) that depend on a predefined gene-set collection whose memberships must match a historical KEGG/Biocarta version.

## When NOT to use

- When gene-sets must be defined de novo from custom biology rather than pulled from KEGG or Biocarta
- When an already-pinned, article-matched gene-set version is supplied — re-curation from live KEGG/Biocarta would break comparability with the reported ES and p-values
- When the analysis targets a single pathway of interest rather than the full ~400-pathway collection used for permutation-based significance

## Inputs

- KEGG pathway gene-membership listings
- Biocarta pathway/complex gene-membership listings
- Target scope of ~400 pathways and complexes as used in the paper

## Outputs

- Version-pinned gene-set collection (~400 KEGG and Biocarta pathway gene-sets) suitable for PLAGE/GSEA input
- Provenance record mapping each gene-set to its source database and version

## How to apply

Assemble the target collection by pulling gene memberships for each pathway and complex listed on the Biocarta and KEGG websites, targeting the paper's scale of about 400 gene-sets. Record which version of each source database you used, because the KEGG and Biocarta pathways expand and evolve over time and the article itself archived the exact versions at the PLAGE website rather than in the text. Do not re-download memberships from live databases without pinning, since a changed membership list changes downstream enrichment scores and p-values. Validate the curated collection by running it through the downstream analysis — e.g. signal-to-noise-ranked GSEA with 1000 permutations on the DM2 vs NGT comparison should yield the KEGG ribosome set as the top ES (278, p = 0.073) and oxidative phosphorylation next (261, p = 0.092).

## Related tools

- **PLAGE** (Web program that archives and distributes the exact KEGG/Biocarta pathway versions used in the article for downstream SVD-based pathway analysis) — http://dulci.biostat.edu/pathways
- **KEGG** (Source database for pathway gene memberships in the ~400-pathway collection)
- **Biocarta** (Source database for pathway and complex gene memberships in the ~400-pathway collection)

## Evaluation signals

- The curated collection contains about 400 pathways and complexes drawn from KEGG and Biocarta, matching the paper's scope
- Database versions are recorded; the article's versions are identifiable via the PLAGE website archive
- Downstream reproduction check: DM2 vs NGT GSEA gives KEGG ribosome ES 278 (p = 0.073, 1000 permutations) and oxidative phosphorylation ES 261 (p = 0.092)
- Ribosome gene-set ranks highest in ES in 3 of the 4 comparisons, with significance only in former vs never smokers (ES 304, p = 0.047)
- Reported ES and p-values match within permutation-sampling variability across all four comparisons

## Limitations

- KEGG and Biocarta pathways continuously expand and evolve, so unpinned re-curation cannot guarantee exact reproduction
- No version numbers or accessions are stated in the article; the versions used are only available at the PLAGE website
- Per-gene-set size filters or gene-mapping criteria are not specified, so collection assembly relies on the archived versions for fidelity

## Evidence

- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website"
- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"

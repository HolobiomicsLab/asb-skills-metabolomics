---
name: levenshtein-fuzzy-name-matching
description: Use when a metabolomics feature table has free-text or non-standard metabolite
  column names that must be converted to standard HMDB IDs before pathway mapping,
  and exact/synonym lookups leave names unmapped.
when_to_use_negative:
- Input metabolites are already represented by standard HMDB/PubChem/METLIN IDs —
  the name-to-ID standardization step is unnecessary
- The goal is downstream pathway scoring (lilikoi.get_pd_scores, lilikoi.select_pathways)
  rather than metabolite name standardization
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lilikoi
  role: R package providing lilikoi.metab_to_pathway, which chains exact matching,
    synonym lookup, and Levenshtein-distance fuzzy matching to map metabolite names
    to HMDB IDs and pathways
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Runtime environment for the lilikoi package and the annotation databases (data.hmdb,
    data.metaboAnalyst) it ships with
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:99e05e19f9f83a2887d50d5231536c76d843063bece5cc4592761f06acbbb25d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - lilikoi
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Levenshtein fuzzy name matching

## Summary

A metabolite-name standardization step from Lilikoi's feature mapping module in which user-supplied metabolite names are matched to standard HMDB IDs, with Levenshtein edit distance used as a fuzzy-string similarity fallback after exact and synonym matching.

## When to use

Use when a metabolomics feature table has free-text or non-standard metabolite column names that must be converted to standard HMDB IDs before pathway mapping, and exact/synonym lookups leave names unmapped.

## When NOT to use

- Input metabolites are already represented by standard HMDB/PubChem/METLIN IDs — the name-to-ID standardization step is unnecessary
- The goal is downstream pathway scoring (lilikoi.get_pd_scores, lilikoi.select_pathways) rather than metabolite name standardization

## Inputs

- Character vector of user-provided metabolite names (e.g. colnames(metaboliteMeasurements)[-1])
- Lilikoi's bundled annotation databases: data.hmdb, data.metaboAnalyst (from HMDB, SMPDB, and MetaboAnalyst)
- Input CSV such as plasma_breast_cancer.csv

## Outputs

- metabolitePathwayTable with each input metabolite name, matched standard HMDB ID, and assigned pathway(s)
- CSV file such as metabolite_hmdb_mapping.csv (one row per input metabolite)
- Mapping rate (fraction of input metabolites assigned a standard HMDB ID)

## How to apply

Load the user's metabolite names (e.g. colnames of the measurements matrix, as in Lilikoi's plasma_breast_cancer.csv example) and run lilikoi.metab_to_pathway(metaboliteNames, 'name'). The function first attempts an exact match against HMDB entries bundled with the package (data.hmdb), falls back to a synonym table (data.metaboAnalyst), and finally applies Levenshtein edit distance as the string-similarity measure for any remaining unmatched names. Write the resulting metabolitePathwayTable (input name, matched HMDB ID, assigned pathway(s)) to a CSV. Compute the mapping rate as the fraction of inputs assigned a standard HMDB ID; for Lilikoi's plasma dataset the reported rate is 182 of 227 metabolites (~80%), with the ~20% residual attributed in part to the Levenshtein-distance similarity measure.

## Related tools

- **lilikoi** (R package providing lilikoi.metab_to_pathway, which chains exact matching, synonym lookup, and Levenshtein-distance fuzzy matching to map metabolite names to HMDB IDs and pathways) — https://github.com/lanagarmire/lilikoi
- **R** (Runtime environment for the lilikoi package and the annotation databases (data.hmdb, data.metaboAnalyst) it ships with) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")
```

## Evaluation signals

- Output CSV exists, is well-formed, and contains one row per input metabolite
- Each mapped row carries a valid standard HMDB ID and at least one assigned pathway
- Mapping rate is consistent with the paper's reported rate for the plasma dataset (182 of 227 metabolites, ~80%), accepting that ~20% may remain unmapped
- Unmapped names should be inspected for near-miss spellings, since the Levenshtein fallback may both recover and mis-assign such names

## Limitations

- A small percentage (~20%) of metabolites still cannot be mapped to the standard names in databases
- The Levenshtein edit-distance similarity measure is identified by the authors as a possible cause of metabolite name mismatching
- Lilikoi ships with HMDB, SMPDB, and MetaboAnalyst data as of the publication date; refreshing requires lilikoi.update_database(), which downloads >5GB and can take over 20 minutes

## Evidence

- [abstract] feature mapping module: standardizes metabolite names and maps them to pathways: "feature mapping module, which standardizes the metabolite names provided by users and maps them to pathways"
- [results] standardization as first pipeline step: "Lilikoi first transforms the metabolite names to standard IDs"
- [results] reported plasma mapping rate: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [discussion] Levenshtein distance as mismatch cause: "One possible reason for this mismatching is that we used the Levenshtein distance as a measure of the similarity between the user's query metabolites"
- [discussion] unmapped residual: "a small percentage (20%) of the metabolites still cannot be mapped to the standard names in databases"
- [methods] core invocation: "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"

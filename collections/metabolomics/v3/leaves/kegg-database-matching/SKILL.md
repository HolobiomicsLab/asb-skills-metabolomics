---
name: kegg-database-matching
description: Use when starting an mWISE annotation run on unannotated LC-MS peak lists
  (m/z features) and you need KEGG compound candidates before applying the cluster-based
  filter and diffusion scoring.
when_to_use_negative:
- Annotation targets databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) — mWISE
  only offers KEGG data for annotation
- Input is already an annotated compound list or post-filter peak set; re-matching
  would be redundant before clustering/diffusion
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3346
tools:
- name: mWISE
  role: Package whose matching stage performs KEGG database matching of LC-MS peaks
  repo: github.com/b2slab/mWISE
- name: Genomic Ranges
  role: R package used by the matching stage to rapidly match LC-MS peaks to KEGG
    considering adducts and fragments
- name: R
  role: Runtime environment for mWISE and its matching stage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:1838ec4b54edabb96c1041f07b2295e4c5be177da7db8f3437a7252a12f265a5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3346
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# kegg-database-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches LC-MS peak m/z values against the KEGG database considering adducts and fragments, producing candidate compound lists that feed mWISE's clustering, filtering and diffusion prioritization stages.

## When to use

Use when starting an mWISE annotation run on unannotated LC-MS peak lists (m/z features) and you need KEGG compound candidates before applying the cluster-based filter and diffusion scoring.

## When NOT to use

- Annotation targets databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) — mWISE only offers KEGG data for annotation
- Input is already an annotated compound list or post-filter peak set; re-matching would be redundant before clustering/diffusion

## Inputs

- Filtered LC-MS peak list (features with m/z values; features without signal removed, 80% rule applied to zero-intensity cases)
- KEGG database compound reference
- Set of adducts and fragments considered for matching

## Outputs

- Per-peak list of matched KEGG compound candidates
- Matched peak set feeding downstream featuresClustering, clusterBased.filter and diffusion stages

## How to apply

Run mWISE's matching stage, which uses the GenomicRanges R package to rapidly match every LC-MS peak to KEGG considering a set of adducts and fragments. Before matching, filter the input peak list by removing LC-MS features without signal and applying the 80% rule where intensity equals 0. The matched peaks carry their candidate KEGG compounds into the subsequent featuresClustering, clusterBased.filter and set.diffusion (DiffuStats) stages. Because mWISE only offers KEGG data for annotation, this matching step is KEGG-specific; other databases (HMDB, Lipid Maps, T3DB) are handled by alternative tools like xMSannotator, not by this matching stage.

## Related tools

- **mWISE** (Package whose matching stage performs KEGG database matching of LC-MS peaks) — github.com/b2slab/mWISE
- **Genomic Ranges** (R package used by the matching stage to rapidly match LC-MS peaks to KEGG considering adducts and fragments)
- **R** (Runtime environment for mWISE and its matching stage)

## Examples

```
Rscript -e 'library(mWISE); peaks <- readRDS("filtered_peaks.rds"); matched <- matchingStage(peaks, adducts = defaultAdducts(), fragments = defaultFragments())'
```

## Evaluation signals

- Each LC-MS peak in the filtered input has been tested against KEGG with the configured adducts and fragments (no peaks silently dropped from the matching universe)
- Downstream featuresClustering receives a peak list with matched KEGG candidates; unmatched-only peaks can be flagged for QC
- Sensitivity of the matching stage can be cross-checked against reference assays (e.g. mWISE matching sensitivity for Assay 1 was 0.92, 144 TP of 157 references)
- The 80% rule and zero-signal removal were applied before matching, so no raw unfiltered peak list was passed in
- Computation stays in the fast per-dataset range reported for the FELLA-raw pipeline (15.70–53.38 s), far below xMSannotator's 106.34–318.04 s

## Limitations

- mWISE only offers KEGG data for annotation — an important limitation with respect to xMSannotator that should be addressed in future versions
- Matching quality depends on the supplied adduct/fragment set; features outside those assumptions yield no or wrong candidates
- mWISE lacks support for databases other than KEGG, so multi-database annotation requires a different tool

## Evidence

- [methods] matching uses GenomicRanges against KEGG: [evidence span withheld — anchor status: not_found]

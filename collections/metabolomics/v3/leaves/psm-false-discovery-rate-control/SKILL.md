---
name: psm-false-discovery-rate-control
description: Use when PSMs from a spectral library or database search (e.g., ANN-SoLo,
  MS-GF+) must be filtered to a 1% FDR before counting identifications, propagating
  peptide labels, or building training resources.
when_to_use_negative:
- Identifications are already provided with FDR-controlled q-values in the source
  dataset (e.g., MassIVE-KB metadata TSV), so re-filtering would be redundant
- The downstream analysis requires sensitivity over specificity, since 1% FDR deliberately
  discards lower-confidence matches
- The search results are de novo sequences or spectral embeddings (e.g., GLEAMS 32-dimensional
  vectors) that lack target-decoy PSM scores
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Open modification spectral library search whose results are filtered at 1%
    FDR to annotate GLEAMS cluster medoids
- name: MS-GF+
  role: Database search engine producing the 185 million PSMs at 1% FDR from which
    GLEAMS training identifications were drawn
- name: GLEAMS
  role: Embedding and clustering tool whose medoid spectra are searched and whose
    cluster labels are used to propagate 1% FDR-controlled PSM identifications
  repo: https://github.com/bittremieux/GLEAMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:5a2c6d0e18bdbb2630976941d4ad0176b8a8b61a5963443e5df01ac61ada879c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - MS-GF+
  - Python
  - conda
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# psm-false-discovery-rate-control

## Summary

Applies 1% false discovery rate (FDR) control to peptide-spectrum matches (PSMs) from database and spectral library searching so that only confident identifications are counted, propagated, or used for downstream annotation. In this workflow, FDR-controlled PSMs underpin both GLEAMS training data compilation and validation of ANN-SoLo open modification search results.

## When to use

Use when PSMs from a spectral library or database search (e.g., ANN-SoLo, MS-GF+) must be filtered to a 1% FDR before counting identifications, propagating peptide labels, or building training resources.

## When NOT to use

- Identifications are already provided with FDR-controlled q-values in the source dataset (e.g., MassIVE-KB metadata TSV), so re-filtering would be redundant
- The downstream analysis requires sensitivity over specificity, since 1% FDR deliberately discards lower-confidence matches
- The search results are de novo sequences or spectral embeddings (e.g., GLEAMS 32-dimensional vectors) that lack target-decoy PSM scores

## Inputs

- Search-engine output (e.g., mzTab PSM tables from MS-GF+ search tasks on MassIVE, or ANN-SoLo open modification search results)
- Target and decoy peptide-spectrum matches for FDR estimation
- MassIVE-KB (v1) Human HCD spectral library search results

## Outputs

- Filtered PSM list at 1% FDR (e.g., 185 million MS-GF+ PSMs; ~5.3 million ANN-SoLo PSMs)
- Per-precursor PSM subsets (maximum top 100 PSMs per unique precursor)
- Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids

## How to apply

Run the search engine (e.g., MS-GF+ for database search or ANN-SoLo v0.3.3 open modification search against the MassIVE-KB (v1) Human HCD spectral library), then apply the 1% FDR cutoff — typically via target-decoy competition — to retain only confident peptide-spectrum matches. In this paper's pipeline, all 185 million PSMs obtained with MS-GF+ were controlled at 1% FDR, from which the top 100 PSMs per precursor for 2.1 million unique precursors formed the 30 million PSMs used to train GLEAMS. After open modification searching of GLEAMS cluster medoids, ANN-SoLo search results were filtered at 1% FDR to yield ~5.3 million confident PSMs whose peptide labels could be propagated to ~44 million cluster-member spectra. Validation should confirm that the FDR is held at 1% and that resulting PSM counts match the reported figures.

## Related tools

- **ANN-SoLo** (Open modification spectral library search whose results are filtered at 1% FDR to annotate GLEAMS cluster medoids)
- **MS-GF+** (Database search engine producing the 185 million PSMs at 1% FDR from which GLEAMS training identifications were drawn)
- **GLEAMS** (Embedding and clustering tool whose medoid spectra are searched and whose cluster labels are used to propagate 1% FDR-controlled PSM identifications) — https://github.com/bittremieux/GLEAMS

## Evaluation signals

- The applied FDR threshold is explicitly 1% and target-decoy estimation is documented for the search results
- Direct PSM count matches the reported ~5.3 million identified spectra from ANN-SoLo searching of 45 million cluster medoids
- Propagated PSM count matches the reported ~44 million additional PSMs from propagating medoid peptide labels to cluster members
- Training-resource PSMs are a subset of the 185 million MS-GF+ PSMs at 1% FDR, capped at the top 100 PSMs for 2.1 million unique precursors
- Counts are confirmed consistent via expert review of the direct and propagated PSM totals

## Limitations

- FDR control relies on the search engine's target-decoy scoring; results depend on the quality of the reference library (MassIVE-KB v1 Human HCD spectral library)
- The 30 million training PSMs are only a subset (top 100 per precursor) of all 1% FDR identifications, so total identified spectra exceed what is used downstream
- FDR is an estimated, not a guaranteed, error rate and is not transferable between search engines or datasets without recalibration

## Evidence

- [methods] All 185 million PSMs at 1% FDR obtained using MS-GF+: "All 185 million PSMs at 1% FDR obtained using MS-GF+"
- [supplementary] Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids.: "Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids."
- [readme] These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors).: "These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors)."
- [readme] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"

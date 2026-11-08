---
name: tsne-embedding-of-eic-features
description: Use when raw mzXML spectra from HPTLC-HRMS tracks have been converted
  into per-ion EIC features and a low-dimensional embedding is needed before clusterising
  ions of interest versus background.
when_to_use_negative:
- Data are LC-MS/MS or direct-infusion spectra that lack HPTLC planar-chromatography
  track structure — the embedding targets EIC features from HPTLC-HRMS tracks.
- A per-ion cluster label table already exists and only plotting/visualisation of
  that clustering is needed — t-SNE re-embedding would be redundant.
- The input is raw mzXML spectra with no EIC feature extraction performed yet; build
  the EIC features before embedding.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Rtsne
  role: Computes the t-SNE embedding of the EIC features derived from HPTLC-HRMS spectra
- name: tsne
  role: Alternative R implementation of t-SNE for embedding the EIC features when
    Rtsne is not available
- name: readMzXmlData
  role: Reads the mzXML spectral files from which the EIC features to be embedded
    are built
- name: eicCluster
  role: Encapsulating R/Shiny package whose workflow uses the t-SNE embedding to clusterise
    EIC features of HPTLC-HRMS data
  repo: github.com/dimitrif/eicCluster
- name: shiny
  role: Interactive app layer through which the embedding-plus-clusterisation analysis
    is performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.chroma.2018.09.050
    title: eicCluster software, an open-source in silico tool, and on-surface syntheses,
      an in situ concept, both exploited for signal highlighting in high-resolution
      mass spectrometry to ease structure elucidation in planar chromatography
  source_content_hash: sha256:ee59bb4da0d4978a7088abf7f451075838e8d001f54880c48af0d7b31bae67c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - RColorBrewer
  - Rtsne
  - eicCluster
  - readMzXmlData
  - shiny
  - tsne
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# tsne-embedding-of-eic-features

## Summary

Computes a t-SNE embedding of extracted ion chromatogram (EIC) features derived from HPTLC-HRMS data so that subsequent clustering can separate ions of interest from background ions.

## When to use

Use when raw mzXML spectra from HPTLC-HRMS tracks have been converted into per-ion EIC features and a low-dimensional embedding is needed before clusterising ions of interest versus background.

## When NOT to use

- Data are LC-MS/MS or direct-infusion spectra that lack HPTLC planar-chromatography track structure — the embedding targets EIC features from HPTLC-HRMS tracks.
- A per-ion cluster label table already exists and only plotting/visualisation of that clustering is needed — t-SNE re-embedding would be redundant.
- The input is raw mzXML spectra with no EIC feature extraction performed yet; build the EIC features before embedding.

## Inputs

- mzXML files of HPTLC-HRMS runs (raw spectral data with ion m/z and intensity over retention/track position)
- Extracted ion chromatogram (EIC) features, one per detected ion, extracted from the mzXML spectra

## Outputs

- Low-dimensional t-SNE embedding of the EIC features, one point per ion, suitable for clustering

## How to apply

First load the HPTLC-HRMS-derived mzXML spectral data (e.g. with the readMzXmlData package) and build EIC features for each detected ion. Compute a dimensionality-reduced embedding of those EIC features with the Rtsne package — the tsne package is an acceptable alternative implementation of the same embedding — reducing the high-dimensional EIC feature space so that ions with similar chromatographic behaviour become neighbours. The resulting embedding is consumed by the clusterisation step that partitions ions of interest from background ions, which is the core aim of eicCluster for HPTLC-HRMS data. Cluster labels assigned on the embedding should be reported per ion so a later filter step can write an assignment table of interest/background membership.

## Related tools

- **Rtsne** (Computes the t-SNE embedding of the EIC features derived from HPTLC-HRMS spectra)
- **tsne** (Alternative R implementation of t-SNE for embedding the EIC features when Rtsne is not available)
- **readMzXmlData** (Reads the mzXML spectral files from which the EIC features to be embedded are built)
- **eicCluster** (Encapsulating R/Shiny package whose workflow uses the t-SNE embedding to clusterise EIC features of HPTLC-HRMS data) — github.com/dimitrif/eicCluster
- **shiny** (Interactive app layer through which the embedding-plus-clusterisation analysis is performed)

## Examples

```
eicCluster::run_eicCluster()
```

## Evaluation signals

- The embedding output contains one point per ion/EIC feature from the mzXML input — point count matches the ion table.
- Downstream clustering on the embedding yields at least one cluster assigned as 'interesting' and one as 'background' on the HPTLC-HRMS data.
- Neighbourhood structure is coherent: ions with similar extracted ion chromatograms end up close in the embedding, as judged by expert review of the clusterisation.
- The embedding feeds directly into the run_eicCluster() Shiny workflow without shape/dimension errors.
- Re-running the embedding step on the same EIC features and same Rtsne settings reproduces consistent cluster assignments (subject to t-SNE stochasticity).

## Limitations

- t-SNE is stochastic: different runs (or the tsne vs Rtsne implementations) can give different embeddings, so cluster labels should be validated by expert review rather than assumed stable.
- The embedding is only meaningful for HPTLC-HRMS-derived EIC features; it does not substitute for the EIC feature extraction from raw mzXML spectra.
- Embedding quality depends on the upstream EIC feature construction — poor EICs will produce poorly separable clusters.

## Evidence

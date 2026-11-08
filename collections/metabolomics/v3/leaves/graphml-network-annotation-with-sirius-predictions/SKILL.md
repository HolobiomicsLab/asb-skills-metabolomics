---
name: graphml-network-annotation-with-sirius-predictions
description: Use when you have downloaded a GNPS FBMN/IIMN GraphML network and want
  to overlay SIRIUS/CSI:FingerID structural and formula predictions onto it for MSI
  level 2/3 annotation.
when_to_use_negative:
- The GNPS job has not yet finished and the GraphML network file has not been downloaded.
- You only want spectral-library (MSI level 2) annotations and no structural/formula
  predictions — the Spectral matching step alone suffices.
- Input is already a fully annotated feature matrix with SIRIUS assignments; re-integrating
  them into the network is redundant.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: 'Produces #1-ranked molecular formula and structural (CSI:FingerID) predictions
    per feature, which are merged into the GraphML network.'
- name: CSI:FingerID
  role: Generates the structure predictions that accompany the SIRIUS formulas for
    network annotation.
- name: UmetaFlow (Graph view / data-integration step)
  role: Implements the GraphML parsing and merging of SIRIUS predictions into the
    GNPS network for visualization.
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings used in the Jupyter notebook implementation to read and write
    the annotated GraphML and feature tables.
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: GNPS
  role: Source of the FBMN/IIMN GraphML network and the MS/MS library-match annotations
    that can also be merged into the feature matrix.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:8a58c3136da2b8ac806952b4cab03827b426affab44cabdc4dc6677392430771
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - CSI:FingerID
  - GNPS
  - OpenMS
  - Python
  - SIRIUS
  - UmetaFlow
  - UmetaFlow (Graph view / data-integration step)
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# GraphML network annotation with SIRIUS predictions

## Summary

Integrates #1-ranked SIRIUS/CSI:FingerID formula and structural predictions into the GNPS FBMN/IIMN GraphML network file so that MSI level 2/3 molecular annotations are visible directly on the network for visual inspection.

## When to use

Use when you have downloaded a GNPS FBMN/IIMN GraphML network and want to overlay SIRIUS/CSI:FingerID structural and formula predictions onto it for MSI level 2/3 annotation.

## When NOT to use

- The GNPS job has not yet finished and the GraphML network file has not been downloaded.
- You only want spectral-library (MSI level 2) annotations and no structural/formula predictions — the Spectral matching step alone suffices.
- Input is already a fully annotated feature matrix with SIRIUS assignments; re-integrating them into the network is redundant.

## Inputs

- GNPS FBMN/IIMN output GraphML network file (downloaded from the completed GNPS job)
- SIRIUS/CSI:FingerID #1-ranked formula and structure prediction results per consensus feature ID
- Optional: GNPS MS/MS library matching annotations (MGF/MSP) for feature-table annotation

## Outputs

- Annotated GraphML network containing SIRIUS-derived formula/structure assignments (MSI level 3) viewable in Cytoscape/GNPS viewer
- Optionally, a feature matrix annotated with GNPS MS/MS library matches (MSI level 2)

## How to apply

Run SIRIUS 5 with CSI:FingerID and CANOPUS on the consensus feature map's MS2 spectra, then take the #1 ranked formula and structure predictions per feature ID. After the GNPS FBMN or IIMN job completes and the resulting network is downloaded, parse the GNPS-generated GraphML file, merge the SIRIUS predictions into it by matching on feature IDs, and write the annotated GraphML back out. This is the final 'Graph view' step of UmetaFlow (step 7 in the pyOpenMS notebooks, step 7 in the Snakemake DAG). The rationale is that GNPS networks alone carry only MS/MS library hits, so integrating SIRIUS predictions lets you visually inspect MSI level 3 formula/structure assignments directly on the network without leaving the Cytoscape/GNPS viewer. Optionally also annotate the feature matrix with GNPS MS/MS library matching hits (MSI level 2) in the same pass.

## Related tools

- **SIRIUS** (Produces #1-ranked molecular formula and structural (CSI:FingerID) predictions per feature, which are merged into the GraphML network.)
- **CSI:FingerID** (Generates the structure predictions that accompany the SIRIUS formulas for network annotation.)
- **UmetaFlow (Graph view / data-integration step)** (Implements the GraphML parsing and merging of SIRIUS predictions into the GNPS network for visualization.) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings used in the Jupyter notebook implementation to read and write the annotated GraphML and feature tables.) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **GNPS** (Source of the FBMN/IIMN GraphML network and the MS/MS library-match annotations that can also be merged into the feature matrix.)

## Examples

```
snakemake --use-conda results/GraphML/SIRIUS_annotated_graphml.graphml
```

## Evaluation signals

- The output GraphML parses successfully and each node corresponding to a consensus feature carries a formula and/or structure attribute derived from SIRIUS/CSI:FingerID.
- Feature IDs in the annotated GraphML match the IDs in the SIRIUS prediction table (no orphan nodes left un-annotated when a prediction exists).
- Annotated assignments can be verified as MSI level 3 (formula/structure from SIRIUS) versus MSI level 2 (GNPS library match), consistent with UmetaFlow's annotation levels.
- The network opens in Cytoscape and displays the structural/formula annotations without altering GNPS network topology (edges and groupings unchanged).
- The feature matrix, when optionally annotated with GNPS library matches, contains hits with scores above the 60 threshold only.

## Limitations

- SIRIUS predictions are MSI level 3 — they are computational predictions, not spectral-library confirmations; only GNPS MS/MS library matches (score > 60) reach MSI level 2.
- The workflow assumes a completed GNPS FBMN/IIMN job; if the network is regenerated with different parameters, the annotation must be redone.
- Only #1-ranked SIRIUS/CSI:FingerID predictions are integrated — lower-ranked candidates are discarded.

## Evidence

- [intro] structural and formula predictions via SIRIUS and CSI:FingerID: "structural and formula predictions via SIRIUS [18] and CSI:FingerID [19]"
- [readme] Graph view step of UmetaFlow integrates SIRIUS predictions to the network (GraphML): "**Graph view**: Integrate SIRIUS predictions to the network (GraphML) and GNPS library annotations to the feature matrix - MSI level 2 (optional)."
- [readme] Data integration notebook integrates #1 ranked SIRIUS and CSI:FingerID predictions to the graphml file from GNPS FBMN: [evidence span withheld — anchor status: external_doc]
- [readme] Annotate feature tables with SIRIUS/CSI:FingerID #1 ranked predictions (MSI level 3) and spectral matches (MSI level 2): [evidence span withheld — anchor status: external_doc]
- [readme] Spectral matching annotations with score above 60 are MSI level 2: "annotation of the feature matrix with matches that have a score above 60 (MSI level 2)"

---
name: disease-network-visualization
description: Use when a deep-learning metabolomics run has produced ranked m/z contribution
  scores (e.g. ensemble_RISE.npy) and the goal is to connect those metabolic signals
  to proteins and pathways for disease interpretation.
when_to_use_negative:
- Input is already a curated feature/peak table rather than model-derived contribution
  scores over raw spectral m/z signals
- The aim is biomarker classification accuracy rather than biological network interpretation
  (use run_train/run_predict instead)
- No interaction database coverage (iRefIndex/HMDB/Recon2) exists for the metabolites
  of interest, so Steiner-tree prize collection cannot connect them
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3520
tools:
- name: DeepMSProfiler
  role: Produces the ensemble RISE contribution heatmap over raw LC-MS m/z signals
    used to rank and select high-contribution peaks
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: PIUMet
  role: Runs the prize-collecting Steiner Forest algorithm on annotated metabolite
    prizes over iRefIndex v13 with HMDB v3 and Recon2 to extract the disease-specific
    subnetwork
- name: MetaboAnalyst
  role: Performs joint pathway enrichment on the network metabolites/proteins to flag
    pathways at FDR < 0.05
- name: Python (>= 3.6)
  role: Implements peak ranking/filtering, annotation matching, and network/panel
    visualization scripting
  repo: https://github.com/yjdeng9/DeepMSProfiler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:a1a06d9481f8e84be9fca229ec928094ebc8c622791f5723e57f9bc8f18a6d12
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - DeepMSProfiler
  - MetaboAnalyst
  - PIUMet
  - Python (>= 3.6)
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# disease-network-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and visualize a disease-specific metabolite-protein network from high-contribution m/z peaks identified by an ensemble deep-learning model, then overlay pathway enrichment. This turns interpretable spectral features (RISE heatmap) into a biological network figure linking metabolites to hidden proteins.

## When to use

Use when a deep-learning metabolomics run has produced ranked m/z contribution scores (e.g. ensemble_RISE.npy) and the goal is to connect those metabolic signals to proteins and pathways for disease interpretation.

## When NOT to use

- Input is already a curated feature/peak table rather than model-derived contribution scores over raw spectral m/z signals
- The aim is biomarker classification accuracy rather than biological network interpretation (use run_train/run_predict instead)
- No interaction database coverage (iRefIndex/HMDB/Recon2) exists for the metabolites of interest, so Steiner-tree prize collection cannot connect them

## Inputs

- Ensemble RISE feature heatmap (ensemble_RISE.npy) from the DeepMSProfiler run_feature step
- Ranked m/z peak list with contribution scores (retained threshold > 0.70)
- HMDB v3 metabolite reference annotations (m/z and MS2 spectra matching)
- iRefIndex v13 protein-protein and protein-metabolite interaction network
- Recon2 metabolic network database

## Outputs

- Disease-specific metabolite-protein subnetwork as luad_metabolite_protein_network.tsv (e.g. 82 proteins and 121 metabolites)
- MetaboAnalyst joint pathway enrichment results (pathways passing FDR < 0.05)
- Network and pathway panel visualizations corresponding to Fig. 5g-l

## How to apply

Rank spectral peaks by their ensemble RISE contribution score and retain only peaks with contribution score > 0.70. Annotate the retained peaks by matching m/z values and MS2 spectra against HMDB v3 reference annotations. Run PIUMet, a prize-collecting Steiner Forest algorithm, using the annotated metabolites as prizes on the iRefIndex v13 protein-protein and protein-metabolite interaction network together with HMDB v3 and Recon2, to extract the disease-specific subnetwork (e.g. 82 proteins and 121 metabolites for LUAD) and save it as luad_metabolite_protein_network.tsv. Submit the network metabolites/proteins to MetaboAnalyst joint pathway enrichment, retaining pathways with FDR < 0.05, then render the network and enriched pathways as a panel set (analogous to Fig. 5g-l).

## Related tools

- **DeepMSProfiler** (Produces the ensemble RISE contribution heatmap over raw LC-MS m/z signals used to rank and select high-contribution peaks) — https://github.com/yjdeng9/DeepMSProfiler
- **PIUMet** (Runs the prize-collecting Steiner Forest algorithm on annotated metabolite prizes over iRefIndex v13 with HMDB v3 and Recon2 to extract the disease-specific subnetwork)
- **MetaboAnalyst** (Performs joint pathway enrichment on the network metabolites/proteins to flag pathways at FDR < 0.05)
- **Python (>= 3.6)** (Implements peak ranking/filtering, annotation matching, and network/panel visualization scripting) — https://github.com/yjdeng9/DeepMSProfiler

## Examples

```
run_feature(job_dir='DeepMSProfiler/example/out/jobs007'); show_feature(job_dir='DeepMSProfiler/example/out/jobs007',mode='ensemble')
```

## Evaluation signals

- Reconstructed network contains exactly 82 proteins and 121 metabolites for the LUAD case
- All reported enriched pathways satisfy FDR < 0.05
- Only peaks with ensemble RISE contribution score > 0.70 were carried into annotation and network inference
- Output network file luad_metabolite_protein_network.tsv exists with metabolite and protein node columns populated from HMDB/iRefIndex identifiers
- Heatmap input exists at the expected path ../jobs/jobs007/feature_results/ensemble_RISE.npy with documented shape (6, 341, 341) in the demo run

## Limitations

- Annotation is restricted to HMDB v3 reference metabolites, so unannotated metabolites enter only indirectly through PIUMet's hidden-node inference
- PIUMet is described as a protein-protein and protein-metabolite interaction tool; coverage depends on iRefIndex v13 and Recon2 database versions
- Contribution cutoff (0.70) and network size validation values are specific to the LUAD analysis and must be re-derived for other diseases

## Evidence

- [other] PIUMet constructs disease-related metabolite-protein networks from input m/z peaks using the prize-collecting Steiner Forest algorithm: [evidence span withheld — anchor status: not_found]
- [other] integrating iRefIndex (v13), HMDB (v3) and Recon2 databases: [evidence span withheld — anchor status: not_found]
- [other] rank spectral peaks by their contribution score, retaining only peaks with contribution score > 0.70: [evidence span withheld — anchor status: not_found]

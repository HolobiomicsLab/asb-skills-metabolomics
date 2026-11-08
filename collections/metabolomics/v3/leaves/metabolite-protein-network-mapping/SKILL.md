---
name: metabolite-protein-network-mapping
description: Use when a trained DeepMSProfiler ensemble has produced ensemble_RISE.npy
  saliency heatmaps and the goal is to identify the hidden metabolites and proteins
  connected to the high-contribution spectral peaks.
when_to_use_negative:
- Input is already a curated metabolite/protein list rather than a model-generated
  saliency heatmap — the >0.70 contribution-score filtering step has no meaning without
  ensemble_RISE.npy scores
- The analysis goal is disease classification accuracy rather than network interpretability
  — use run_predict with plot_auc/plot_cm instead
- No MS2 spectra or m/z reference annotations are available for peak-to-metabolite
  matching, since PIUMet requires annotated metabolites as Steiner Forest prizes
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3520
tools:
- name: DeepMSProfiler
  role: Produces the ensemble RISE saliency heatmap (ensemble_RISE.npy) whose high-contribution
    m/z peaks are ranked and filtered before network mapping
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: PIUMet
  role: Runs the prize-collecting Steiner Forest algorithm over iRefIndex v13, HMDB
    v3 and Recon2 to extract the disease-specific metabolite-protein subnetwork from
    annotated peak prizes
- name: MetaboAnalyst
  role: Performs joint pathway enrichment on the recovered network metabolites/proteins,
    retaining pathways with FDR < 0.05
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:73d8fb93b4b044f34505427d9df0d38d6cc06e446b5137a2d0682c6888d3d743
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
  - CUDA
  - DeepMSProfiler
  - Keras
  - Liquid chromatography mass spectrometry (LC-MS)
  - MetaboAnalyst
  - PIUMet
  - Python (>= 3.6)
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 24
---

# metabolite-protein-network-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps ensemble RISE-highlighted m/z peaks from DeepMSProfiler onto disease-related metabolite-protein interaction networks using PIUMet, then enriches the recovered subnetwork via MetaboAnalyst. This turns model saliency into interpretable biochemical pathways.

## When to use

Use when a trained DeepMSProfiler ensemble has produced ensemble_RISE.npy saliency heatmaps and the goal is to identify the hidden metabolites and proteins connected to the high-contribution spectral peaks.

## When NOT to use

- Input is already a curated metabolite/protein list rather than a model-generated saliency heatmap — the >0.70 contribution-score filtering step has no meaning without ensemble_RISE.npy scores
- The analysis goal is disease classification accuracy rather than network interpretability — use run_predict with plot_auc/plot_cm instead
- No MS2 spectra or m/z reference annotations are available for peak-to-metabolite matching, since PIUMet requires annotated metabolites as Steiner Forest prizes

## Inputs

- Ensemble RISE saliency heatmap (ensemble_RISE.npy) from DeepMSProfiler run_feature
- Ranked m/z peak list with contribution scores (threshold: score > 0.70)
- Metabolite reference annotations: HMDB v3 (m/z and MS2 spectral matching)
- Interaction databases: iRefIndex v13, HMDB v3, Recon2

## Outputs

- Disease-specific metabolite-protein network TSV (e.g., luad_metabolite_protein_network.tsv with 82 proteins and 121 metabolites for LUAD)
- MetaboAnalyst joint pathway enrichment results (pathways with FDR < 0.05)
- Network and pathway visualization panels (e.g., corresponding to Fig. 5g-l)

## How to apply

Load the ensemble RISE heatmap (e.g., feature_results/ensemble_RISE.npy) and rank spectral peaks by contribution score, retaining only peaks with contribution score > 0.70. Annotate the retained peaks by matching their m/z values and MS2 spectra against metabolite reference annotations (HMDB v3). Run PIUMet, the prize-collecting Steiner Forest algorithm, with the annotated metabolites as prizes on the iRefIndex v13 protein-protein and protein-metabolite interaction network, integrating HMDB v3 and Recon2 to link metabolic signals to hidden metabolites and proteins. Extract the resulting disease-specific subnetwork (e.g., the LUAD network of 82 proteins and 121 metabolites), save it as a TSV (e.g., luad_metabolite_protein_network.tsv), and submit its metabolites/proteins to MetaboAnalyst joint pathway enrichment retaining pathways with FDR < 0.05. The rationale is that the saliency heatmap localizes disease-relevant raw spectral regions without requiring complete metabolite annotation, and PIUMet fills in unannotated metabolites through network propagation.

## Related tools

- **DeepMSProfiler** (Produces the ensemble RISE saliency heatmap (ensemble_RISE.npy) whose high-contribution m/z peaks are ranked and filtered before network mapping) — https://github.com/yjdeng9/DeepMSProfiler
- **PIUMet** (Runs the prize-collecting Steiner Forest algorithm over iRefIndex v13, HMDB v3 and Recon2 to extract the disease-specific metabolite-protein subnetwork from annotated peak prizes)
- **MetaboAnalyst** (Performs joint pathway enrichment on the recovered network metabolites/proteins, retaining pathways with FDR < 0.05)

## Examples

```
python showFeature.py  # after: python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature; then feed high-contribution peaks (score>0.70) to PIUMet
```

## Evaluation signals

- Reconstructed network must contain the expected node counts (e.g., exactly 82 proteins and 121 metabolites for the LUAD network)
- All reported enriched pathways from MetaboAnalyst must satisfy FDR < 0.05
- Filter invariant: only peaks with contribution score > 0.70 from the RISE heatmap should enter annotation
- Pan-cancer check: network analysis across 23 CCLE cancer types should recover 14 shared metabolites and 3 shared proteins co-occurring in all networks (Fig. 6)
- Heatmap output must match the DeepMSProfiler log format: ensemble_RISE.npy saved under feature_results/ with shape (n_classes, 341, 341) in the demo

## Limitations

- Metabolite annotation depends on HMDB v3 reference matching of m/z and MS2 spectra; unannotated peaks cannot serve as Steiner Forest prizes directly (PIUMet instead infers them as hidden nodes)
- Pre-trained DeepMSProfiler model access requires contacting the author, so heatmap generation may need full retraining first
- Network conclusions are contingent on iRefIndex v13, HMDB v3 and Recon2 coverage; interactions absent from these databases will be missed
- The method targets raw LC-MS metabolomic data; results depend on the upstream ensemble's saliency quality (e.g., 10 repeated DenseNet121 training runs with adam, lr 1e-04)

## Evidence

- [full_text] PIUMet constructs disease-related metabolite-protein networks from input m/z peaks using the prize-collecting Steiner Forest algorithm: [evidence span withheld — anchor status: not_found]
- [full_text] Load the ensemble RISE feature heatmap and rank spectral peaks by contribution score: [evidence span withheld — anchor status: not_found]
- [full_text] Annotate the retained peaks by matching their m/z values and MS2 spectra against metabolite reference annotations (HMDB v3): [evidence span withheld — anchor status: not_found]

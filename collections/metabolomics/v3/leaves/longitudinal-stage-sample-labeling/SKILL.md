---
name: longitudinal-stage-sample-labeling
description: Use when an NMR-derived metabolite table spans multiple differentiation
  stages (hESC, EB, Rosette, hNPC, Neuron) and samples must be tagged with stage labels
  before normalization/PCA.
when_to_use_negative:
- The input is a single-population dataset with no longitudinal/staged design — there
  are no developmental stages to label and the skill adds no information.
- The table is already normalized and stage-labeled downstream output (e.g., normalized_metabolite_table.csv
  from a prior step) — relabeling risks duplicating or overwriting the metadata column.
- Sample population identity is unknown or cannot be verified against marker-based
  purity checks — labels would be guesses, corrupting stage-specific PCA interpretation.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the raw NMR metabolite table while retaining sample stage labels as
    metadata columns during preprocessing
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: 'Downstream consumer: interprets PCA scores (10 components, first three used)
    per labeled developmental stage'
  repo: github.com/jlichtarge/pcaGLASSO
- name: pcaglasso
  role: Overlays GLASSO graphs with stage-specific PCA scores to identify significant
    sub-networks per cell population
  repo: github.com/jlichtarge/pcaGLASSO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:90619e6b52c480c784e41bb42d2aef1cf35c2841eadc434484a5377f2c129484
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - pandas
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# longitudinal-stage-sample-labeling

## Summary

Assigns and carries consistent developmental-stage labels (hESC, EB, Rosette, hNPC, Neuron) on NMR metabolomics samples so that longitudinal, stage-specific analyses such as MetaboLINK's PCA-GLASSO can distinguish cell populations across the differentiation timeline.

## When to use

Use when an NMR-derived metabolite table spans multiple differentiation stages (hESC, EB, Rosette, hNPC, Neuron) and samples must be tagged with stage labels before normalization/PCA.

## When NOT to use

- The input is a single-population dataset with no longitudinal/staged design — there are no developmental stages to label and the skill adds no information.
- The table is already normalized and stage-labeled downstream output (e.g., normalized_metabolite_table.csv from a prior step) — relabeling risks duplicating or overwriting the metadata column.
- Sample population identity is unknown or cannot be verified against marker-based purity checks — labels would be guesses, corrupting stage-specific PCA interpretation.

## Inputs

- Raw metabolite concentration/intensity table from 1H-NMR peak integration (Supplementary Table S1; 90 metabolites) with sample identifiers
- Sample-to-population mapping from the differentiation protocol (hESC, EB, Rosette, hNPC, Neuron)
- Optional stage-purity evidence (flow cytometry marker results: CD24, ZO-1, Nestin, MAP2, vGLUT1)

## Outputs

- Stage-labeled metabolite table (e.g., normalized_metabolite_table.csv) with metabolites as rows/features, stage samples as columns, and stage labels preserved as metadata
- Consistent stage vocabulary (hESC, EB, Rosette, hNPC, Neuron) usable by downstream PCA/GLASSO steps

## How to apply

Load the raw metabolite concentration table (90 metabolites estimated from integrated 1H-NMR peak areas, Supplementary Table S1) with pandas, retaining each sample's population identity alongside its intensity columns. Map every sample to one of the five cell populations defined by the paper's differentiation protocol — hESC, Embryonic Body (EB), Rosette, human Neuroprogenitor (hNPC), Neuron — following the four-step 'hESCs-to-EBs-to-Rosettes-to-hNPCs-to-Neuron' trajectory. Validate that labels reflect the actual population assayed by cross-checking against stage-specific purity markers used in the study (CD24 for EBs, ZO-1 for Rosettes, Nestin for hNPCs, MAP2 and vGLUT1 for neurons). Keep stage labels as a metadata column separate from the metabolite intensity features, so that the subsequent mean- and stddev-normalization operates on features only and PCA (10 components, first three retained) can be interpreted per stage. Preserve original metabolite identifiers and stage labels when writing the table (e.g., normalized_metabolite_table.csv) so downstream pcaglasso visualization can overlay stage-specific PCA scores on GLASSO networks.

## Related tools

- **pandas** (Loads the raw NMR metabolite table while retaining sample stage labels as metadata columns during preprocessing)
- **MetaboLINK (PCA-GLASSO algorithm)** (Downstream consumer: interprets PCA scores (10 components, first three used) per labeled developmental stage) — github.com/jlichtarge/pcaGLASSO
- **pcaglasso** (Overlays GLASSO graphs with stage-specific PCA scores to identify significant sub-networks per cell population) — github.com/jlichtarge/pcaGLASSO

## Evaluation signals

- Every sample in the output table carries exactly one of the five labels hESC, EB, Rosette, hNPC, or Neuron — no unlabeled or off-vocabulary values.
- Label assignments are consistent with the four-stage differentiation order (hESC → EB → Rosette → hNPC → Neuron) and with each sample's marker-based purity evidence (CD24, ZO-1, Nestin, MAP2, vGLUT1).
- The number of labeled populations matches the NMR acquisitions (one population per 1,000,000-cell sample set).
- Stage labels remain a metadata column: the mean- and stddev-normalization and PCA (10 components) are computed over the 90 metabolite features only, with no label leakage into the feature matrix.
- Re-reading the output table preserves original metabolite identifiers and stage labels, enabling reproducible stage-stratified PCA score plots in pcaglasso.

## Limitations

- Labels depend on marker-based purity assessment (CD24, ZO-1, Nestin, MAP2, vGLUT1); impure or partially differentiated populations can be mislabeled, propagating error into stage-specific PCA/GLASSO interpretation.
- The validated vocabulary covers only the hESC-derived neural lineage (H9 line) differentiation to glutamatergic neurons; other tissue systems or species require a new stage definition.
- No accession or availability statement for the metabolomics dataset is given in the article, so stage-label verification against raw deposited data may not be possible.

## Evidence

- [methods] Stage vocabulary from the differentiation protocol: "The stage-specific differentiation to glutamatergic neurons is comprised of four distinct stages “hESCs-to-Embryonic Bodies (EBs)-to-Rosettes-to-human Neuroprogenitors (hNPCs)-to-Neuron.”"
- [methods] Marker-based validation of stage identity: "We used cell specific markers to examine cell population purity at each stage: CD24 for EBs, ZO-1 for Rosettes, Nestin for hNPCs, and MAP2 and vGLUT1 for neurons."
- [methods] One NMR sample per labeled population: "NMR was acquired on 1,000,000 cells for each population."
- [methods] Stage labels as metadata for downstream PCA: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [readme] Downstream use of stage-labeled PCA scores: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."

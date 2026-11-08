---
name: mass-spectrometry-feature-table-processing
description: Use when you have GNPS FBMN output (DBResult/Node_info files), a CANOPUS
  canopus_summary.tsv, and need putative ClassyFire or NPClassifier annotations per
  ion feature for clustering or comparative metabolomics.
when_to_use_negative:
- Input is already a fully annotated feature table with consensus chemical classes
  — go straight to aggregation/clustering rather than re-running ConCISE
- Library spectra lack ClassyFire ontology data and you cannot use NPClassifier or
  manually retrieve ClassyFire annotations — ConCISE consensus will be incomplete
- The goal is de novo metabolite structure elucidation rather than ontology-level
  classification of many ion features (use SIRIUS/CANOPUS directly)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ConCISE
  role: Core tool that fuses molecular networking, spectral library matching, and
    in silico class predictions into consensus ClassyFire/NPClassifier annotations
    for ion features
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Provides feature-based molecular networking and spectral library matching
    outputs consumed as consensus inputs
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: SIRIUS/CANOPUS
  role: In silico structural annotation source supplying canopus_summary.tsv class
    predictions per ion feature
  repo: https://bio.informatik.uni-jena.de/software/
- name: Python
  role: Runs the ConCISE CLI and downstream pandas aggregation plus scipy hierarchical
    clustering of class-abundance profiles
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:a9991d341eceb4a4686ecb123fdf88fc190976130313d418685782213f7c4f46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - ConCISE
  - GNPS
  - Python
  - SIRIUS/CANOPUS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mass-spectrometry-feature-table-processing

## Summary

Process GNPS feature-based molecular networking outputs and CANOPUS in silico summaries into per-ion-feature consensus chemical classifications (ClassyFire superclass/class/subclass), then aggregate these into species- or sample-level chemical-class profiles for downstream analyses such as hierarchical clustering.

## When to use

Use when you have GNPS FBMN output (DBResult/Node_info files), a CANOPUS canopus_summary.tsv, and need putative ClassyFire or NPClassifier annotations per ion feature for clustering or comparative metabolomics.

## When NOT to use

- Input is already a fully annotated feature table with consensus chemical classes — go straight to aggregation/clustering rather than re-running ConCISE
- Library spectra lack ClassyFire ontology data and you cannot use NPClassifier or manually retrieve ClassyFire annotations — ConCISE consensus will be incomplete
- The goal is de novo metabolite structure elucidation rather than ontology-level classification of many ion features (use SIRIUS/CANOPUS directly)

## Inputs

- GNPS feature-based molecular networking task ID or spectral library match (DBResult) file
- CANOPUS canopus_summary.tsv from SIRIUS in silico structure prediction
- GNPS networking info file (Node_info.tsv) with subnetwork/component membership
- Per-ion-feature intensity/abundance table with sample or species of origin

## Outputs

- ConCISE consensus annotation file with per-ion-feature ClassyFire/NPClassifier superclass, class, and subclass labels and percent-consensus values
- Sample/species-by-chemical-class relative abundance matrix (e.g. species_class_abundance.csv)
- Hierarchical clustering dendrogram of samples by chemical-class profiles (e.g. species_cluster_dendrogram.png)

## How to apply

Run ConCISE (CLI, GUI, or myBinder Jupyter notebook) on a GNPS task ID or spectral library match file together with a canopus_summary.tsv and Node_info.tsv, setting percent-consensus thresholds (defaults: superclass 50, class 70, subclass 70) and choosing ClassyFire or NPClassifier ontologies via the NPC flag. ConCISE fuses molecular networking, GNPS spectral library matching, and SIRIUS/CANOPUS in silico class predictions, propagating annotations across subnetworks only when structural classes are identical for the majority of ion features, which yields a consensus annotation file per ion feature. Load the consensus file with pandas, aggregate annotated ion features per species/sample, and compute the relative production of each chemical class as the fraction of annotated features (and their total intensity) per class. Assemble a sample-by-class relative-abundance matrix (e.g. species_class_abundance.csv), then cluster samples with scipy hierarchical linkage on those profiles. Note the known issue that GNPS no longer supplies ClassyFire ontology data for library matches, requiring NPClassifier substitution or manual ClassyFire retrieval via the Fiehn Labs classyfire Batch service using InChiKeys from the DBResult file.

## Related tools

- **ConCISE** (Core tool that fuses molecular networking, spectral library matching, and in silico class predictions into consensus ClassyFire/NPClassifier annotations for ion features) — https://github.com/Zquinlan/conCISE
- **GNPS** (Provides feature-based molecular networking and spectral library matching outputs consumed as consensus inputs) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **SIRIUS/CANOPUS** (In silico structural annotation source supplying canopus_summary.tsv class predictions per ion feature) — https://bio.informatik.uni-jena.de/software/
- **Python** (Runs the ConCISE CLI and downstream pandas aggregation plus scipy hierarchical clustering of class-abundance profiles)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Consensus file coverage: a reported benchmark is annotation of up to 76% of ion features (case study: 56.3% annotated) with mean consensus ≥ 87.8%
- True positive rate of consensus annotations exceeds 95% across all ChemOnt levels (superclass, class, subclass) per the paper's validation
- Relative class-abundance fractions per sample sum to 1 across classes
- Dendrogram sanity check: known-to-be-similar samples (e.g. the two domoic-acid-producing toxic Pseudo-nitzschia species) cluster together in one clade
- Consensus thresholds applied as configured (superclass ≥50%, class ≥70%, subclass ≥70% by default)

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are NPClassifier substitution (NPC=True) or manually merging Fiehn Labs classyfire Batch results (keyed on InChiKeys) into the DBResult file with superclass/class/subclass columns
- Annotation propagation is intentionally limited to structural classes identical for the majority of ion features in a subnetwork, so heterogeneous subnetworks remain unannotated
- Mac GUI is deprecated; Mac users must use the CLI or myBinder interface
- Consensus coverage depends on in silico prediction quality; percent-consensus thresholds trade coverage against confidence

## Evidence

- [abstract] ConCISE expanded the proportion of reliable and consistent ion feature annotation up to 76%: "The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%"
- [abstract] ConCISE TPR >95% across ChemOnt levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [readme] Consensus thresholds defaults: "Superclass percent consensus (optional; default = 50) Class percent consensus (optional; default = 70) Subclass percent consensus (optional; default = 70)"
- [readme] GNPS ClassyFire gap and workarounds: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [abstract] Majority-rule propagation rationale: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"

---
name: consensus-annotation-aggregation
description: Use when untargeted MS/MS ion features annotated via GNPS feature-based
  molecular networking plus a CANOPUS summary file must be fused into consensus ClassyFire
  (or NPClassifier) labels with confidence thresholds, or when consensus chemical-class
  profiles must be clustered across species/samples.
when_to_use_negative:
- GNPS library matches lack ClassyFire ontology columns (GNPS stopped supplying classyfire
  ontology information for spectral library matches) and no workaround has been applied
  — either switch to NPClassifier (NPC=True) or manually pull ClassyFire data via
  the Fiehn Labs classyfire Batch identifier and merge it back into the DBResult file
  with correct column names before running
- The analysis has no in silico structure predictions (no CANOPUS summary file) or
  no feature-based molecular networking output, since ConCISE consensus requires all
  three annotation sources
- The input is already a per-feature consensus annotation table or class-abundance
  matrix — skip directly to the aggregation/clustering step rather than re-running
  consensus computation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ConCISE
  role: Fuses molecular networking, GNPS spectral library matching, and in silico
    class predictions to compute consensus ClassyFire/NPClassifier annotations per
    ion feature; invoked via CLI (conciseCLI.py), GUI, or myBinder notebook
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking (Node_info file) and library spectral
    matches with ClassyFire ontologies used as consensus inputs
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides the in silico class predictions (canopus_summary.tsv) that ConCISE
    fuses with library matches into consensus annotations
- name: ClassyFire
  role: Defines the ChemOnt hierarchical ontology (superclass, class, subclass) on
    which consensus thresholds are evaluated; can also be manually queried via the
    Fiehn Labs classyfire Batch identifier
- name: scipy/Python
  role: Downstream aggregation of the consensus file into a species-by-class relative
    abundance matrix and hierarchical clustering (scipy.cluster.hierarchy.linkage,
    average linkage) with dendrogram visualization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:11b469d3fb7ea4a27db3d6be8eafa9c395573df0b1b59e353d1e0a41c8c56ffc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - Python
  - scipy/Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# consensus-annotation-aggregation

## Summary

Aggregate putative chemical classifications from GNPS library spectral matches and in silico predictions (CANOPUS/SIRIUS) into per-ion-feature consensus annotations using ConCISE, which propagates ClassyFire ChemOnt (superclass/class/subclass) labels only when a majority of ion features in a subnetwork agree. The consensus profiles can then be aggregated into species- or sample-level chemical-class abundance matrices for downstream clustering.

## When to use

Use when untargeted MS/MS ion features annotated via GNPS feature-based molecular networking plus a CANOPUS summary file must be fused into consensus ClassyFire (or NPClassifier) labels with confidence thresholds, or when consensus chemical-class profiles must be clustered across species/samples.

## When NOT to use

- GNPS library matches lack ClassyFire ontology columns (GNPS stopped supplying classyfire ontology information for spectral library matches) and no workaround has been applied — either switch to NPClassifier (NPC=True) or manually pull ClassyFire data via the Fiehn Labs classyfire Batch identifier and merge it back into the DBResult file with correct column names before running
- The analysis has no in silico structure predictions (no CANOPUS summary file) or no feature-based molecular networking output, since ConCISE consensus requires all three annotation sources
- The input is already a per-feature consensus annotation table or class-abundance matrix — skip directly to the aggregation/clustering step rather than re-running consensus computation

## Inputs

- GNPS task ID or GNPS spectral library match (DBResult) file with superclass/class/subclass ontology columns
- CANOPUS canopus_summary.tsv in silico class prediction file (from SIRIUS/CANOPUS)
- GNPS Node_info.tsv networking information file from feature-based molecular networking
- ConCISE consensus annotation file with per-ion-feature consensus ClassyFire superclass/class/subclass labels and sample/species of origin

## Outputs

- ConCISE consensus annotation file assigning putative ClassyFire (or NPClassifier) classifications to ion features with percent-consensus values
- species_class_abundance.csv — species-by-chemical-class relative abundance matrix (fractions summing to 1 per species)
- species_cluster_dendrogram.png — hierarchical clustering dendrogram of species by consensus class-abundance profiles

## How to apply

Run ConCISE (CLI, GUI, or the myBinder Jupyter notebook) with three inputs: a GNPS task ID or spectral library match file, a CANOPUS canopus_summary.tsv, and a Node_info.tsv networking info file, setting superclass/class/subclass percent-consensus thresholds (defaults 50/70/70) and the NPC flag (False for ClassyFire ChemOnt, True for NPClassifier ontologies). ConCISE fuses molecular networking, spectral library matching, and in silico class predictions, propagating annotations only when structural classes are identical for the majority of ion features within a subnetwork, and exports a consensus annotation file with per-ion superclass/class/subclass labels and percent consensus. For cross-sample comparison (as in the Pseudo-nitzschia exometabolite dataset MSV000081731 case study), load the consensus file with pandas, compute per-species relative production of each chemical class as the fraction of annotated ion features (and total intensity) per class, assemble a species-by-class matrix, and cluster with scipy hierarchical clustering (average linkage) to test whether chemically similar organisms group together. The method maintained a true positive rate greater than 95% across all ChemOnt levels, and in the case study 56.3% of ion features received a putative consensus annotation with a mean consensus of 87.8%.

## Related tools

- **ConCISE** (Fuses molecular networking, GNPS spectral library matching, and in silico class predictions to compute consensus ClassyFire/NPClassifier annotations per ion feature; invoked via CLI (conciseCLI.py), GUI, or myBinder notebook) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking (Node_info file) and library spectral matches with ClassyFire ontologies used as consensus inputs) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides the in silico class predictions (canopus_summary.tsv) that ConCISE fuses with library matches into consensus annotations)
- **ClassyFire** (Defines the ChemOnt hierarchical ontology (superclass, class, subclass) on which consensus thresholds are evaluated; can also be manually queried via the Fiehn Labs classyfire Batch identifier)
- **scipy/Python** (Downstream aggregation of the consensus file into a species-by-class relative abundance matrix and hierarchical clustering (scipy.cluster.hierarchy.linkage, average linkage) with dendrogram visualization)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Consensus accuracy: ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology (superclass, class, subclass)
- Annotation coverage: in the case study, 56.3% of ion features received a putative consensus annotation with a mean consensus of 87.8% — substantially above the paper's reported baseline of 76%
- Row-normalization invariant: relative class-abundance fractions per species must sum to 1 across classes
- Biological validation: the dendrogram must place the two domoic-acid-producing toxic Pseudo-nitzschia species together in one clade, as reported in the paper
- CLI thresholds match defaults (superclass 50, class 70, subclass 70 percent consensus) unless deliberately changed

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; users must either use NPClassifier instead or manually pull ClassyFire data for library-match InChiKeys via the Fiehn Labs classyfire Batch and merge it back into the DBResult file with the correct column names (superclass, class, subclass)
- Annotation propagation is limited to structural classes identical for the majority of ion features within a subnetwork, so low-agreement subnetworks receive no consensus label (only ~56–76% of features annotated)
- The Mac GUI is deprecated; Mac users must use the command line interface or the myBinder Jupyter notebook

## Evidence

- [abstract] the first tool to fuse molecular networking, spectral library matching and in silico class predictions to establish accurate putative classifications for entire subnetworks: "the first tool to fuse molecular networking, spectral library matching and in silico class predictions to establish accurate putative classifications for entire subnetworks"
- [abstract] By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [methods] ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [readme] python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [readme] Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire"

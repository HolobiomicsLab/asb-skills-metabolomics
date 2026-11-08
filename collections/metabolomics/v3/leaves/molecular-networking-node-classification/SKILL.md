---
name: molecular-networking-node-classification
description: Use when you have a GNPS feature-based molecular networking job (Node_info.tsv),
  CANOPUS in silico class predictions (canopus_summary.tsv), and spectral library
  match data, and need consensus chemical class annotations for ion features across
  subnetworks.
when_to_use_negative:
- When no molecular network or spectral library matches are available — ConCISE fuses
  molecular networking, spectral library matching and in silico class predictions,
  and cannot propagate consensus without a subnetwork structure.
- When you need exact structure elucidation rather than chemical-class annotation
  — ConCISE yields putative classifications at the ChemOnt superclass/class/subclass
  levels, not identified metabolite structures.
- When annotation confidence requirements cannot tolerate consensus thresholds — propagation
  is limited to structural classes identical for the majority of ion features within
  a subnetwork, so low-redundancy subnetworks with few class predictions will yield
  few consensus annotations.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: ConCISE
  role: Primary tool that fuses molecular networking, spectral library matching and
    in silico class predictions into consensus ChemOnt annotations via its CLI/GUI/myBinder
    interfaces
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking (task ID, Node_info.tsv) and spectral
    library matching data with ClassyFire ontologies
- name: SIRIUS
  role: In silico tool whose structural annotations feed the consensus classification
- name: CANOPUS
  role: Generates the canopus_summary.tsv in silico class predictions consumed by
    ConCISE
- name: ClassyFire
  role: Provides the ChemOnt hierarchical ontology (superclass, class, subclass) used
    for consensus classification; can be manually batch-queried with InChiKeys if
    GNPS no longer supplies ontology data
- name: NPClassifier
  role: Alternative ontology (NPC argument True) used in place of ClassyFire for both
    library matches and in silico matches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:e36db032fd49ec15c581216ee84807fc2fcdfbfabb92afa51cce821c3b9a0610
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - NPClassifier
  - Python
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# molecular-networking-node-classification

## Summary

Annotates ion features in GNPS feature-based molecular networking by fusing spectral library matches with SIRIUS/CANOPUS in silico class predictions into consensus ClassyFire ChemOnt classifications (superclass, class, subclass) per subnetwork, propagating annotations only where a structural class is shared by the majority of ion features.

## When to use

Use when you have a GNPS feature-based molecular networking job (Node_info.tsv), CANOPUS in silico class predictions (canopus_summary.tsv), and spectral library match data, and need consensus chemical class annotations for ion features across subnetworks.

## When NOT to use

- When no molecular network or spectral library matches are available — ConCISE fuses molecular networking, spectral library matching and in silico class predictions, and cannot propagate consensus without a subnetwork structure.
- When you need exact structure elucidation rather than chemical-class annotation — ConCISE yields putative classifications at the ChemOnt superclass/class/subclass levels, not identified metabolite structures.
- When annotation confidence requirements cannot tolerate consensus thresholds — propagation is limited to structural classes identical for the majority of ion features within a subnetwork, so low-redundancy subnetworks with few class predictions will yield few consensus annotations.

## Inputs

- GNPS task ID or spectral library match file (GNPS DBResult file with superclass/class/subclass columns)
- CANOPUS in silico class prediction summary file (canopus_summary.tsv) generated via SIRIUS/CANOPUS
- GNPS feature-based molecular networking node information file (Node_info.tsv)

## Outputs

- Consensus annotation file (per ion feature: consensus superclass, class, subclass at ChemOnt levels)
- Consensus annotation rate per dataset (e.g., fraction of ion features annotated; 58.6%, 56.3%, 76.5% in the three case studies)
- Fold expansion of annotation coverage relative to GNPS spectral library matching alone (up to 1244%)

## How to apply

Run ConCISE's CLI (python3 src/conciseCLI.py) supplying the GNPS task ID (or spectral library match file), the canopus_summary.tsv from SIRIUS/CANOPUS, and the Node_info.tsv networking file, with Superclass percent consensus = 50, Class percent consensus = 70, Subclass percent consensus = 70, and NPClassifier = False so ClassyFire ChemOnt ontologies are used for both library matches and in silico annotations. ConCISE finds consensus annotations of putative annotations and propagates them within subnetworks, restricting propagation to structural classes identical for the majority of ion features in a subnetwork. Export the consensus annotation file to the working directory, then count ion features receiving a consensus annotation divided by total ion features to obtain the consensus annotation rate, and compute fold expansion of coverage relative to GNPS spectral library matching alone. Note that since GNPS has stopped supplying ClassyFire ontology information for spectral library matches, either switch the NPC argument to True to use NPClassifier ontologies, or manually pull ClassyFire data by copying InChiKeys from the GNPS DBResult file into the Fiehn Labs classyfire Batch service and merging results back with superclass/class/subclass column names.

## Related tools

- **ConCISE** (Primary tool that fuses molecular networking, spectral library matching and in silico class predictions into consensus ChemOnt annotations via its CLI/GUI/myBinder interfaces) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking (task ID, Node_info.tsv) and spectral library matching data with ClassyFire ontologies)
- **SIRIUS** (In silico tool whose structural annotations feed the consensus classification)
- **CANOPUS** (Generates the canopus_summary.tsv in silico class predictions consumed by ConCISE)
- **ClassyFire** (Provides the ChemOnt hierarchical ontology (superclass, class, subclass) used for consensus classification; can be manually batch-queried with InChiKeys if GNPS no longer supplies ontology data)
- **NPClassifier** (Alternative ontology (NPC argument True) used in place of ClassyFire for both library matches and in silico matches)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- True positive rate should exceed 95% at all ChemOnt levels (superclass, class, subclass) when validated against known annotations.
- Consensus annotation rate = (ion features with a consensus annotation) / (total ion features); the three case studies report 58.6%, 56.3%, and 76.5%.
- Annotation coverage expansion over GNPS spectral library matching alone should be quantified; the paper reports expansion up to 1244% in subnetwork annotation rate.
- Verify output consensus file columns use the correct ClassyFire ontology column names (superclass, class, subclass) when merging manually pulled ClassyFire data.
- Propagated annotations should only appear where the structural class is identical for the majority of ion features within a subnetwork.

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are using NPClassifier (NPC=True) or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch service using InChiKeys from the GNPS DBResult file.
- The Mac GUI is deprecated; Mac users must use the command line or myBinder interface.
- Consensus thresholds (Superclass 50%, Class 70%, Subclass 70%) restrict propagation to majority-shared classes, limiting annotations in heterogeneous subnetworks.
- In silico inputs depend on SIRIUS/CANOPUS; new in silico tools historically could not be used without significant modification.

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [abstract] fusion of networking, library matching and in silico predictions: "Herein we present ConCISE (Consensus Classifications of In Silico Elucidations) which is the first tool to fuse molecular networking, spectral library matching and in silico class predictions"
- [abstract] propagation limited to majority-identical classes: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] CLI arguments and defaults: "Superclass percent consensus (optional; default = 50)
- Class percent consensus (optional; default = 70)
- Subclass percent consensus (optional; default = 70)"
- [readme] GNPS ClassyFire workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [readme] example CLI invocation: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"

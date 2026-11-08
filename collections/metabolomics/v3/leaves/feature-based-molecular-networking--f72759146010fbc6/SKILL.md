---
name: feature-based-molecular-networking
description: Use when untargeted LC-MS/MS feature tables need to be organized into
  molecular subnetworks so that per-node library matches and CANOPUS class predictions
  can be propagated and scored at the subnetwork level.
when_to_use_negative:
- When the analysis goal is per-node spectral library identification alone without
  subnetwork-level propagation — molecular networking adds no consensus benefit if
  consensus across a subnetwork is not required.
- When GNPS library matches cannot be obtained with ClassyFire ontology columns and
  NPClassifier ontologies are incompatible with the downstream tool's expected ChemOnt
  hierarchy.
- When the data are not untargeted tandem MS feature detections (e.g., targeted quantification
  or genomics data) — networking requires MS/MS spectra of ion features.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: GNPS
  role: Hosts feature-based molecular networking; supplies spectral library matches
    and the ClassyFire/NPClassifier ontology annotations used for consensus classification
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: ConCISE
  role: Downstream tool that fuses molecular networking, spectral library matching
    and in silico class predictions to establish consensus classifications for entire
    subnetworks
  repo: https://github.com/Zquinlan/conCISE
- name: CANOPUS
  role: Provides in silico class predictions (canopus_summary.tsv) for ion features
    lacking library matches, propagated across the molecular network
- name: SIRIUS
  role: In silico structural elucidation tool whose output classes feed CANOPUS predictions
    combined with networking results
  repo: https://bio.informatik.uni-jena.de/software/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:80feb8f4a56765e7fc90123750b4c50af336976c3a961f23a6ee01bb036a4002
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - Python
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# feature-based-molecular-networking

## Summary

Generate GNPS feature-based molecular networking outputs (DBResult/library spectral matches and Node_info.tsv) that group ion features into molecular subnetworks, providing the network topology and node-level annotations that downstream consensus annotation propagation (e.g., ConCISE) requires.

## When to use

Use when untargeted LC-MS/MS feature tables need to be organized into molecular subnetworks so that per-node library matches and CANOPUS class predictions can be propagated and scored at the subnetwork level.

## When NOT to use

- When the analysis goal is per-node spectral library identification alone without subnetwork-level propagation — molecular networking adds no consensus benefit if consensus across a subnetwork is not required.
- When GNPS library matches cannot be obtained with ClassyFire ontology columns and NPClassifier ontologies are incompatible with the downstream tool's expected ChemOnt hierarchy.
- When the data are not untargeted tandem MS feature detections (e.g., targeted quantification or genomics data) — networking requires MS/MS spectra of ion features.

## Inputs

- Untargeted LC-MS/MS feature table with associated MS/MS spectra (e.g., from MZmine, as processed for MassIVE ID MSV000082083)
- GNPS feature-based molecular networking job outputs: DBResult spectral library match file and Node_info.tsv
- CANOPUS in silico class predictions (canopus_summary.tsv from SIRIUS)
- ClassyFire or NPClassifier ontology annotations per node (superclass, class, subclass)

## Outputs

- Molecular subnetwork assignments per ion feature (Node_info.tsv)
- Per-node spectral library matches with chemical ontology annotations (DBResult)
- Subnetwork-level input structure for consensus annotation propagation (consensus_scores.csv via ConCISE)

## How to apply

Process LC-MS/MS data through GNPS feature-based molecular networking to produce a networking job whose outputs include the spectral library match file (DBResult with superclass/class/subclass ClassyFire ontology columns) and the Node_info.tsv networking info file mapping each ion feature to its molecular subnetwork. Feed these, together with the CANOPUS in silico class predictions (canopus_summary.tsv from SIRUS/CANOPUS), into consensus annotation propagation. Note that GNPS currently does not supply ClassyFire ontology information for spectral library matches, so either switch to NPClassifier ontologies (NPC argument True) or manually batch-query InChiKeys from the GNPS DBResult via the Fiehn Labs classyFire Batch service and merge the superclass/class/subclass columns back into the libraryID file. Keep ontology defaults (superclass 50%, class 70%, subclass 70% consensus thresholds) when the resulting subnetworks are used for ConCISE consensus scoring, as in the coral reef primary producer dataset MSV000082083.

## Related tools

- **GNPS** (Hosts feature-based molecular networking; supplies spectral library matches and the ClassyFire/NPClassifier ontology annotations used for consensus classification) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **ConCISE** (Downstream tool that fuses molecular networking, spectral library matching and in silico class predictions to establish consensus classifications for entire subnetworks) — https://github.com/Zquinlan/conCISE
- **CANOPUS** (Provides in silico class predictions (canopus_summary.tsv) for ion features lacking library matches, propagated across the molecular network)
- **SIRIUS** (In silico structural elucidation tool whose output classes feed CANOPUS predictions combined with networking results) — https://bio.informatik.uni-jena.de/software/

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Node_info.tsv maps every ion feature to exactly one molecular subnetwork; subnetwork counts match the feature count after clustering.
- For the coral reef primary producer dataset (MSV000082083), ConCISE annotated 61.09% of subnetworks with mean consensus 87.4% — a reproducible benchmark.
- Subnetwork 131 (43 annotated features) reaches 95.3% consensus and fatty amide subnetwork 2263 reaches 72.0% consensus when the pipeline is run correctly.
- Library match DBResult file contains superclass/class/subclass columns populated (via manual classyFire Batch merge or NPClassifier flag) despite GNPS discontinuing ClassyFire ontology supply.
- ConCISE true positive rate exceeds 95% across ChemOnt superclass, class, and subclass levels when validated against known library matches.

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; workarounds are NPClassifier mode or manual batch retrieval of ClassyFire data via the Fiehn Labs classyFire Batch service and merging columns back into the DBResult.
- Mac GUI is deprecated; command line or MyBinder must be used on macOS.
- Annotation propagation is limited to structural classes identical for the majority of ion features within a subnetwork, so sparse or heterogeneous subnetworks may receive low or no consensus.

## Evidence

- [abstract] the first tool to fuse molecular networking, spectral library matching and in silico class predictions to establish accurate putative classifications for entire subnetworks: "the first tool to fuse molecular networking, spectral library matching and in silico class predictions to establish accurate putative classifications for entire subnetworks"
- [methods] networking tools from [GNPS] such as feature based molecular networking: "networking tools from [GNPS](https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp) such as feature based molecular networking***"
- [readme] ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches and in silico putative annotations."
- [intro] Currently GNPS has stopped supplying classyfire ontology information for spectral library matches: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches"
- [readme] You can copy the InChiKey's from the GNPS DBResult file into the Fiehn Labs' classyfire Batch identifier: "You can copy the InChiKey's from the GNPS DBResult file into the Fiehn Labs' [classyfire Batch](https://cfb.fiehnlab.ucdavis.edu/) identifier."

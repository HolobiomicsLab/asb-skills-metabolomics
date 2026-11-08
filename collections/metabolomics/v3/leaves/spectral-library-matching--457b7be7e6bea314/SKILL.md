---
name: spectral-library-matching
description: Use when annotating ion features from untargeted LC-MS/MS feature-based
  molecular networking runs against a spectral library, to supply high-confidence
  superclass/class/subclass ontology labels for consensus propagation.
when_to_use_negative:
- GNPS no longer supplies ClassyFire ontology information for spectral library matches
  — without applying the NPClassifier switch or the classyfire Batch InChiKey workaround,
  library-match-based ClassyFire consensus cannot be computed
- The target compounds are absent from spectral libraries and only in silico (CANOPUS)
  annotations are available — the in-silico-only consensus path has a lower true positive
  rate (96.1% ± 4.96%) and library matching adds nothing
- The analysis is not MS/MS feature-based molecular networking (e.g., no molecular
  networking subnetworks), since ConCISE consensus propagation relies on subnetwork-level
  annotation majority
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ConCISE
  role: Consumes the GNPS spectral library match file together with canopus_summary.tsv
    and Node_info.tsv to compute consensus superclass/class/subclass annotations
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Provides feature-based molecular networking and spectral library matching;
    supplies ontology annotations for library spectral matches used in consensus annotation
- name: CANOPUS
  role: Supplies in silico ClassyFire ChemOnt class predictions (canopus_summary.tsv)
    that are fused with library spectral matches during consensus annotation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:cefe5571271e1e9a889352b7dcef9d960a617f5002bf34465d396cba22b240d9
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
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spectral-library-matching

## Summary

Assign chemical identities to ion features by matching MS/MS spectra against a reference spectral library (GNPS library spectral matches), then feed those library annotations into ConCISE's consensus classification. In the ConCISE case studies, library-match consensus annotations achieved a true positive rate of 98.88% ± 1.62% across ClassyFire ChemOnt levels.

## When to use

Use when annotating ion features from untargeted LC-MS/MS feature-based molecular networking runs against a spectral library, to supply high-confidence superclass/class/subclass ontology labels for consensus propagation.

## When NOT to use

- GNPS no longer supplies ClassyFire ontology information for spectral library matches — without applying the NPClassifier switch or the classyfire Batch InChiKey workaround, library-match-based ClassyFire consensus cannot be computed
- The target compounds are absent from spectral libraries and only in silico (CANOPUS) annotations are available — the in-silico-only consensus path has a lower true positive rate (96.1% ± 4.96%) and library matching adds nothing
- The analysis is not MS/MS feature-based molecular networking (e.g., no molecular networking subnetworks), since ConCISE consensus propagation relies on subnetwork-level annotation majority

## Inputs

- MS/MS spectra / feature-based molecular networking output from a GNPS task (Node_info.tsv)
- GNPS spectral library match file (libraryID / DBResult) with superclass, class, subclass columns
- CANOPUS class prediction summary file (canopus_summary.tsv) from SIRIUS 4

## Outputs

- Consensus annotation file with consensus superclass, class, and subclass assignments per ion feature
- True positive rate table (tpr_summary_table.tsv) with one row per dataset per consensus-threshold condition and ontology level

## How to apply

Process each dataset with GNPS feature-based molecular networking so that, in addition to the Node_info.tsv networking file, a GNPS spectral library match file (libraryID/DBResult) is produced containing library-hit compound identities. Supply this library match file to ConCISE alongside the SIRIUS 4/CANOPUS canopus_summary.tsv so consensus annotations are computed from both library spectral matches and in silico putative annotations using ClassyFire ChemOnt ontologies, running the CLI with superclass/class/subclass consensus thresholds of 50/70/70. Because GNPS has stopped supplying ClassyFire ontology information for spectral library matches, either switch the NPC argument to True to use NPClassifier ontologies, or manually copy InChiKeys from the GNPS DBResult file into the Fiehn Labs classyfire Batch service and merge superclass/class/subclass columns back into the file used in place of libraryID. Finally, compare consensus superclass, class, and subclass assignments against the GNPS-reported compound identities to compute true positive rates per ontology level.

## Related tools

- **ConCISE** (Consumes the GNPS spectral library match file together with canopus_summary.tsv and Node_info.tsv to compute consensus superclass/class/subclass annotations) — https://github.com/Zquinlan/conCISE
- **GNPS** (Provides feature-based molecular networking and spectral library matching; supplies ontology annotations for library spectral matches used in consensus annotation)
- **CANOPUS** (Supplies in silico ClassyFire ChemOnt class predictions (canopus_summary.tsv) that are fused with library spectral matches during consensus annotation)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Computed superclass, class, and subclass true positive rates exceed 95% across all levels of the ChemOnt hierarchical ontology
- Library-match consensus annotations reach the reported true positive rate of 98.88% ± 1.62% when validated against GNPS-reported compound identities
- Consensus annotation file is exported to the working directory with superclass, class, and subclass columns populated at the 50/70/70 consensus thresholds
- tpr_summary_table.tsv contains one row per dataset per consensus-threshold condition and ontology level across the three MassIVE datasets (MSV000082083, MSV000081731, MSV000083729)

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; a workaround (NPClassifier ontologies or manual classyfire Batch InChiKey lookup) is required
- Annotation propagation is limited to structural classes identical for the majority of ion features within a subnetwork, so minority classes within a subnetwork are not propagated
- The Mac GUI is deprecated; on macOS use the command line or MyBinder interfaces

## Evidence

- [methods] ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches"
- [abstract] first tool to fuse molecular networking, spectral library matching and in silico class predictions: [evidence span withheld — anchor status: not_found]
- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software"
- [readme] GNPS has stopped supplying classyfire ontology information for spectral library matches: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire"

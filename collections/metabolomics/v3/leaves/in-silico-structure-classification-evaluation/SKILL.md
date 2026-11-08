---
name: in-silico-structure-classification-evaluation
description: 'Use when a ConCISE consensus annotation run must be validated: compare
  consensus superclass/class/subclass assignments against GNPS-reported compound identities
  to compute true positive rates and check the >95% criterion.'
when_to_use_negative:
- Input lacks GNPS-reported compound identities for the ion features, so true positive
  rates against ground truth cannot be computed
- The consensus annotations were produced with NPClassifier ontologies rather than
  ClassyFire ChemOnt, so they cannot be judged against the paper's ClassyFire-based
  >95% TPR criterion
- The goal is novel metabolite annotation rather than validation of an existing consensus
  annotation output
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: ConCISE
  role: Produces the consensus superclass/class/subclass annotations that are evaluated
    against GNPS-reported compound identities
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking results and spectral library matches
    whose compound identities serve as the reference labels for TPR computation
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: SIRIUS
  role: Runs with CANOPUS to generate in silico class predictions that feed the consensus
    annotations being evaluated
- name: CANOPUS
  role: Generates the canopus_summary.tsv ChemOnt class predictions used in consensus
    annotation and TPR evaluation
- name: ClassyFire
  role: Defines the ChemOnt hierarchical ontology (superclass, class, subclass) over
    which true positive rates are computed
- name: Python
  role: Executes the conciseCLI and computes per-level true positive rates and per-subnetwork
    consensus scores
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:2682305e7056ad45ac0fb84622bfcff0fadb498179abca32b21f02f7364b1c60
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# in-silico-structure-classification-evaluation

## Summary

Evaluate the correctness of consensus chemical classifications produced by fusing GNPS molecular networking, spectral library matching, and in silico (SIRIUS/CANOPUS) predictions by computing true positive rates against GNPS-reported compound identities across the ChemOnt hierarchy.

## When to use

Use when a ConCISE consensus annotation run must be validated: compare consensus superclass/class/subclass assignments against GNPS-reported compound identities to compute true positive rates and check the >95% criterion.

## When NOT to use

- Input lacks GNPS-reported compound identities for the ion features, so true positive rates against ground truth cannot be computed
- The consensus annotations were produced with NPClassifier ontologies rather than ClassyFire ChemOnt, so they cannot be judged against the paper's ClassyFire-based >95% TPR criterion
- The goal is novel metabolite annotation rather than validation of an existing consensus annotation output

## Inputs

- ConCISE consensus annotation file (exported from conciseCLI.py)
- GNPS feature-based molecular networking Node_info.tsv
- GNPS spectral library match / DBResult file (with superclass, class, subclass columns)
- SIRIUS 4 + CANOPUS canopus_summary.tsv with ClassyFire ChemOnt annotations
- MassIVE case-study datasets MSV000082083, MSV000081731, MSV000083729

## Outputs

- tpr_summary_table.tsv with true positive rate per dataset per consensus-threshold condition and ChemOnt level
- consensus_scores.csv with per-subnetwork consensus scores
- per-level true positive rates (superclass, class, subclass) compared against the >95% criterion

## How to apply

Run ConCISE with the default ClassyFire ontology thresholds (superclass percent consensus = 50, class = 70, subclass = 70, NPClassifier flag = False) on a GNPS feature-based molecular networking task together with the SIRIUS/CANOPUS canopus_summary.tsv and Node_info.tsv. Export the consensus annotation file, then compare the consensus superclass, class, and subclass assignments against the GNPS-reported compound identities for each dataset. Compute the true positive rate per ontology level and assemble one row per dataset per consensus-threshold condition and level into tpr_summary_table.tsv. Validate against the reported performance: 98.88% ± 1.62% TPR for spectral-library consensus annotations and 96.1% ± 4.96% for in silico consensus annotations, with the criterion of a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology. For case-study level checks (e.g., MSV000082083), also verify that 61.09% of molecular subnetworks received consensus annotations with a mean subnetwork consensus of 87.4%.

## Related tools

- **ConCISE** (Produces the consensus superclass/class/subclass annotations that are evaluated against GNPS-reported compound identities) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking results and spectral library matches whose compound identities serve as the reference labels for TPR computation) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **SIRIUS** (Runs with CANOPUS to generate in silico class predictions that feed the consensus annotations being evaluated)
- **CANOPUS** (Generates the canopus_summary.tsv ChemOnt class predictions used in consensus annotation and TPR evaluation)
- **ClassyFire** (Defines the ChemOnt hierarchical ontology (superclass, class, subclass) over which true positive rates are computed)
- **Python** (Executes the conciseCLI and computes per-level true positive rates and per-subnetwork consensus scores)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Spectral-library consensus annotations yield TPR of 98.88% ± 1.62% and in silico consensus annotations yield TPR of 96.1% ± 4.96% across Superclass, Class, Subclass
- TPR exceeds 95% at every level of the ChemOnt hierarchical ontology (superclass, class, subclass)
- On MSV000082083, 61.09% of molecular subnetworks receive a consensus annotation with a mean subnetwork consensus of 87.4%
- Subnetwork 131 (43 annotated features) reaches 95.3% consensus and fatty amide subnetwork 2263 reaches 72.0% consensus
- tpr_summary_table.tsv contains one row per dataset per consensus-threshold condition and ontology level

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; workarounds are using NPClassifier (NPC argument True) or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch server using InChiKeys from the GNPS DBResult file and merging them back with correct column names
- ConCISE limits annotation propagation to structural classes identical for the majority of ion features within a subnetwork, so subnetworks with low annotation agreement may receive no consensus
- The Mac GUI is deprecated; use the CLI or MyBinder interface on macOS

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [results] TPR values for library and in silico consensus annotations: [evidence span withheld — anchor status: not_found]
- [methods] CLI invocation with consensus thresholds: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [readme] Consensus thresholds are CLI parameters: "Superclass percent consensus (optional; default = 50)"
- [readme] GNPS ClassyFire ontology gap and workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire by checking the box in the GUI"

---
name: mass-spectrometry-feature-annotation-qc
description: Use when untargeted MS/MS feature tables (e.g., a GNPS FBMN task with
  canopus_summary.tsv and Node_info.tsv) need consensus chemical-class coverage and
  per-subnetwork consensus-score QC metrics.
when_to_use_negative:
- You need library spectral matches annotated with ClassyFire ontology directly from
  GNPS — GNPS has stopped supplying classyfire ontology information for spectral library
  matches, so a workaround (NPClassifier or manual Fiehn Labs classyfire Batch lookup
  merged back into the DBResult file) is required
- You need metabolite structural elucidation rather than consensus class-level annotation
  — structure prediction must be done upstream with tools such as SIRIUS/CANOPUS before
  ConCISE
- Your subnetworks lack majority-shared structural classes — annotation propagation
  is limited to classes identical for the majority of ion features within a subnetwork,
  so heterogeneous subnetworks will yield low or no consensus
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Fuses molecular networking, spectral library matching and in silico class
    predictions to compute consensus annotations and per-subnetwork consensus scores;
    run via CLI (conciseCLI.py), GUI, or myBinder Jupyter notebook
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking results and spectral library matches
    with ClassyFire ontology annotations (Node_info.tsv, DBResult)
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides in silico class predictions (canopus_summary.tsv) that serve as putative
    annotations for consensus computation
- name: SIRIUS
  role: Upstream in silico structural annotation tool whose outputs feed CANOPUS class
    predictions used by ConCISE
- name: ClassyFire
  role: ChemOnt hierarchical ontology (superclass, class, subclass) against which
    consensus percentages are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:080e356cb9de1eb3aff3190046231cc3fbab388a6f1a09fdf35d38c22ae01a67
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mass-spectrometry-feature-annotation-qc

## Summary

Quantifies how many molecular subnetworks and ion features received consensus annotations and at what consensus scores, using ConCISE to fuse GNPS feature-based molecular networking, spectral library matches, and CANOPUS in silico class predictions across the ClassyFire ChemOnt hierarchy.

## When to use

Use when untargeted MS/MS feature tables (e.g., a GNPS FBMN task with canopus_summary.tsv and Node_info.tsv) need consensus chemical-class coverage and per-subnetwork consensus-score QC metrics.

## When NOT to use

- You need library spectral matches annotated with ClassyFire ontology directly from GNPS — GNPS has stopped supplying classyfire ontology information for spectral library matches, so a workaround (NPClassifier or manual Fiehn Labs classyfire Batch lookup merged back into the DBResult file) is required
- You need metabolite structural elucidation rather than consensus class-level annotation — structure prediction must be done upstream with tools such as SIRIUS/CANOPUS before ConCISE
- Your subnetworks lack majority-shared structural classes — annotation propagation is limited to classes identical for the majority of ion features within a subnetwork, so heterogeneous subnetworks will yield low or no consensus

## Inputs

- GNPS task ID or spectral library match (DBResult) file
- CANOPUS in silico class predictions file (canopus_summary.tsv)
- GNPS networking info file (Node_info.tsv)
- Optional CLI parameters: export directory, superclass/class/subclass percent consensus thresholds (50/70/70), NPClassifier flag (True/False)

## Outputs

- ConCISE consensus annotation file (superclass/class/subclass per subnetwork)
- Per-subnetwork consensus score file (consensus_scores.csv)
- Summary metrics: fraction of subnetworks annotated and mean subnetwork consensus score

## How to apply

Retrieve the GNPS feature-based molecular networking results (GNPS task ID or library match file plus Node_info.tsv) and the CANOPUS in silico class predictions (canopus_summary.tsv) from SIRIUS. Run the ConCISE CLI (python3 src/conciseCLI.py) with the GNPS task ID, canopus_summary.tsv, and Node_info.tsv, keeping the ClassyFire ontology defaults: superclass percent consensus = 50, class percent consensus = 70, subclass percent consensus = 70, NPClassifier flag = False. ConCISE propagates annotations across a subnetwork only when structural classes are identical for the majority of ion features, then parses the exported consensus file to compute the fraction of subnetworks annotated at each ChemOnt level and the mean per-subnetwork consensus score. Extract scores for specific subnetworks to verify against reported values, e.g., coral reef primary producer subnetwork 131 (43 annotated features, 95.3% consensus) and fatty amide subnetwork 2263 (72.0%), writing consensus_scores.csv to the output directory.

## Related tools

- **ConCISE** (Fuses molecular networking, spectral library matching and in silico class predictions to compute consensus annotations and per-subnetwork consensus scores; run via CLI (conciseCLI.py), GUI, or myBinder Jupyter notebook) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking results and spectral library matches with ClassyFire ontology annotations (Node_info.tsv, DBResult)) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides in silico class predictions (canopus_summary.tsv) that serve as putative annotations for consensus computation)
- **SIRIUS** (Upstream in silico structural annotation tool whose outputs feed CANOPUS class predictions used by ConCISE)
- **ClassyFire** (ChemOnt hierarchical ontology (superclass, class, subclass) against which consensus percentages are computed)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Reported benchmark: true positive rate > 95% across all ChemOnt levels (superclass, class, subclass); flag if your reproduction falls far below
- Dataset-level check: ConCISE expanded the proportion of reliable and consistent ion feature annotation up to 76%, and annotated 61.09% of subnetworks in the coral reef primary producer dataset with mean consensus 87.4%
- Per-subnetwork verification: subnetwork 131 (43 annotated features) should reach 95.3% consensus and fatty amide subnetwork 2263 should reach 72.0%
- Parameter invariants: defaults superclass=50, class=70, subclass=70 percent consensus and NPClassifier=False must be honored when reproducing published metrics
- Schema check: consensus export must carry ClassyFire ontology levels (superclass, class, subclass) as columns

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC=True) or manually pulling classyfire data by copying InChiKeys from the GNPS DBResult file into the Fiehn Labs classyfire Batch and merging results back with correct column names (superclass, class, subclass)
- Mac GUI is deprecated; Mac users must use the CLI or myBinder interface
- Annotation propagation is limited to structural classes identical for the majority of ion features within a subnetwork, so subnetworks without a majority-shared class receive no consensus

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software"
- [readme] cli arguments: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [readme] known issue: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [abstract] propagation rule: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [methods] superclass threshold: "Superclass percent consensus (optional; default = 50)"

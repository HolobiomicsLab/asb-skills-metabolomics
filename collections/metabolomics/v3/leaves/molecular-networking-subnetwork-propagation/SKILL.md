---
name: molecular-networking-subnetwork-propagation
description: Use when you have GNPS feature-based molecular networking output plus
  a CANOPUS canopus_summary.tsv and node info with library spectral matches, and need
  to propagate chemical class annotations to unannotated subnetwork members.
when_to_use_negative:
- Subnetworks (or singletons) contain no library spectral match — ConCISE discards
  these nodes and assigns no in silico annotation, so propagation cannot help.
- GNPS has not supplied ClassyFire ontology columns for your library matches and you
  have not applied a workaround (NPClassifier or manual Fiehn classyfire Batch lookup)
  — the consensus tally cannot be computed without superclass/class/subclass annotations.
- You need structural (SMILES/InChI) annotations rather than hierarchical class labels
  — ConCISE only propagates ChemOnt (or NPClassifier) classification levels.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Performs the consensus annotation propagation across subnetworks via GUI,
    CLI, or myBinder notebook
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking and spectral library matches with
    ClassyFire ontology columns used for consensus tallying
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides the in silico ChemOnt class predictions (canopus_summary.tsv) fused
    with library matches
- name: NPClassifier
  role: Optional alternative ontology used for both library matches and in silico
    matches when ClassyFire data is unavailable (NPC argument True)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:d3fd65c6f9903fa94d68c97b440837e3c8f222e2d2dc2b788672b12ce4eec33d
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# molecular-networking-subnetwork-propagation

## Summary

ConCISE propagates consensus ClassyFire ChemOnt classifications from library-matched and CANOPUS in-silico-annotated nodes to unannotated ion features within GNPS molecular networking subnetworks, using percent-consensus cutoffs (superclass >= 50%, class/subclass >= 70%).

## When to use

Use when you have GNPS feature-based molecular networking output plus a CANOPUS canopus_summary.tsv and node info with library spectral matches, and need to propagate chemical class annotations to unannotated subnetwork members.

## When NOT to use

- Subnetworks (or singletons) contain no library spectral match — ConCISE discards these nodes and assigns no in silico annotation, so propagation cannot help.
- GNPS has not supplied ClassyFire ontology columns for your library matches and you have not applied a workaround (NPClassifier or manual Fiehn classyfire Batch lookup) — the consensus tally cannot be computed without superclass/class/subclass annotations.
- You need structural (SMILES/InChI) annotations rather than hierarchical class labels — ConCISE only propagates ChemOnt (or NPClassifier) classification levels.

## Inputs

- GNPS task ID or GNPS spectral library match (DBResult) file with superclass/class/subclass columns
- CANOPUS canopus_summary.tsv file (SIRIUS/CANOPUS in silico ChemOnt class predictions)
- Node_info.tsv networking info file from GNPS feature-based molecular networking

## Outputs

- ConCISEConsensus.csv with one row per feature containing featureNumber, consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico)

## How to apply

Load the GNPS Node_info.tsv (networking and library match info) and CANOPUS canopus_summary.tsv (in silico class predictions) into Python/pandas, group ion features into their molecular networking subnetworks, and restrict propagation to subnetworks containing at least one library spectral match — singleton nodes without library matches receive no annotation. For each subnetwork, tally agreement of ClassyFire ChemOnt annotations among library matches and CANOPUS putative annotations at the superclass, class, and subclass levels, then apply the consensus cutoffs (superclass >= 50%, class >= 70%, subclass >= 70%). Select the lowest (most granular) ChemOnt level that passes its cutoff together with all higher levels as the consensus annotation. Run via the CLI with defaults (thresholds 50 70 70, NPClassifier disabled) to write ConCISEConsensus.csv with one row per feature including consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico). The rationale is that propagation is limited to structural classes identical for the majority of ion features within a subnetwork, which maintains a true positive rate > 95% across all ChemOnt levels.

## Related tools

- **ConCISE** (Performs the consensus annotation propagation across subnetworks via GUI, CLI, or myBinder notebook) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking and spectral library matches with ClassyFire ontology columns used for consensus tallying) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides the in silico ChemOnt class predictions (canopus_summary.tsv) fused with library matches)
- **NPClassifier** (Optional alternative ontology used for both library matches and in silico matches when ClassyFire data is unavailable (NPC argument True))

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- ConCISEConsensus.csv exists and parses with the required columns: featureNumber, hierarchy, score, level, node count, and source.
- Every exported consensus score satisfies the 50/70/70 percent-cutoff criteria at superclass/class/subclass levels.
- The exported consensus annotation appears at the lowest (most granular) ChemOnt level passing its cutoff, with all higher hierarchy levels included.
- Subnetworks without any library spectral match (and singleton nodes) are absent from the consensus output or carry no annotation.
- Reported propagated annotations maintain a true positive rate greater than 95% at superclass, class, and subclass levels.

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC argument True) or manually merging Fiehn Labs classyfire Batch results into the GNPS DBResult file with superclass/class/subclass column names.
- Singleton nodes without library matches receive no in silico annotation, and propagation is restricted to subnetworks containing at least one library match.
- The Mac GUI is deprecated; Mac users must use the CLI or myBinder notebook interface.

## Evidence

- [abstract] Herein we present ConCISE (Consensus Classifications of In Silico Elucidations) which is the first tool to fuse molecular networking, spectral library matching and in silico class predictions: "Herein we present ConCISE (Consensus Classifications of In Silico Elucidations) which is the first tool to fuse molecular networking, spectral library matching and in silico class predictions"
- [abstract] By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] Superclass percent consensus (optional; default = 50): "Superclass percent consensus (optional; default = 50)"
- [readme] python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology (superclass, class, subclass): "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"

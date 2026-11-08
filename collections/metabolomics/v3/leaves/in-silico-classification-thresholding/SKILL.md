---
name: in-silico-classification-thresholding
description: Use when propagating ClassyFire ChemOnt classifications across GNPS molecular
  networking subnetworks by fusing library spectral matches with CANOPUS in silico
  predictions and applying per-level consensus thresholds.
when_to_use_negative:
- Subnetworks contain no library spectral matches (singleton nodes without library
  matches receive no in silico annotation, so consensus propagation cannot be applied)
- You need compound-level structural assignments rather than per-subnetwork chemical-class
  consensus — this skill propagates ChemOnt class annotations, not structures
- You require NPClassifier ontologies at different thresholds — the 50/70/70 defaults
  were established for ClassyFire ChemOnt levels
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Runs the consensus annotation propagation workflow across subnetworks via
    conciseCLI.py
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking and spectral library match annotations
    used for consensus tallies
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides in silico ClassyFire ChemOnt class predictions (canopus_summary.tsv)
    for subnetwork features
- name: ClassyFire
  role: Defines the superclass/class/subclass ChemOnt hierarchical ontology on which
    the 50/70/70 thresholds operate
- name: NPClassifier
  role: Alternative ontology that can replace ClassyFire for both library and in silico
    matches via the NPC argument
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:c87e4a7ea79e1c6945ac8b054d7da549f0686cb231c7365aeed7f7b6ada19300
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# in-silico-classification-thresholding

## Summary

Reconstructs ConCISE's consensus rule for propagating chemical-class annotations across a molecular subnetwork: superclass, class, and subclass percent-consensus cutoffs of 50/70/70 select the most granular ChemOnt level that passes, which is then exported with its hierarchy and consensus score.

## When to use

Use when propagating ClassyFire ChemOnt classifications across GNPS molecular networking subnetworks by fusing library spectral matches with CANOPUS in silico predictions and applying per-level consensus thresholds.

## When NOT to use

- Subnetworks contain no library spectral matches (singleton nodes without library matches receive no in silico annotation, so consensus propagation cannot be applied)
- You need compound-level structural assignments rather than per-subnetwork chemical-class consensus — this skill propagates ChemOnt class annotations, not structures
- You require NPClassifier ontologies at different thresholds — the 50/70/70 defaults were established for ClassyFire ChemOnt levels

## Inputs

- GNPS task ID or spectral library match (DBResult) file with superclass/class/subclass columns
- CANOPUS canopus_summary.tsv file of in silico class predictions
- GNPS Node_info.tsv molecular networking file
- Consensus thresholds (superclass, class, subclass; defaults 50, 70, 70)
- NPClassifier toggle (True/False)

## Outputs

- ConCISEConsensus.csv with one row per ion feature containing consensus annotation, ChemOnt hierarchy, percent-consensus score, level reached, node count, and source (library or in silico)

## How to apply

Group ion features into molecular networking subnetworks from GNPS Node_info and restrict propagation to subnetworks containing at least one library spectral match; singleton nodes without library matches receive no in silico annotation. For each subnetwork, tally agreement of ClassyFire ChemOnt annotations among library matches and CANOPUS putative annotations at superclass, class, and subclass levels. Apply the consensus cutoffs — superclass >= 50%, class >= 70%, subclass >= 70% — and select the lowest (most granular) level passing its cutoff together with all higher levels as the subnetwork's consensus annotation. Export per-feature consensus records to ConCISEConsensus.csv including hierarchy, percent-consensus score, level reached, node count, and source (library or in silico), using CLI defaults with NPClassifier disabled. Note that GNPS no longer supplies ClassyFire ontology information for library matches, so library data must be patched via the Fiehn Labs classyfire Batch service or NPClassifier ontologies must be substituted.

## Related tools

- **ConCISE** (Runs the consensus annotation propagation workflow across subnetworks via conciseCLI.py) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking and spectral library match annotations used for consensus tallies) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides in silico ClassyFire ChemOnt class predictions (canopus_summary.tsv) for subnetwork features)
- **ClassyFire** (Defines the superclass/class/subclass ChemOnt hierarchical ontology on which the 50/70/70 thresholds operate)
- **NPClassifier** (Alternative ontology that can replace ClassyFire for both library and in silico matches via the NPC argument)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- ConCISEConsensus.csv exists and parses with the required columns (featureNumber, hierarchy, score, level, node count, source)
- Every exported consensus score satisfies its level's cutoff: superclass >= 50%, class >= 70%, subclass >= 70% percent consensus
- The consensus level exported is the lowest (most granular) level passing its cutoff, together with all higher hierarchy levels
- No consensus annotation is emitted for singleton nodes lacking a library spectral match
- Subnetworks lacking any library-match annotation are excluded from propagation

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; users must patch InChiKeys through Fiehn Labs' classyfire Batch service or switch to NPClassifier ontologies
- Singleton nodes without a library match receive no in silico annotation, so they cannot gain consensus classes
- Annotation propagation is limited to structural classes identical for the majority of ion features within a subnetwork, so heterogeneous subnetworks fall back to less granular levels

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology (superclass, class, subclass): "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [methods] filter-step defaults: "Superclass percent consensus (optional; default = 50)"
- [methods] consensus mechanism: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches and in silico putative annotations."
- [readme] GNPS ClassyFire gap workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [abstract] propagation restriction: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"

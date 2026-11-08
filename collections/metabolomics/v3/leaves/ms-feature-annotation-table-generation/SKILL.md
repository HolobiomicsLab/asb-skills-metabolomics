---
name: ms-feature-annotation-table-generation
description: Use when you have a GNPS feature-based molecular networking task with
  CANOPUS in silico class predictions (canopus_summary.tsv) and node/library-match
  info (Node_info.tsv), and need per-feature chemical consensus annotations.
when_to_use_negative:
- The question is about compound-level structural elucidation or MS/MS spectrum identification
  rather than propagating class-level annotations across a molecular network — ConCISE
  annotates subnetworks with ChemOnt classes, not individual structures.
- Subnetworks contain no GNPS library spectral matches — propagation is restricted
  to subnetworks with at least one library match, and singleton nodes without library
  matches receive no in silico annotation.
- You require NPClassifier-based natural-product ontologies only for in silico matches
  but ClassyFire for library matches — the NPC flag applies NPClassifier ontologies
  to BOTH library matches and in silico matches.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Primary tool that computes consensus ClassyFire/NPClassifier annotations across
    subnetworks and writes ConCISEConsensus.csv via CLI or GUI
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking and spectral library matches (Node_info.tsv)
    whose ChemOnt annotations are tallied for consensus
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides the in silico class predictions (canopus_summary.tsv) fused with
    library matches for consensus classification
  repo: https://bio.informatik.uni-jena.de/software/
- name: ClassyFire
  role: Defines the ChemOnt hierarchical ontology (superclass, class, subclass) used
    for consensus levels and cutoffs
- name: NPClassifier
  role: Alternative ontology set usable in place of ClassyFire via the NPC argument
    for both library and in silico matches
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:0fe8f9ed701909418cdf9cdaf0792a8e532bfb2f4b4ea7f8e482ccd49cc1a183
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ms-feature-annotation-table-generation

## Summary

Generates a consensus chemical-annotation table for ion features in untargeted tandem MS by fusing GNPS molecular networking, spectral library matching, and CANOPUS in silico class predictions across the ClassyFire ChemOnt hierarchy (superclass/class/subclass). Rationale: propagating only annotations that a majority of a subnetwork agrees on yields reliable class assignments (TPR > 95%) and expands consistent feature annotation up to 76%.

## When to use

Use when you have a GNPS feature-based molecular networking task with CANOPUS in silico class predictions (canopus_summary.tsv) and node/library-match info (Node_info.tsv), and need per-feature chemical consensus annotations.

## When NOT to use

- The question is about compound-level structural elucidation or MS/MS spectrum identification rather than propagating class-level annotations across a molecular network — ConCISE annotates subnetworks with ChemOnt classes, not individual structures.
- Subnetworks contain no GNPS library spectral matches — propagation is restricted to subnetworks with at least one library match, and singleton nodes without library matches receive no in silico annotation.
- You require NPClassifier-based natural-product ontologies only for in silico matches but ClassyFire for library matches — the NPC flag applies NPClassifier ontologies to BOTH library matches and in silico matches.

## Inputs

- GNPS task ID or spectral library match file (DBResult with superclass/class/subclass ontology columns)
- CANOPUS in silico class prediction file (canopus_summary.tsv from SIRIUS/CANOPUS)
- GNPS networking info file (Node_info.tsv from feature-based molecular networking)
- Optional parameters: export directory, superclass/class/subclass percent-consensus cutoffs (defaults 50/70/70), NPClassifier on/off flag

## Outputs

- ConCISEConsensus.csv with one row per ion feature: featureNumber, consensus annotation, hierarchy, percent-consensus score, level reached (superclass/class/subclass), node count, and source (library or in silico)

## How to apply

Load the GNPS networking/library-match file (Node_info.tsv) and the CANOPUS canopus_summary.tsv, group ion features into molecular networking subnetworks, and restrict propagation to subnetworks containing at least one library spectral match; singleton nodes without library matches receive no in silico annotation. For each subnetwork, tally agreement of ClassyFire ChemOnt annotations among library matches and CANOPUS putative annotations at superclass, class, and subclass levels. Apply consensus cutoffs — superclass >= 50%, class >= 70%, subclass >= 70% (the CLI defaults) — and select the lowest (most granular) level passing its cutoff together with all higher levels as the consensus annotation. Export one row per feature with the consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico). Note: GNPS has stopped supplying ClassyFire ontology information for library matches, so either switch to NPClassifier (NPC argument True) or manually re-annotate library matches via the Fiehn Labs classyfire Batch service and merge results back with superclass/class/subclass columns.

## Related tools

- **ConCISE** (Primary tool that computes consensus ClassyFire/NPClassifier annotations across subnetworks and writes ConCISEConsensus.csv via CLI or GUI) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking and spectral library matches (Node_info.tsv) whose ChemOnt annotations are tallied for consensus) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides the in silico class predictions (canopus_summary.tsv) fused with library matches for consensus classification) — https://bio.informatik.uni-jena.de/software/
- **ClassyFire** (Defines the ChemOnt hierarchical ontology (superclass, class, subclass) used for consensus levels and cutoffs)
- **NPClassifier** (Alternative ontology set usable in place of ClassyFire via the NPC argument for both library and in silico matches)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- ConCISEConsensus.csv exists and parses with the required columns (featureNumber, hierarchy, score, level, node count, source)
- Consensus percent scores satisfy the applied cutoffs: >= 50% at superclass, >= 70% at class, >= 70% at subclass
- The level recorded is the lowest (most granular) ChemOnt level passing its cutoff, and all higher hierarchy levels are also populated
- Singleton nodes without library matches have no in silico annotation, and no subnetwork lacking a library match received propagated annotations
- Source field correctly distinguishes library spectral matches from in silico (CANOPUS) putative annotations

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch service and merging with correct column names (superclass, class, subclass)
- The Mac GUI is deprecated; Mac users must use the CLI or the myBinder notebook
- Subclass-level consensus requires 70% agreement, so very heterogeneous subnetworks may only reach superclass (50%) level or no consensus at all

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology (superclass, class, subclass): "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [methods] consensus thresholds and granularity rule: "Superclass percent consensus (optional; default = 50)"
- [readme] CLI invocation example: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [readme] GNPS ClassyFire gap workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."

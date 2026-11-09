---
name: chemont-hierarchy-consensus-annotation
description: Use when you have GNPS feature-based molecular networking output (Node_info.tsv,
  spectral library matches) plus CANOPUS in silico class predictions (canopus_summary.tsv)
  and need to annotate ion features lacking library hits via subnetwork consensus.
when_to_use_negative:
- Subnetworks (especially singletons) contain no library spectral match, since no
  in silico annotation is assigned to singleton nodes without a library match
- You need compound-level structural identifications rather than hierarchical chemical-class
  consensus annotations
- GNPS library matches lack ChemOnt columns and you cannot supply them — in that case
  the tool requires NPClassifier ontologies or manually merged ClassyFire data, which
  is a distinct setup
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: 'Implements the consensus annotation propagation: computes ChemOnt agreement
    within subnetworks, applies the 50/70/70 percent-cutoffs, and exports ConCISEConsensus.csv
    via its CLI or GUI'
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking and spectral library matches (Node_info.tsv,
    DBResult) whose ChemOnt annotations are tallied for consensus
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides in silico class predictions (canopus_summary.tsv) via SIRUS/CANOPUS
    that are combined with library matches in the consensus tally
- name: ClassyFire
  role: Defines the ChemOnt hierarchical ontology (superclass, class, subclass) used
    for both library and in silico annotations
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:650a9cb39ceee2c83c0fdfc9d2cd597956cffe7a59bf34c1127835ddd6ea67f9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# chemont-hierarchy-consensus-annotation

## Summary

Propagate consensus chemical annotations across molecular networking subnetworks by tallying ClassyFire ChemOnt agreement among GNPS spectral library matches and CANOPUS in silico predictions, applying level-specific percent-consensus cutoffs (superclass 50%, class 70%, subclass 70%) as implemented in ConCISE.

## When to use

Use when you have GNPS feature-based molecular networking output (Node_info.tsv, spectral library matches) plus CANOPUS in silico class predictions (canopus_summary.tsv) and need to annotate ion features lacking library hits via subnetwork consensus.

## When NOT to use

- Subnetworks (especially singletons) contain no library spectral match, since no in silico annotation is assigned to singleton nodes without a library match
- You need compound-level structural identifications rather than hierarchical chemical-class consensus annotations
- GNPS library matches lack ChemOnt columns and you cannot supply them — in that case the tool requires NPClassifier ontologies or manually merged ClassyFire data, which is a distinct setup

## Inputs

- GNPS task ID or GNPS spectral library match file (DBResult)
- CANOPUS in silico class prediction summary file (canopus_summary.tsv, from SIRIUS/CANOPUS)
- GNPS networking info file (Node_info.tsv) with subnetwork/feature groupings and library-match flags

## Outputs

- ConCISEConsensus.csv with one row per feature including consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico)
- Per-dataset consensus annotation rate (e.g., fraction of ion features annotated, such as 58.6%, 56.3%, 76.5% in the case studies)

## How to apply

Group ion features into GNPS feature-based molecular networking subnetworks and restrict propagation to subnetworks containing at least one library spectral match, discarding singletons without library hits. For each subnetwork, tally agreement of ClassyFire ChemOnt annotations among library matches and CANOPUS putative annotations at superclass, class, and subclass levels. Apply the consensus cutoffs — superclass >= 50%, class >= 70%, subclass >= 70% — and select the lowest (most granular) level that passes its cutoff, together with all higher levels, as the consensus annotation. Export per-feature results to ConCISEConsensus.csv with the consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico). Rationale: limiting propagation to classes identical for a majority of ion features within a subnetwork preserves a true positive rate above 95% across all ChemOnt levels.

## Related tools

- **ConCISE** (Implements the consensus annotation propagation: computes ChemOnt agreement within subnetworks, applies the 50/70/70 percent-cutoffs, and exports ConCISEConsensus.csv via its CLI or GUI) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking and spectral library matches (Node_info.tsv, DBResult) whose ChemOnt annotations are tallied for consensus) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides in silico class predictions (canopus_summary.tsv) via SIRUS/CANOPUS that are combined with library matches in the consensus tally)
- **ClassyFire** (Defines the ChemOnt hierarchical ontology (superclass, class, subclass) used for both library and in silico annotations)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- ConCISEConsensus.csv exists and parses with the required columns (featureNumber, hierarchy, score, level, node count, source)
- Every reported consensus score satisfies its level's percent cutoff: superclass >= 50%, class >= 70%, subclass >= 70%
- Level reached is consistent with the score: no consensus is exported below the most granular level that passed its cutoff, and higher-level levels are always included
- Singleton nodes without library matches receive no consensus annotation (absent or empty rows in the output)
- Resulting annotation rates expand coverage over library matching alone (case-study rates were 58.6%, 56.3%, 76.5%, with expansion up to 1244%); true positive rate should remain >95% across ChemOnt levels

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC argument True) or manually merging ClassyFire batch results (from Fiehn Labs classyfire Batch, keyed on InChiKey) back into the GNPS DBResult file with superclass/class/subclass column names
- The Mac GUI is deprecated; use the CLI or MyBinder notebook on macOS
- Propagation only occurs within subnetworks containing at least one library match; singleton nodes without a library match receive no in silico annotation

## Evidence

- [methods] ConCISE requires 50% superclass consensus in a subnetwork and 70% at class and subclass levels, selecting the lowest (most granular) level that passes cutoffs: [evidence span withheld — anchor status: not_found]
- [readme] Consensus cutoff defaults in CLI arguments: "Superclass percent consensus (optional; default = 50) ... Class percent consensus (optional; default = 70) ... Subclass percent consensus (optional; default = 70)"
- [abstract] True positive rate across ChemOnt levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software"
- [abstract] Restriction of propagation to majority-identical classes: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] GNPS ClassyFire supply gap and workarounds: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire"
- [methods] Consensus mechanism as fusion of networking, library, and in silico predictions: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches and in silico putative annotations."

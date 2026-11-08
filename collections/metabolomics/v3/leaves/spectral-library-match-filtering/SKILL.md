---
name: spectral-library-match-filtering
description: Use when a feature-based molecular networking GNPS job has spectral library
  matches plus a CANOPUS canopus_summary.tsv, and you need to filter/propagate only
  annotation classes agreed upon by a majority of ion features in each subnetwork.
when_to_use_negative:
- A subnetwork (e.g. an isolated singleton node) contains no GNPS library spectral
  match — consensus annotation receives no in silico annotation there and the skill
  should not be applied to that node
- The goal is per-spectrum spectral matching quality filtering (cosine score thresholds),
  not network-level consensus classification propagation across ChemOnt hierarchy
  levels
- Only a CANOPUS summary is available with no networking info file, since propagation
  is defined over subnetworks of ion features
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: 'Implements the consensus filtering: finds consensus annotations of putative
    annotations using ClassyFire ontologies and applies the 50/70/70 percent-consensus
    cutoffs per subnetwork'
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking subnetworks and spectral library
    matches (with ClassyFire ontology information) that are the substrate for filtering
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides in silico class predictions (canopus_summary.tsv) that contribute
    putative annotations to the consensus tally alongside library matches
  repo: https://bio.informatik.uni-jena.de/software/
- name: ClassyFire
  role: ChemOnt hierarchical ontology (superclass, class, subclass) at which consensus
    cutoffs are evaluated; can be manually retrieved via the Fiehn Labs classyfire
    Batch tool when GNPS columns are missing
- name: NPClassifier
  role: Optional alternative ontology (NPC argument True) used in place of ClassyFire
    for both library matches and in silico matches
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:63d92b35269311e536a764ce424ef7a3ea1f0b4cd50beec9739e8c29747043f7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectral-library-match-filtering

## Summary

Consensus-based filtering and propagation of GNPS spectral library matches and CANOPUS in silico ClassyFire annotations across molecular networking subnetworks, using ChemOnt percent-consensus cutoffs (superclass 50%, class 70%, subclass 70%) as in ConCISE.

## When to use

Use when a feature-based molecular networking GNPS job has spectral library matches plus a CANOPUS canopus_summary.tsv, and you need to filter/propagate only annotation classes agreed upon by a majority of ion features in each subnetwork.

## When NOT to use

- A subnetwork (e.g. an isolated singleton node) contains no GNPS library spectral match — consensus annotation receives no in silico annotation there and the skill should not be applied to that node
- The goal is per-spectrum spectral matching quality filtering (cosine score thresholds), not network-level consensus classification propagation across ChemOnt hierarchy levels
- Only a CANOPUS summary is available with no networking info file, since propagation is defined over subnetworks of ion features

## Inputs

- GNPS task ID or GNPS spectral library match / DBResult file (with superclass, class, subclass ChemOnt columns)
- canopus_summary.tsv (CANOPUS in silico class predictions from SIRIUS/CANOPUS)
- Node_info.tsv (GNPS feature-based molecular networking and library match info)

## Outputs

- ConCISEConsensus.csv with per-feature consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico)

## How to apply

Load the GNPS spectral library match / DBResult data (or GNPS task ID) and the Node_info.tsv networking file, then group ion features into molecular networking subnetworks and restrict propagation to subnetworks containing at least one library-spectral-match annotation — singleton nodes without library matches get no annotation. For each subnetwork, tally agreement of ClassyFire ChemOnt labels among library matches and in silico (CANOPUS) putative annotations at superclass, class, and subclass levels. Apply the percent-consensus cutoffs — superclass >= 50%, class >= 70%, subclass >= 70% — and select the lowest (most granular) level that passes its cutoff together with all higher levels. Export one row per feature (ConCISEConsensus.csv) recording the consensus annotation, hierarchy, percent-consensus score, level reached, node count, and source (library or in silico). Rationale: limiting propagation to classes identical for the majority of ion features keeps the true positive rate above 95% at all ChemOnt levels.

## Related tools

- **ConCISE** (Implements the consensus filtering: finds consensus annotations of putative annotations using ClassyFire ontologies and applies the 50/70/70 percent-consensus cutoffs per subnetwork) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking subnetworks and spectral library matches (with ClassyFire ontology information) that are the substrate for filtering) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides in silico class predictions (canopus_summary.tsv) that contribute putative annotations to the consensus tally alongside library matches) — https://bio.informatik.uni-jena.de/software/
- **ClassyFire** (ChemOnt hierarchical ontology (superclass, class, subclass) at which consensus cutoffs are evaluated; can be manually retrieved via the Fiehn Labs classyfire Batch tool when GNPS columns are missing)
- **NPClassifier** (Optional alternative ontology (NPC argument True) used in place of ClassyFire for both library matches and in silico matches)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- ConCISEConsensus.csv exists, parses, and contains the required consensus/hierarchy/score/level/node-count/source columns
- Every exported consensus score satisfies its level's cutoff: superclass >= 50%, class >= 70%, subclass >= 70% percent consensus
- The exported annotation is the lowest (most granular) ChemOnt level passing its cutoff, and all higher levels of its hierarchy are also populated
- Singleton nodes without a library spectral match carry no consensus annotation
- Reported true positive rate of propagated consensus annotations is > 95% across superclass, class, and subclass levels

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC = True) or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch tool and merging superclass/class/subclass columns back into the GNPS DBResult file
- The Mac GUI is deprecated; use the CLI or myBinder notebook on macOS
- Annotation propagation is deliberately limited to structural classes identical for the majority of ion features in a subnetwork, so heterogeneous subnetworks fall back to coarser levels or fail the cutoffs

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [abstract] Annotation propagation limited to majority-identical classes: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] Consensus thresholds as CLI defaults: "Superclass percent consensus (optional; default = 50)"
- [readme] Consensus over ClassyFire ontologies supplied by GNPS: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [readme] Known issue affecting library match ontology columns: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches"

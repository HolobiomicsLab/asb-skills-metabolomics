---
name: chemont-ontology-superclass-summarization
description: Use when a GNPS FBMN task has ConCISE consensus annotations with ChemOnt
  superclass labels and you need organism- or group-level compositional summaries
  (e.g., which structural classes dominate which exudates) rather than compound-level
  IDs.
when_to_use_negative:
- When the goal is definitive molecular identification or MS/MS spectral-level identification
  rather than hierarchical ChemOnt class-level compositional profiling
- When ClassyFire ontology data are required for library matches but cannot be obtained
  — GNPS has stopped supplying classyfire ontology information for spectral library
  matches, so the NPClassifier workaround or manual Fiehn Labs classyfire Batch lookup
  must be applied first
- When ion features lack both GNPS library matches and in silico (CANOPUS/SIRIUS)
  class predictions, since consensus annotation depends on putative annotations from
  these sources
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ConCISE
  role: Fuses molecular networking, spectral library matching and in silico class
    predictions to assign consensus ChemOnt superclass/class/subclass labels to ion
    features; run via conciseCLI.py
  repo: https://github.com/Zquinlan/conCISE
- name: CANOPUS
  role: Supplies in silico structure-class predictions (canopus_summary.tsv) that
    feed the consensus classification
- name: GNPS
  role: Provides feature-based molecular networking and the Node_info.tsv file; previously
    supplied ClassyFire ontologies for spectral library matches
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: Python
  role: Parses the consensus annotation file, computes superclass relative abundances
    per organism, writes superclass_abundance_table.csv, and renders grouped bar charts
    in matplotlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:b44a65aa1346f36a16c310c458b34fa545eb8717ddcf791423db8d85e4a71ea5
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ChemOnt ontology superclass summarization

## Summary

Summarizes ion features from GNPS feature-based molecular networking into ChemOnt superclass-level categories (via ConCISE consensus annotations) and computes superclass relative-abundance profiles per sample group or organism, enabling compositional comparisons such as coral vs. turf-algae exometabolome differences.

## When to use

Use when a GNPS FBMN task has ConCISE consensus annotations with ChemOnt superclass labels and you need organism- or group-level compositional summaries (e.g., which structural classes dominate which exudates) rather than compound-level IDs.

## When NOT to use

- When the goal is definitive molecular identification or MS/MS spectral-level identification rather than hierarchical ChemOnt class-level compositional profiling
- When ClassyFire ontology data are required for library matches but cannot be obtained — GNPS has stopped supplying classyfire ontology information for spectral library matches, so the NPClassifier workaround or manual Fiehn Labs classyfire Batch lookup must be applied first
- When ion features lack both GNPS library matches and in silico (CANOPUS/SIRIUS) class predictions, since consensus annotation depends on putative annotations from these sources

## Inputs

- GNPS feature-based molecular networking task ID (e.g., MSV000082083 task) or spectral library match file
- CANOPUS canopus_summary.tsv (in silico structure-class predictions from SIRIUS/CANOPUS)
- GNPS Node_info.tsv networking file mapping ion features to molecular subnetworks
- Mapping of ion features to source organism or sample group (e.g., coral vs. turf-algae exudate samples)

## Outputs

- ConCISE consensus annotation file with ChemOnt superclass/class/subclass labels per ion feature
- Superclass-by-organism relative abundance matrix (superclass_abundance_table.csv)
- Grouped bar chart of superclass relative abundances per organism (figure3b_superclass_bars.png)

## How to apply

Run the ConCISE CLI (conciseCLI.py) on the GNPS feature-based molecular networking task together with the CANOPUS canopus_summary.tsv and GNPS Node_info.tsv file, using ClassyFire ChemOnt ontologies at the default consensus thresholds (superclass 50%, class 70%, subclass 70%). Parse the exported consensus annotation file and extract the ChemOnt superclass label assigned to each ion feature. Map each annotated feature to its source organism or sample group (e.g., Pocillopora verrucosa, Porites lobata, Dictyota ceylanica, turfing algae) and compute superclass-level relative abundances per group, writing the superclass-by-group matrix to superclass_abundance_table.csv. Because annotation propagation is limited to structural classes identical for the majority of ion features within a subnetwork, the superclass labels represent consensus classifications for entire molecular subnetworks, not just individually matched spectra. Finally, render grouped bar charts of superclass relative abundances by organism (e.g., figure3b_superclass_bars.png) and inspect directional class patterns such as organic acids enriched in coral exudates vs. organoheterocyclic compounds enriched in turfing-algae exudates.

## Related tools

- **ConCISE** (Fuses molecular networking, spectral library matching and in silico class predictions to assign consensus ChemOnt superclass/class/subclass labels to ion features; run via conciseCLI.py) — https://github.com/Zquinlan/conCISE
- **CANOPUS** (Supplies in silico structure-class predictions (canopus_summary.tsv) that feed the consensus classification)
- **GNPS** (Provides feature-based molecular networking and the Node_info.tsv file; previously supplied ClassyFire ontologies for spectral library matches) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **Python** (Parses the consensus annotation file, computes superclass relative abundances per organism, writes superclass_abundance_table.csv, and renders grouped bar charts in matplotlib)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Relative abundances per organism sum to 1 within each organism column of superclass_abundance_table.csv
- Directional superclass patterns are reproduced: organic acid superclass abundance is higher in coral (Pocillopora verrucosa, Porites lobata) exudates and organoheterocyclic compound superclass abundance is higher in turfing-algae exudates
- Consensus thresholds trace to ConCISE defaults: superclass 50%, class 70%, subclass 70%
- Superclass labels conform to the ChemOnt hierarchical ontology vocabulary (superclass, class, subclass levels)
- ConCISE reported true positive rate is greater than 95% across all ChemOnt levels, so a large share of subnetwork annotations should be consistent rather than conflicting

## Limitations

- GNPS has stopped supplying classyfire ontology information for spectral library matches; workarounds are switching to NPClassifier ontologies (NPC argument True) or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch identifier and merging superclass/class/subclass columns back into the GNPS DBResult file
- Annotation propagation is deliberately limited to structural classes identical for the majority of ion features within a subnetwork, so heterogeneous subnetworks may remain unannotated
- The annotation proportion ceiling is ~76% of ion features reliably and consistently annotated; unannotated features are excluded from superclass abundance profiles

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software"
- [abstract] ConCISE expanded annotation up to 76%: "The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%"
- [abstract] Annotation propagation limited to majority-identical structural classes in a subnetwork: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [methods] Superclass percent consensus default 50: "Superclass percent consensus (optional; default = 50)"
- [readme] ConCISE finds consensus annotations using ClassyFire ontologies supplied by GNPS: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [readme] GNPS no longer supplies classyfire ontology information for library matches: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."

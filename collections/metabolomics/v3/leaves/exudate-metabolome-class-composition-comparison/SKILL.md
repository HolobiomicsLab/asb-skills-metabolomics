---
name: exudate-metabolome-class-composition-comparison
description: Use when comparing chemical class/superclass composition of ion features
  across organism exudates or sample groups, e.g., testing whether coral vs algal
  exometabolomes differ in organic acid vs organoheterocyclic compound abundance.
when_to_use_negative:
- Input is already a curated feature table with chemical class annotations — consensus
  propagation via ConCISE is unnecessary and the workflow should start at the abundance
  computation step instead
- The analysis targets individual metabolite identifications or spectral library matches
  rather than superclass-level compositional comparison across organism groups
- No CANOPUS/in silico predictions or molecular networking outputs are available,
  so ConCISE cannot form consensus annotations
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ConCISE
  role: Fuses molecular networking, spectral library matching, and in silico class
    predictions into consensus ChemOnt annotations for ion features via conciseCLI.py
  repo: https://github.com/Zquinlan/conCISE
- name: CANOPUS
  role: Supplies in silico structural class predictions (canopus_summary.tsv) that
    ConCISE fuses with networking and library matches
- name: GNPS
  role: Provides the feature-based molecular networking task and Node_info.tsv networking
    file consumed by ConCISE
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: Python
  role: Runs conciseCLI.py and performs superclass abundance computation and matplotlib
    grouped bar chart rendering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:2af8f597f39161dd908c1b0ee404aec1c4d8c6b7742e6dae1151da94ed504fc9
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
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# exudate-metabolome-class-composition-comparison

## Summary

Compares ChemOnt superclass-level composition of exometabolomes (exudates) across organisms by running ConCISE consensus annotation on a GNPS feature-based molecular networking task plus CANOPUS output, then computing and visualizing per-organism superclass relative abundances (e.g., corals enriched in organic acids vs turfing algae enriched in organoheterocyclic compounds).

## When to use

Use when comparing chemical class/superclass composition of ion features across organism exudates or sample groups, e.g., testing whether coral vs algal exometabolomes differ in organic acid vs organoheterocyclic compound abundance.

## When NOT to use

- Input is already a curated feature table with chemical class annotations — consensus propagation via ConCISE is unnecessary and the workflow should start at the abundance computation step instead
- The analysis targets individual metabolite identifications or spectral library matches rather than superclass-level compositional comparison across organism groups
- No CANOPUS/in silico predictions or molecular networking outputs are available, so ConCISE cannot form consensus annotations

## Inputs

- GNPS feature-based molecular networking task ID or spectral library match file (e.g., MSV000082083 task with Node_info.tsv)
- CANOPUS canopus_summary.tsv from SIRIUS/CANOPUS in silico class predictions
- GNPS Node_info.tsv networking information file
- Feature-to-organism mapping assigning each ion feature to its exudate source (coral vs algal species)

## Outputs

- ConCISE consensus annotation file with ChemOnt superclass/class/subclass labels per ion feature
- superclass_abundance_table.csv: superclass-by-organism relative abundance matrix (columns sum to 1 per organism)
- Grouped bar chart of superclass relative abundances by organism (e.g., figure3b_superclass_bars.png)

## How to apply

Run the ConCISE CLI (conciseCLI.py) on a GNPS feature-based molecular networking task (e.g., MSV000082083) together with the CANOPUS canopus_summary.tsv and the GNPS Node_info.tsv networking file, using ClassyFire ChemOnt ontologies at the default consensus thresholds (superclass 50%, class 70%, subclass 70%; NPC=False unless switching to NPClassifier). Parse the exported consensus annotation file and extract the ChemOnt superclass label assigned to each ion feature. Map each annotated ion feature to its exudate source organism (e.g., Pocillopora verrucosa, Porites lobata, Dictyota ceylanica, turfing algae) and compute superclass-level relative abundances per organism, writing the superclass-by-organism matrix to superclass_abundance_table.csv. Render a grouped bar chart of superclass relative abundances by organism in matplotlib (e.g., figure3b_superclass_bars.png) and check for the expected directional patterns, since ConCISE propagates annotations to subnetworks whose majority of ion features share identical structural classes (>95% true positive rate across superclass, class, subclass).

## Related tools

- **ConCISE** (Fuses molecular networking, spectral library matching, and in silico class predictions into consensus ChemOnt annotations for ion features via conciseCLI.py) — https://github.com/Zquinlan/conCISE
- **CANOPUS** (Supplies in silico structural class predictions (canopus_summary.tsv) that ConCISE fuses with networking and library matches)
- **GNPS** (Provides the feature-based molecular networking task and Node_info.tsv networking file consumed by ConCISE) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **Python** (Runs conciseCLI.py and performs superclass abundance computation and matplotlib grouped bar chart rendering)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Relative abundances per organism sum to 1 within each organism column of superclass_abundance_table.csv
- Organic acid superclass relative abundance is higher in coral exudates (Pocillopora verrucosa, Porites lobata) than in algal exudates
- Organoheterocyclic compound superclass relative abundance is higher in turfing algae (and Dictyota ceylanica) exudates than in coral exudates
- Consensus thresholds trace to ConCISE defaults: superclass 50%, class 70%, subclass 70%
- ConCISE true positive rate expectation of >95% across superclass/class/subclass levels holds for the propagated annotations

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are to use NPClassifier (NPC=True) or manually pull ClassyFire data via the Fiehn Labs classyfire Batch service using InChiKeys from the GNPS DBResult file and merge back with correct superclass/class/subclass column names
- ConCISE Mac GUI is deprecated; on macOS use the CLI or myBinder interface
- Annotation propagation only applies to structural classes identical for the majority of ion features within a subnetwork, so subnetworks without a majority consensus receive no propagated class

## Evidence

- [readme] CLI consensus thresholds: "Superclass percent consensus (optional; default = 50) ... Class percent consensus (optional; default = 70) ... Subclass percent consensus (optional; default = 70)"
- [readme] CLI invocation pattern: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [abstract] Consensus mechanism: "the first tool to fuse molecular networking, spectral library matching and in silico class predictions to establish accurate putative classifications for entire subnetworks"
- [abstract] Validation performance: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [readme] Known GNPS workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire"

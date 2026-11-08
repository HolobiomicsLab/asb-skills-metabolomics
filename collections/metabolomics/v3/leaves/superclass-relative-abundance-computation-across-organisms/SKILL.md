---
name: superclass-relative-abundance-computation-across-organisms
description: Use when a ConCISE consensus annotation file (from a GNPS feature-based
  molecular networking task plus CANOPUS canopus_summary.tsv) must be summarized into
  superclass-relative-abundance profiles that are compared across organisms or sample
  sources.
when_to_use_negative:
- The input is already a feature/abundance table with superclass assignments — no
  consensus annotation step is needed
- No GNPS molecular networking task and CANOPUS canopus_summary.tsv exist for the
  dataset, so ConCISE consensus annotations cannot be generated
- The goal is compound-level structural elucidation rather than superclass-level compositional
  profiling across organisms
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Produces the consensus annotation file whose superclass labels per ion feature
    are aggregated into relative abundances; run via conciseCLI.py on the GNPS task
    with CANOPUS and Node_info inputs
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies the feature-based molecular networking task and Node_info.tsv used
    to map ion features into subnetworks and organisms
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides in silico class predictions (canopus_summary.tsv) that feed the ClassyFire/ChemOnt
    consensus used to assign superclass labels
- name: ClassyFire
  role: Defines the ChemOnt hierarchical ontology (superclass, class, subclass) under
    which superclass relative abundances are computed
- name: Python
  role: Parses the consensus export, computes superclass relative abundances per organism,
    and renders the grouped bar chart in matplotlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:2396b19d64a44dbb7474ef43cbce4e463d1f3568237325f169756c63132d496a
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
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# superclass-relative-abundance-computation-across-organisms

## Summary

Compute ChemOnt superclass-level relative abundances of ion features per organism from a ConCISE consensus annotation export, to test whether exometabolome composition (e.g., organic acids vs organoheterocyclic compounds) differs between source organisms such as corals and turfing algae.

## When to use

Use when a ConCISE consensus annotation file (from a GNPS feature-based molecular networking task plus CANOPUS canopus_summary.tsv) must be summarized into superclass-relative-abundance profiles that are compared across organisms or sample sources.

## When NOT to use

- The input is already a feature/abundance table with superclass assignments — no consensus annotation step is needed
- No GNPS molecular networking task and CANOPUS canopus_summary.tsv exist for the dataset, so ConCISE consensus annotations cannot be generated
- The goal is compound-level structural elucidation rather than superclass-level compositional profiling across organisms

## Inputs

- ConCISE consensus annotation export (superclass label per ion feature)
- GNPS feature-based molecular networking task (e.g., MSV000082083)
- CANOPUS canopus_summary.tsv
- GNPS Node_info.tsv networking file
- Mapping of ion features to exudate source organism

## Outputs

- superclass_abundance_table.csv (superclass-by-organism relative abundance matrix)
- Grouped bar chart of superclass relative abundances by organism (e.g., figure3b_superclass_bars.png)

## How to apply

Run the ConCISE CLI (conciseCLI.py) on the GNPS FBMN task together with the CANOPUS canopus_summary.tsv and Node_info.tsv networking file using ClassyFire ontologies at the default consensus thresholds (superclass 50%, class 70%, subclass 70%). Parse the exported consensus annotation file and extract the ChemOnt superclass label assigned to each ion feature. Map each annotated ion feature to its source organism (e.g., Pocillopora verrucosa, Porites lobata, Dictyota ceylanica, turfing algae) and compute superclass-level relative abundances per organism. Write the superclass-by-organism matrix to superclass_abundance_table.csv and render a grouped bar chart (e.g., figure3b_superclass_bars.png) in matplotlib. The rationale is that ConCISE propagation limits annotations to structural classes identical for the majority of ion features within a subnetwork, making superclass-level totals per organism a reliable basis for compositional comparison.

## Related tools

- **ConCISE** (Produces the consensus annotation file whose superclass labels per ion feature are aggregated into relative abundances; run via conciseCLI.py on the GNPS task with CANOPUS and Node_info inputs) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies the feature-based molecular networking task and Node_info.tsv used to map ion features into subnetworks and organisms) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides in silico class predictions (canopus_summary.tsv) that feed the ClassyFire/ChemOnt consensus used to assign superclass labels)
- **ClassyFire** (Defines the ChemOnt hierarchical ontology (superclass, class, subclass) under which superclass relative abundances are computed)
- **Python** (Parses the consensus export, computes superclass relative abundances per organism, and renders the grouped bar chart in matplotlib)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Relative abundances per organism sum to 1 within each organism column of superclass_abundance_table.csv
- Organic acid superclass abundance is higher in coral (Pocillopora verrucosa / Porites lobata) exudates
- Organoheterocyclic compound superclass abundance is higher in turfing-algae exudates
- QC thresholds trace to ConCISE consensus defaults: superclass 50%, class 70%, subclass 70%
- Both output artifacts exist: superclass_abundance_table.csv and the rendered bar chart PNG

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier ontologies (NPC argument True) or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch using InChiKeys from the GNPS DBResult file
- Annotation propagation only occurs for structural classes identical for the majority of ion features within a subnetwork, so unannotated features reduce coverage of the abundance matrix
- The Mac GUI is deprecated; use the CLI or MyBinder interface on macOS

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology: [evidence span withheld — anchor status: not_found]
- [methods] ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [methods] Superclass percent consensus default is 50: "Superclass percent consensus (optional; default = 50)"
- [abstract] Annotation propagation limited to majority-identical classes: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] GNPS no longer supplies classyfire ontology information: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."

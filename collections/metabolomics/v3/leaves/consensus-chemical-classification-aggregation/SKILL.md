---
name: consensus-chemical-classification-aggregation
description: Use when a GNPS FBMN task plus a CANOPUS canopus_summary.tsv must be
  converted into subnetwork-level consensus superclass/class/subclass annotations
  (ChemOnt) to compare compositional profiles (e.g., organic acids vs organoheterocycles)
  across organisms or treatments.
when_to_use_negative:
- Input is only a standalone MS/MS spectral library match with no molecular networking
  context — ConCISE's value is annotation propagation across subnetworks, not single-spectrum
  classification.
- ClassyFire ontology columns are missing from GNPS library matches and neither workaround
  (NPClassifier flag or manual Fiehn Labs classyfire Batch lookup merged into the
  DBResult file) has been applied.
- The analysis requires biological-role/chemical-diversity vocabulary rather than
  the ChemOnt superclass/class/subclass hierarchy used by ConCISE.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ConCISE
  role: 'Core tool: finds consensus annotations of putative annotations using ClassyFire
    ontologies and propagates them across subnetworks; provides CLI, GUI, and MyBinder
    interfaces'
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking output (task ID, Node_info.tsv)
    and spectral library matches that anchor consensus propagation
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: In silico class predictor whose canopus_summary.tsv provides structural class
    predictions fused with library matches into the consensus
- name: SIRIUS
  role: In silico tool providing structural annotations that feed the consensus classification
    pipeline
- name: ClassyFire
  role: ChemOnt hierarchical ontology (superclass, class, subclass) used as the classification
    vocabulary for consensus assignments
- name: Python
  role: Runtime for conciseCLI.py and downstream parsing/aggregation (e.g., superclass_abundance_table.csv,
    matplotlib bar chart)
  repo: https://github.com/zquinlan/concise/releases
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:16ffa779da028684c369c8e0db70cd1a01a6b83c44698f1a6587201283087d4a
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# consensus-chemical-classification-aggregation

## Summary

Aggregates putative chemical classifications across a GNPS feature-based molecular network by fusing spectral library matches (ClassyFire ChemOnt ontologies) with in silico class predictions from SIRIUS/CANOPUS, propagating consensus annotations to entire subnetworks and expanding reliable ion-feature annotation to ~76% of features.

## When to use

Use when a GNPS FBMN task plus a CANOPUS canopus_summary.tsv must be converted into subnetwork-level consensus superclass/class/subclass annotations (ChemOnt) to compare compositional profiles (e.g., organic acids vs organoheterocycles) across organisms or treatments.

## When NOT to use

- Input is only a standalone MS/MS spectral library match with no molecular networking context — ConCISE's value is annotation propagation across subnetworks, not single-spectrum classification.
- ClassyFire ontology columns are missing from GNPS library matches and neither workaround (NPClassifier flag or manual Fiehn Labs classyfire Batch lookup merged into the DBResult file) has been applied.
- The analysis requires biological-role/chemical-diversity vocabulary rather than the ChemOnt superclass/class/subclass hierarchy used by ConCISE.

## Inputs

- GNPS feature-based molecular networking task ID (or a GNPS spectral library match / DBResult file with superclass, class, subclass columns)
- CANOPUS canopus_summary.tsv (in silico class predictions from SIRIUS/CANOPUS)
- GNPS Node_info.tsv networking information file
- Consensus thresholds: superclass 50%, class 70%, subclass 70% (defaults)
- Ontology flag: ClassyFire (False) or NPClassifier (True)

## Outputs

- Consensus annotation file assigning ChemOnt superclass/class/subclass labels per ion feature
- Superclass-by-organism relative abundance matrix (columns sum to 1)
- Downstream figures, e.g., grouped bar chart of superclass relative abundances per organism (figure3b_superclass_bars.png)

## How to apply

Run ConCISE (CLI conciseCLI.py or MyBinder notebook) on a GNPS feature-based molecular networking task ID together with the CANOPUS canopus_summary.tsv and the GNPS Node_info.tsv networking file. Use the default consensus percent thresholds — superclass 50%, class 70%, subclass 70% — and propagate annotations only to structural classes identical for the majority of ion features within a subnetwork. Choose ClassyFire (NPC=False) or NPClassifier (NPC=True) ontologies; because GNPS has stopped supplying ClassyFire ontology information for spectral library matches, either switch to NPClassifier or manually pull ClassyFire data for the library-match InChiKeys via the Fiehn Labs classyfire Batch service and merge superclass/class/subclass columns back into the GNPS DBResult file. Parse the exported consensus annotation file to extract the ChemOnt superclass per ion feature, then map features to sample groups (e.g., coral vs turfing-algae exudates) and compute superclass-level relative abundances (each organism column sums to 1). Note that ConCISE maintains a true positive rate >95% across ChemOnt levels, so consensus assignments can be trusted as propagation-ready classifications.

## Related tools

- **ConCISE** (Core tool: finds consensus annotations of putative annotations using ClassyFire ontologies and propagates them across subnetworks; provides CLI, GUI, and MyBinder interfaces) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking output (task ID, Node_info.tsv) and spectral library matches that anchor consensus propagation) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (In silico class predictor whose canopus_summary.tsv provides structural class predictions fused with library matches into the consensus)
- **SIRIUS** (In silico tool providing structural annotations that feed the consensus classification pipeline)
- **ClassyFire** (ChemOnt hierarchical ontology (superclass, class, subclass) used as the classification vocabulary for consensus assignments)
- **Python** (Runtime for conciseCLI.py and downstream parsing/aggregation (e.g., superclass_abundance_table.csv, matplotlib bar chart)) — https://github.com/zquinlan/concise/releases

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Consensus file assigns a ChemOnt superclass label to a large fraction of ion features; paper benchmark is expansion of reliable annotation up to 76% of features.
- Relative abundances per organism sum to 1 within each organism column of superclass_abundance_table.csv.
- Directional pattern reproduced: organic acid superclass enriched in coral exudates (Pocillopora verrucosa, Porites lobata) and organoheterocyclic compounds enriched in turfing-algae exudates.
- QC thresholds trace to ConCISE consensus defaults: superclass 50%, class 70%, subclass 70%.
- True-positive-rate benchmark: ConCISE maintains >95% TPR across superclass, class, and subclass levels of the ChemOnt ontology.

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are using NPClassifier ontologies or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch and merging superclass/class/subclass columns into the GNPS DBResult file.
- The Mac GUI is deprecated; on macOS use the command line or MyBinder interface.
- Annotation propagation is deliberately restricted to structural classes identical for the majority of ion features within a subnetwork, so heterogeneous subnetworks may remain unannotated.

## Evidence

- [abstract] first tool to fuse molecular networking, spectral library matching and in silico class predictions: "Herein we present ConCISE (Consensus Classifications of In Silico Elucidations) which is the first tool to fuse molecular networking, spectral library matching and in silico class predictions"
- [abstract] >95% true positive rate across ChemOnt levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software"
- [methods] consensus mechanism: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches"
- [methods] default consensus thresholds: "Superclass percent consensus (optional; default = 50)"
- [readme] GNPS ClassyFire gap and workarounds: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [readme] CLI example with arguments: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"

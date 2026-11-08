---
name: gene-identifier-mapping
description: Use when feature names from CCLE omics layers (e.g. RNA splicing Ensemble
  IDs) must be converted to common gene names or Bigg IDs before matching against
  Recon3D reactions or SynLethDB SL gene pairs.
when_to_use_negative:
- Feature names are already common gene names and the target database (e.g. SynLethDB)
  uses the same identifier namespace, so mapping adds no resolution
- The analysis targets non-gene feature layers (e.g. metabolites as predictors) where
  a gene-name-to-BiggID translation is inapplicable
- Mutations-based feature sets excluded from the random-forest results due to their
  binary structure — mapping gene space there is moot
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3345
tools:
- name: recon_mapping (MATLAB and Python scripts)
  role: Extracts genes from Recon3D reactions involving metabolites of interest and
    matches them with top feature lists via the common-gene-name-to-BiggID csv
  repo: sriram-lab/Metab8D
- name: BioTools.fr
  role: Converts RNA splicing Ensemble IDs to common gene names prior to network matching
- name: Python
  role: Language for running the recon_mapping translation scripts and loading human_1_mapping.csv
  repo: sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:129f2f31bd36866fc4391e1c9209f5d369d923b7d4f3aee3ef81b3b7b892cc56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - BioTools.fr
  - Python
  - recon_mapping (MATLAB and Python scripts)
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-identifier-mapping

## Summary

Reconciles heterogeneous gene and metabolite identifiers across omics feature lists, the Recon3D/Human1 metabolic model, and interaction databases (SynLethDB, STRING, TarBase) so that predictor features can be matched to curated gene pairs and reactions. Without this mapping, downstream SL-enrichment and regulome-network analyses in Metab8D cannot join features to pathway entities.

## When to use

Use when feature names from CCLE omics layers (e.g. RNA splicing Ensemble IDs) must be converted to common gene names or Bigg IDs before matching against Recon3D reactions or SynLethDB SL gene pairs.

## When NOT to use

- Feature names are already common gene names and the target database (e.g. SynLethDB) uses the same identifier namespace, so mapping adds no resolution
- The analysis targets non-gene feature layers (e.g. metabolites as predictors) where a gene-name-to-BiggID translation is inapplicable
- Mutations-based feature sets excluded from the random-forest results due to their binary structure — mapping gene space there is moot

## Inputs

- Top-20 feature lists per metabolite from Metab8D_network.xlsx with confidence scores (0–8)
- recon_mapping gene-name-to-BiggID translation csv
- human_1_mapping.csv (Recon3D metabolites with mapped Human1 IDs)
- Recon3D genome-scale metabolic model reaction-gene associations
- RNA splicing feature identifiers (Ensemble IDs) requiring conversion to common names

## Outputs

- Mapped feature lists in consistent common gene name / BiggID space
- Filtered set of metabolically relevant genes present in Recon3D
- Metabolite-of-interest to reaction-gene mappings for regulome network construction
- Aligned gene sets enabling SynLethDB SL pair overlap counts and null-draw feature pools

## How to apply

Collect the top-20 feature lists per metabolite from Metab8D_network.xlsx (nine feature sets, 225 metabolites). Convert non-standard identifiers first — e.g. RNA splicing Ensemble transcript IDs to common gene names via BioTools.fr — using the gene-name-to-BiggID translation csv shipped with the recon_mapping scripts. Filter input datasets containing metabolically relevant genes so only genes present in the Recon3D genome-scale metabolic model are retained, and use human_1_mapping.csv to map Recon3D metabolites to Human1 IDs. When matching top predictor features to reactions, extract genes from reactions involving the metabolite of interest (MATLAB/Python recon_mapping scripts) and match them against the mapped feature list; this mapped gene space is also what the SynLethDB SL pair overlap and null-draw feature pools must operate on.

## Related tools

- **recon_mapping (MATLAB and Python scripts)** (Extracts genes from Recon3D reactions involving metabolites of interest and matches them with top feature lists via the common-gene-name-to-BiggID csv) — sriram-lab/Metab8D
- **BioTools.fr** (Converts RNA splicing Ensemble IDs to common gene names prior to network matching)
- **Python** (Language for running the recon_mapping translation scripts and loading human_1_mapping.csv) — sriram-lab/Metab8D

## Examples

```
import pandas as pd; mapping = pd.read_csv('recon_mapping/gene_name_to_biggid.csv'); features = pd.read_excel('Metab8D_network.xlsx'); features['bigg_id'] = features['gene'].map(mapping.set_index('common_name')['BiggID'])
```

## Evaluation signals

- Every gene feature in the mapped top-20 list resolves to a BiggID via the recon_mapping csv, or is explicitly recorded as unmapped
- Genes retained after filtering are all present in the Recon3D genome-scale metabolic model (no foreign gene symbols leak through)
- human_1_mapping.csv round-trip: Recon3D metabolite IDs map to exactly one Human1 ID with no collisions
- Downstream SL overlap counts (52 interactions, 45 metabolites) are only reproducible if both predictor features and SynLethDB pairs are in a common identifier space — mismatched namespaces should collapse the 27-fold enrichment
- Splicing-derived feature names after BioTools.fr conversion should look up against the same gene pool as transcriptomic features

## Limitations

- Mutations were excluded from random forest model results due to their binary structure, so gene mapping for that feature set is not part of the reported network
- Identifier conversion via BioTools.fr may not cover every Ensemble transcript ID, leaving splicing features unmapped
- Network associations are 'guilt by association' and descriptive — mapped gene pairs do not establish causative mechanisms

## Evidence

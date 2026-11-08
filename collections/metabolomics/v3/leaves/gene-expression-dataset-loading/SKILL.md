---
name: gene-expression-dataset-loading
description: Reference for retrieving the exact expression datasets (type 2 diabetes
  from the Whitehead deposit and airway epithelia from AGED) needed to reproduce PLAGE
  pathway-activity analyses.
when_to_use_negative:
- If the analysis goal is to rank individual genes rather than score predefined pathway
  activity (e.g. via SVD) — this skill only stages the datasets, it does not perform
  the pathway analysis itself
- If a different disease/condition dataset than type 2 diabetes or airway epithelia
  is required, since this skill only covers the two demonstration datasets used in
  the PLAGE article
tools:
- name: PLAGE
  role: Web program that consumes the loaded expression datasets and the KEGG/Biocarta
    gene-sets to compute pathway activity levels via SVD
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:f73b0cf3c6d212ae42e2aba48d265e6a97756bff731a80df72b5b36b4e609a59
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-expression-dataset-loading

## Summary

Loads the specific public microarray expression datasets used by PLAGE's demonstrations: the type 2 diabetes profiles from Mootha et al. and the airway epithelia smoker/non-smoker profiles, so that pathway activity analysis can proceed on the exact inputs.

## When to use

Reference for retrieving the exact expression datasets (type 2 diabetes from the Whitehead deposit and airway epithelia from AGED) needed to reproduce PLAGE pathway-activity analyses.

## When NOT to use

- If the analysis goal is to rank individual genes rather than score predefined pathway activity (e.g. via SVD) — this skill only stages the datasets, it does not perform the pathway analysis itself
- If a different disease/condition dataset than type 2 diabetes or airway epithelia is required, since this skill only covers the two demonstration datasets used in the PLAGE article

## Inputs

- Type 2 diabetes expression profiles (43 samples: 17 DM2, 17 NGT, 9 IGT) from the Whitehead Institute deposit
- Airway epithelia expression profiles (75 samples: 34 current, 18 former, 23 never smokers) from the AGED database
- KEGG and Biocarta pathway gene-set definitions (~400 pathways and complexes)

## Outputs

- Expression data matrices with sample group labels (DM2/NGT/IGT or current/former/never) ready for pathway activity analysis

## How to apply

Obtain the type 2 diabetes expression profiles (43 samples: 17 DM2, 17 NGT, 9 IGT) from the deposit at the Whitehead Institute and the airway epithelia profiles (75 samples: 34 current, 18 former, 23 never smokers) from the AGED database. Also assemble the ~400 pathway/complex gene-sets used in the article from the KEGG and Biocarta websites, using the versions that were current at the time (archived at the PLAGE website). Confirm the sample group labels (DM2/NGT/IGT; current/former/never) before computing pathway activity, since the pairwise comparisons — e.g. DM2 vs NGT and former vs never smokers — depend on these groupings.

## Related tools

- **PLAGE** (Web program that consumes the loaded expression datasets and the KEGG/Biocarta gene-sets to compute pathway activity levels via SVD) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Sample counts match the article: 43 samples for the diabetes cohort (17 DM2 + 17 NGT + 9 IGT) and 75 samples for the airway epithelia cohort (34 current + 18 former + 23 never)
- Group labels are correctly assigned so that pairwise comparisons such as DM2 vs NGT and current vs never can be formed as in the article
- The KEGG/Biocarta pathway files correspond to the versions archived at the PLAGE website (dulci.biostat.edu/pathways)

## Limitations

- KEGG and Biocarta pathways evolve; the article notes the versions used are available at the PLAGE website but does not state specific version numbers, so exact reproduction may require obtaining those archived pathway files
- No version numbers or accessions for the datasets are given in the extracted text — provenance beyond the named sources (Whitehead Institute deposit, AGED database) is not documented here

## Evidence

- [abstract] Datasets: "expression data from a study of type 2 diabetes"
- [abstract] Repos: "accessible at http://dulci.biostat.edu/pathways"
- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Findings: "The versions used for this article are available at the PLAGE website [11]."

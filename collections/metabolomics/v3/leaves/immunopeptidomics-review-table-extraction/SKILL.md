---
name: immunopeptidomics-review-table-extraction
description: Use when a question requires a structured side-by-side comparison of
  immunopeptidomics PSM rescoring tools (e.g. which support predicted binding-affinity
  features) from the review's Table 1 rather than prose.
when_to_use_negative:
- The goal is primary experimental validation or benchmarking of rescoring performance
  — the source provides a comparative catalog, and no evaluation results, metrics,
  or validation data are described.
- The input is already a structured feature-support table; re-extraction would duplicate
  the comparison rather than curate it from prose.
- The question concerns tools outside the eight pipelines reviewed in Table 1 (e.g.
  general-purpose rescoring engines like Percolator or mokapot as standalone classifiers).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3473
tools:
- name: MS2Rescore
  role: Reviewed pipeline row in Table 1; uses predicted peptidoform properties (MS2PIP
    fragment intensities, DeepLC retention time) and its open-source Apache-2.0 availability
    anchors the license/availability column
  repo: https://github.com/compomics/ms2rescore
- name: MHCquant
  role: One of the two reviewed pipelines identified as supporting predicted MHC binding
    affinity-based rescoring features
- name: inSPIRE
  role: The other reviewed pipeline identified as supporting predicted MHC binding
    affinity-based rescoring features
- name: Percolator
  role: Semi-supervised rescoring engine referenced as an external dependency underlying
    the reviewed pipelines; cited without a specified version in the source
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:976e4e445cb3f59c8e93a8a057430dbbe0ba2184bf64bd33d99794d3cd4f5a8f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - AlphaPeptDeep
  - DeepRescore
  - INFERYS
  - MHCquant
  - MS2Rescore
  - MSBooster
  - Oktoberfest
  - Percolator
  - inSPIRE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# immunopeptidomics-review-table-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the comparative Table 1 of machine learning-based PSM rescoring pipelines for immunopeptidomics from the review (DOI 10.1002/pmic.202300336) into a machine-readable tab-separated table with feature-support, availability/license, acquisition-mode, and search-engine columns for eight named pipelines.

## When to use

Use when a question requires a structured side-by-side comparison of immunopeptidomics PSM rescoring tools (e.g. which support predicted binding-affinity features) from the review's Table 1 rather than prose.

## When NOT to use

- The goal is primary experimental validation or benchmarking of rescoring performance — the source provides a comparative catalog, and no evaluation results, metrics, or validation data are described.
- The input is already a structured feature-support table; re-extraction would duplicate the comparison rather than curate it from prose.
- The question concerns tools outside the eight pipelines reviewed in Table 1 (e.g. general-purpose rescoring engines like Percolator or mokapot as standalone classifiers).

## Inputs

- Full text of the review article DOI 10.1002/pmic.202300336 (PROTEOMICS, 2023) containing Table 1
- Named pipeline list: AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE, MHCquant, MS2Rescore, MSBooster, Oktoberfest

## Outputs

- table1_rescoring_pipelines.tsv: header row plus exactly 8 data rows (one per pipeline) with columns for predicted peptidoform-property features (fragment ion intensities/MS2, retention time, collisional cross section), MHC binding-affinity features, availability/license/usage restrictions, acquisition modes (DDA, DIA, DDA-PASEF), and supported search engines

## How to apply

Load the full text of the review (DOI 10.1002/pmic.202300336) and locate Table 1, which compares machine learning-based PSM rescoring tools for immunopeptidomics. Create one row per reviewed pipeline — AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE, MHCquant, MS2Rescore, MSBooster, and Oktoberfest — and populate feature-support columns for predicted peptidoform properties (fragment ion intensities/MS2, retention time, collisional cross section) and MHC binding-affinity features. Fill availability, open-source license, and usage-restriction columns from the table's availability entries, then the acquisition-mode columns (DDA, DIA, DDA-PASEF) and supported-search-engine columns. Write the curated table to table1_rescoring_pipelines.tsv with a header row and tab-separated fields; the key review finding is that only MHCquant and inSPIRE support predicted binding affinity-based features.

## Related tools

- **MS2Rescore** (Reviewed pipeline row in Table 1; uses predicted peptidoform properties (MS2PIP fragment intensities, DeepLC retention time) and its open-source Apache-2.0 availability anchors the license/availability column) — https://github.com/compomics/ms2rescore
- **MHCquant** (One of the two reviewed pipelines identified as supporting predicted MHC binding affinity-based rescoring features)
- **inSPIRE** (The other reviewed pipeline identified as supporting predicted MHC binding affinity-based rescoring features)
- **Percolator** (Semi-supervised rescoring engine referenced as an external dependency underlying the reviewed pipelines; cited without a specified version in the source)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame(rows); df.to_csv('table1_rescoring_pipelines.tsv', sep='\t', index=False)"
```

## Evaluation signals

- Output TSV contains exactly 8 data rows, one for each of AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE, MHCquant, MS2Rescore, MSBooster, and Oktoberfest, plus a header row.
- All specified columns (feature support, availability/license, acquisition modes, search engines) are present and populated for every row — no empty required cells.
- The MHC binding-affinity column is affirmative for exactly MHCquant and inSPIRE and negative for the other six pipelines, matching the review's finding.
- Field separator is a tab with no embedded tabs/newlines within fields; round-tripping via a TSV reader (e.g. pandas with sep='\t') yields 8 records with the expected column count.

## Limitations

- Coverage is limited to the eight pipelines reviewed in Table 1 of the 2023 review; newer tools or updated features of these tools are out of scope.
- Availability, license, and usage-restriction entries reflect the review's snapshot and may have changed since publication.
- The review documents no evaluation results or benchmark metrics for the compared pipelines, so the table records capability support only, not measured performance.

## Evidence

- [abstract] PSM rescoring as machine learning-based solution context: [evidence span withheld — anchor status: not_found]
- [abstract] Predicted peptidoform property features covered by the table columns: "features based on predicted peptidoform properties, including fragment ion intensities, retention time, and collisional cross section, have been used to improve the accuracy and sensitivity of"
- [readme] MS2Rescope row scope for immunopeptidomics workflows: "MS²Rescore is **ideal for challenging proteomics identification workflows**, such as
proteogenomics, metaproteomics, or immunopeptidomics."

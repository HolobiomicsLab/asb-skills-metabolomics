---
name: acquisition-mode-search-engine-compatibility-mapping
description: 'Decision support for selecting a PSM rescoring pipeline: when input
  is the review''s Table 1 (pipelines AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE,
  MHCquant, MS2Rescore, MSBooster, Oktoberfest) and the goal is a per-tool comparison
  of binding-affinity features, acquisition modes, and.'
when_to_use_negative:
- Input is raw MS/MS spectra or search-engine output requiring actual PSM rescoring
  — this skill catalogs tool capabilities, it does not rescore PSMs
- 'Only the article abstract is available: the abstract lists tools and feature types
  but not the per-tool Table 1 entries, so the acquisition-mode and search-engine
  columns cannot be populated verifiably'
- The question concerns benchmark performance metrics or identification gains of the
  pipelines rather than their feature, mode, and search-engine compatibility
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MHCquant
  role: One of the two reviewed pipelines that support predicted MHC binding affinity-based
    rescoring features
- name: inSPIRE
  role: One of the two reviewed pipelines that support predicted MHC binding affinity-based
    rescoring features
- name: MS2Rescore
  role: Reviewed pipeline compared in the compatibility table; rescores PSMs with
    LC-MS predictors (MS2PIP, DeepLC) and is a reference for the predicted peptidoform
    property feature column
  repo: https://github.com/compomics/ms2rescore
- name: Percolator
  role: Machine learning-based PSM rescoring solution framing the comparison; rescoring
    engines such as ristretto reimplement its semi-supervised algorithm
- name: Mumble
  role: PSM generator supporting open modification searching, relevant to the PSM-generator/search-engine
    compatibility dimension of the catalog
  repo: https://github.com/compomics/mumble
- name: ristretto
  role: Semi-supervised rescoring engine (svm or lda models) purpose-built for MS²Rescore,
    referenced for the availability/open-source column
  repo: https://github.com/CompOmics/ristretto
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:18f89dbe50eef5dd1fa9369bc2a035a270ffae3686097f1117d856c3ac26039a
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
  - Mumble
  - Oktoberfest
  - Percolator
  - inSPIRE
  - ristretto
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# acquisition-mode-search-engine-compatibility-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a machine-readable compatibility catalog of machine learning-based PSM rescoring pipelines for immunopeptidomics, recording per-tool feature support (predicted peptidoform properties vs MHC binding affinity), availability/license, supported acquisition modes (DDA, DIA, DDA-PASEF), and supported search engines. It turns the narrative comparison in the review (DOI 10.1002/pmic.202300336) into a reproducible TSV for tool selection.

## When to use

Decision support for selecting a PSM rescoring pipeline: when input is the review's Table 1 (pipelines AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE, MHCquant, MS2Rescore, MSBooster, Oktoberfest) and the goal is a per-tool comparison of binding-affinity features, acquisition modes, and search-engine compatibility.

## When NOT to use

- Input is raw MS/MS spectra or search-engine output requiring actual PSM rescoring — this skill catalogs tool capabilities, it does not rescore PSMs
- Only the article abstract is available: the abstract lists tools and feature types but not the per-tool Table 1 entries, so the acquisition-mode and search-engine columns cannot be populated verifiably
- The question concerns benchmark performance metrics or identification gains of the pipelines rather than their feature, mode, and search-engine compatibility

## Inputs

- Full text of the review article (DOI 10.1002/pmic.202300336), specifically Table 1 comparing machine learning-based PSM rescoring tools
- Named pipeline list: AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE, MHCquant, MS2Rescore, MSBooster, Oktoberfest

## Outputs

- table1_rescoring_pipelines.tsv: header row plus exactly 8 tab-separated data rows with feature-support, availability/license, acquisition-mode (DDA, DIA, DDA-PASEF), and supported-search-engine columns

## How to apply

Load the full text of the review (DOI 10.1002/pmic.202300336) and locate Table 1, the comparison of machine learning-based PSM rescoring tools for immunopeptidomics. Create one row per reviewed pipeline — AlphaPeptDeep, DeepRescore, INFERYS, inSPIRE, MHCquant, MS2Rescore, MSBooster, Oktoberfest — in a tab-separated table with columns for predicted peptidoform property features (fragment ion intensities/MS2, retention time, collisional cross section), MHC binding-affinity features, availability/open-source license/usage restrictions, acquisition modes (DDA, DIA, DDA-PASEF), and supported search engines. Populate each cell strictly from the table entries rather than from the abstract prose. The key decision point is the binding-affinity column: only MHCquant and inSPIRE support predicted binding affinity-based features among the reviewed pipelines. Write the curated table to table1_rescoring_pipelines.tsv with a header row.

## Related tools

- **MHCquant** (One of the two reviewed pipelines that support predicted MHC binding affinity-based rescoring features)
- **inSPIRE** (One of the two reviewed pipelines that support predicted MHC binding affinity-based rescoring features)
- **MS2Rescore** (Reviewed pipeline compared in the compatibility table; rescores PSMs with LC-MS predictors (MS2PIP, DeepLC) and is a reference for the predicted peptidoform property feature column) — https://github.com/compomics/ms2rescore
- **Percolator** (Machine learning-based PSM rescoring solution framing the comparison; rescoring engines such as ristretto reimplement its semi-supervised algorithm)
- **Mumble** (PSM generator supporting open modification searching, relevant to the PSM-generator/search-engine compatibility dimension of the catalog) — https://github.com/compomics/mumble
- **ristretto** (Semi-supervised rescoring engine (svm or lda models) purpose-built for MS²Rescore, referenced for the availability/open-source column) — https://github.com/CompOmics/ristretto

## Evaluation signals

- The output TSV contains exactly 8 data rows (one per named pipeline) plus a header row
- All specified columns — predicted peptidoform property features, MHC binding affinity, availability/license, DDA, DIA, DDA-PASEF, supported search engines — are present and populated in every row
- The MHC binding-affinity column flags exactly two tools (MHCquant and inSPIRE) as supporting predicted binding affinity-based features
- Every feature-support, mode, and search-engine cell is traceable to a Table 1 entry of DOI 10.1002/pmic.202300336 rather than inferred from the abstract
- Fields are tab-separated with no stray delimiters, so the file parses as a valid TSV

## Limitations

- Coverage is restricted to the eight pipelines reviewed in the article's Table 1; tools outside that list (e.g. Prosit or NetMHCpan, which appear as feature predictors) are not rows in the catalog
- Cell values are only as reliable as the review's Table 1 entries; acquisition-mode and search-engine support may have changed since publication
- The finding that only MHCquant and inSPIRE support binding-affinity features depends on the full text of Table 1, which is not reproduced in the abstract

## Evidence

- [full_text] Only two PSM rescoring tools, MHCquant and inSPIRE, support predicted binding affinity-based features among the reviewed pipelines.: [evidence span withheld — anchor status: not_found]
- [other] Acquisition-mode and search-engine columns derive from Table 1 entries: [evidence span withheld — anchor status: not_found]
- [other] Validation of the output row count and column completeness: [evidence span withheld — anchor status: not_found]
- [abstract] Predicted peptidoform property feature column definition: "features based on predicted peptidoform properties, including fragment ion intensities, retention time, and collisional cross section"

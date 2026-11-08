---
name: isotope-labeled-compound-tissue-tracking
description: Use when isotope-labeled metabolites (e.g. 13C-labeled amino acids) must
  be tracked across multiple tissues and assigned to distinct tissue-enrichment categories
  such as liver-enriched vs. co-enriched.
when_to_use_negative:
- Data are unlabeled (no 13C enrichment) or tissues are not matched to the same labeled
  diet, since distribution categories depend on isotope-tracing across the five tissues.
- The goal is compound annotation from raw INADEQUATE spectra rather than downstream
  tissue-distribution classification; PyINETA's peak-picking/network-matching skill
  handles that upstream step.
edam_operation: http://edamontology.org/operation_0301
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: Upstream tool that integrates INADEQUATE and 13C-JRES to provide the annotated
    compound peak data used for tissue-distribution classification
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Implementation language for the tissue-distribution classification script
    and tabulation of enrichment assignments
- name: NMRbox
  role: Platform on which PyINETA is installed and publicly available for running
    the pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:d936cd54bb9af04b6d4b29583ec940c41b3a397c17f467b165bb60b0fef53567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_0301
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# isotope-labeled-compound-tissue-tracking

## Summary

Classifies the tissue distribution of metabolites derived from a 13C-labeled diet by quantifying their representative 13C-JRES peaks across tissues (liver, spleen, pancreas, muscle, lung) and assigning each compound to an enrichment category (e.g. liver-enriched, liver-depleted, co-enriched).

## When to use

Use when isotope-labeled metabolites (e.g. 13C-labeled amino acids) must be tracked across multiple tissues and assigned to distinct tissue-enrichment categories such as liver-enriched vs. co-enriched.

## When NOT to use

- Data are unlabeled (no 13C enrichment) or tissues are not matched to the same labeled diet, since distribution categories depend on isotope-tracing across the five tissues.
- The goal is compound annotation from raw INADEQUATE spectra rather than downstream tissue-distribution classification; PyINETA's peak-picking/network-matching skill handles that upstream step.

## Inputs

- Tissue-level 13C-JRES peak intensity or presence data for the labeled compounds across the tissues of interest (e.g. liver, spleen, pancreas, muscle, lung)
- List of compounds derived from the isotope-labeled diet with their candidate tissue distribution annotations

## Outputs

- Structured table (CSV/TSV) of compounds with per-tissue enrichment assignments
- Enrichment category assignment per compound (e.g. Type-A liver-enriched, Type-B liver-depleted, Type-C co-enriched)
- Validation report comparing category counts to the published reference figure

## How to apply

Load tissue-level 13C-JRES peak data for the labeled compounds across the tissues of interest (here: liver, spleen, pancreas, muscle, lung). For each compound, quantify its representative peak intensity or presence in each tissue from the integrated 13C-JRES spectra, then classify its distribution pattern into an enrichment category based on which tissues show highest relative abundance — e.g. Type-A (liver-enriched), Type-B (depleted in liver but enriched in other tissues), Type-C (co-enriched in liver and other tissues). Tabulate the compounds with their per-tissue enrichment assignments in a structured table (CSV/TSV). Validate category counts against the published figure (e.g. Figure 3) to confirm that the expected numbers of compounds per category (13 Type-A, 2 Type-B, 4 Type-C in the PyINETA mouse study) are reproduced. This skill sits downstream of PyINETA's INADEQUATE network annotation, using the 13C-JRES integration step to turn peak pick data into tissue-specific distribution calls.

## Related tools

- **PyINETA** (Upstream tool that integrates INADEQUATE and 13C-JRES to provide the annotated compound peak data used for tissue-distribution classification) — https://github.com/edisonomics/PyINETA
- **Python** (Implementation language for the tissue-distribution classification script and tabulation of enrichment assignments)
- **NMRbox** (Platform on which PyINETA is installed and publicly available for running the pipeline)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Reproduced tissue-by-tissue assignment table matches the published Figure 3 category counts (13 Type-A, 2 Type-B, 4 Type-C).
- Each compound receives exactly one enrichment category and a complete per-tissue vector across all five tissues.
- Assignments are consistent with relative peak-intensity rankings — compounds labeled liver-enriched should show higher 13C-JRES peak intensity in liver than in spleen/pancreas/muscle/lung.
- Output CSV/TSV has one row per compound and columns for each tissue and the assigned category.

## Limitations

- The analysis is limited to tissues and compounds present in the study; labels with low intensity or absent in some tissues may be ambiguous.
- Category boundaries are heuristic (highest relative abundance), not statistically modeled; sensitive to peak-picking noise in the upstream PyINETA step.
- Reference validation relies on the published figure; new datasets without a comparable reference cannot be cross-checked the same way.

## Evidence

- [abstract] PyINETA is an open-source platform that provides structural information of molecules using INADEQUATE: "PyINETA is an open-source platform that provides structural information of molecules using INADEQUATE"
- [abstract] The distribution of those compounds was tissue-specific, showing enrichment of particular metabolites in liver, spleen, pancreas, muscle, or lung.: "The distribution of those compounds was tissue-specific, showing enrichment of particular metabolites in liver, spleen, pancreas, muscle, or lung."
- [other] Reproduce the 19-compound tissue-distribution analysis across 13C-labeled amino acid fed mice: [evidence span withheld — anchor status: not_found]
- [other] 13 compounds enriched in liver (Type-A), 2 metabolites depleted in liver but enriched in pancreas and/or spleen (Type-B, including O-phosphorylethanolamine and hypotaurine), and 4 compounds enriched in both liver and other tissues (Type-C, including valine, threonine, isoleucine, and taurine).: [evidence span withheld — anchor status: not_found]
- [methods] This is the documentation for the PyINETA package version 2.0.0.: "This is the documentation for the PyINETA package version 2.0.0."
- [discussion] PyINETA is installed on NMRbox17 and publicly available.: "PyINETA is installed on NMRbox17 and publicly available."
- [intro] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound"

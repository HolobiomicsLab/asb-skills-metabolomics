---
name: cross-validation-evaluation-data-processing
description: Use when MS/MS spectra plus structure labels (e.g., GNPS cross-validation
  data) must be scored against database structure candidates to produce a top-k correct-identification
  curve for a structure identification tool.
when_to_use_negative:
- The goal is de novo fragmentation tree or molecular formula assignment without evaluating
  structure identification accuracy against a labeled benchmark.
- The input is a novel metabolomics sample batch (e.g., LC-MS/MS of new extracts)
  rather than the labeled GNPS cross-validation evaluation set — SIRIUS identification
  rates >70% there reflect a different, non-cross-validation context.
- The evaluation requires an updated CSI:FingerID version's identification rate rather
  than replicating the original CSI:FingerID 1.1 release setup.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Command-line framework used to load the MS/MS spectra and run PubChem structure
    database searching for each query spectrum (version 6.5.4 in the replication workflow)
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Structure identification service integrated into SIRIUS; its top-ranked PubChem
    candidates are the ranked outputs scored in the top-k identification curve
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS-API SDKs
  role: Optional OpenAPI client (Python/R/Java) for programmatically driving the SIRIUS/CSI:FingerID
    web services and retrieving candidate results
  repo: https://github.com/sirius-ms/sirius-client-openAPI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:b4e55a219a75da3fe99534284a72a8b16eacd7824fc94aa769d62a8993783ee0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  - SIRIUS-API SDKs
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-validation-evaluation-data-processing

## Summary

Reproduces the structure-disjoint cross-validation evaluation of CSI:FingerID structure identification on GNPS MS/MS spectra, computing the top-k identification curve against PubChem search candidates and confirming the reported 40.4% top-1 correct identification rate of CSI:FingerID 1.1.

## When to use

Use when MS/MS spectra plus structure labels (e.g., GNPS cross-validation data) must be scored against database structure candidates to produce a top-k correct-identification curve for a structure identification tool.

## When NOT to use

- The goal is de novo fragmentation tree or molecular formula assignment without evaluating structure identification accuracy against a labeled benchmark.
- The input is a novel metabolomics sample batch (e.g., LC-MS/MS of new extracts) rather than the labeled GNPS cross-validation evaluation set — SIRIUS identification rates >70% there reflect a different, non-cross-validation context.
- The evaluation requires an updated CSI:FingerID version's identification rate rather than replicating the original CSI:FingerID 1.1 release setup.

## Inputs

- CC0 GNPS cross-validation MS/MS spectra and associated metadata (from bio.informatik.uni-jena.de/data/)
- PubChem structure database search results per query spectrum (candidate structures, ranks, scores)
- Ground-truth correct structure labels for each query spectrum

## Outputs

- identification_curve.csv with top-k values and corresponding correct identification fractions
- Top-1 correct identification rate (expected ~40.4% for CSI:FingerID 1.1 on GNPS)

## How to apply

Download the CC0 GNPS cross-validation evaluation data from https://bio.informatik.uni-jena.de/data/ that replicates the original CSI:FingerID release evaluation setup searching in PubChem. Load the MS/MS spectra and metadata with the SIRIUS command-line tool (e.g., version 6.5.4), running structure database searching in PubChem so each query spectrum yields ranked candidates. For each query spectrum, retrieve the top-k structure candidates from the PubChem search results, recording their ranks and scores. For each k, compute the fraction of query spectra whose correct structure falls within the top-k candidates, and verify that at k=1 this fraction matches the reported 40.4% for CSI:FingerID 1.1. Save the (top-k, correct-identification-fraction) pairs to identification_curve.csv.

## Related tools

- **SIRIUS** (Command-line framework used to load the MS/MS spectra and run PubChem structure database searching for each query spectrum (version 6.5.4 in the replication workflow)) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Structure identification service integrated into SIRIUS; its top-ranked PubChem candidates are the ranked outputs scored in the top-k identification curve) — https://github.com/sirius-ms/sirius
- **SIRIUS-API SDKs** (Optional OpenAPI client (Python/R/Java) for programmatically driving the SIRIUS/CSI:FingerID web services and retrieving candidate results) — https://github.com/sirius-ms/sirius-client-openAPI

## Evaluation signals

- At k=1, the fraction of query spectra with the correct structure ranked first matches the reported 40.4% within expected tolerance.
- identification_curve.csv exists and contains one (top-k, correct-identification-fraction) pair per k, with fractions non-decreasing as k increases.
- Every query spectrum in the GNPS cross-validation set contributes a candidate ranking from the PubChem database search (no unprocessed queries).
- Candidate ranks and scores recorded per query spectrum are consistent with the SIRIUS PubChem search output before aggregation into the curve.

## Limitations

- The 40.4% figure applies to CSI:FingerID 1.1 under the original release's evaluation setup; newer SIRIUS versions (e.g., 6.5.4) may yield different rates.
- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are for academic research and education use only; commercial redistribution is prohibited.
- SIRIUS requires high-resolution mass spectra; isotope pattern analysis was rewritten with stability stated only as 'hopefully more stable than before'.

## Evidence

- [abstract] SIRIUS 4 integrates CSI:FingerID for searching in molecular structure databases.: "SIRIUS 4 integrates CSI:FingerID for searching in molecular structure databases."
- [methods] For each query spectrum, retrieve the top-k structure candidates from the PubChem database search results, recording their ranks and scores.: [evidence span withheld — anchor status: not_found]
- [methods] Confirm the reported performance by verifying that at k=1, the fraction of correct identifications matches the reported 40.4%.: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]
- [readme] Results are retrieved from the web service and can be displayed in the SIRIUS graphical user interface. This functionality is also available for the SIRIUS command-line tool.: "Results are retrieved from the web service and can be displayed in the SIRIUS graphical user interface. This functionality is also available for the SIRIUS command-line tool."

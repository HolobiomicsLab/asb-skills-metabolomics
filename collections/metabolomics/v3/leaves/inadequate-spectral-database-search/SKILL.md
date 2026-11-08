---
name: inadequate-spectral-database-search
description: Use when you have filtered networks of picked peaks from an INADEQUATE
  NMR spectrum (e.g. from PyINETA's cluster/find steps) and need to identify which
  metabolites from a simulated INADEQUATE database produced them.
when_to_use_negative:
- Query data are not INADEQUATE peak networks (e.g. plain 1D or JRES peak lists without
  connectivity) — the matching relies on double-quantum relationships between coupled
  carbons
- Peaks have not yet been filtered into networks of peaks from the same compound —
  matching expects network-level input from the find step
- Input files exceed the repository's provided example size (example2 inputs >100
  Mb are not shipped; contact the Edison lab for raw files)
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: 'Runs the matching step: compares filtered peak networks to a simulated INADEQUATE
    database of metabolites and scores hits by coverage'
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for run_pyineta.py, the CLI entry point that exposes the match step
    and its thresholds
- name: NMRbox
  role: Hosted environment where PyINETA (including the matching workflow) is pre-installed
    and publicly available
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:647db7cfd0708a47ef78fb38c51a4204deaa0559907988403ceb5974d4249819
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# inadequate-spectral-database-search

## Summary

Match filtered INADEQUATE peak networks against a simulated INADEQUATE database of metabolites using PyINETA's matching module, scoring each candidate compound with hit and coverage scores to identify metabolites present in the query spectrum.

## When to use

Use when you have filtered networks of picked peaks from an INADEQUATE NMR spectrum (e.g. from PyINETA's cluster/find steps) and need to identify which metabolites from a simulated INADEQUATE database produced them.

## When NOT to use

- Query data are not INADEQUATE peak networks (e.g. plain 1D or JRES peak lists without connectivity) — the matching relies on double-quantum relationships between coupled carbons
- Peaks have not yet been filtered into networks of peaks from the same compound — matching expects network-level input from the find step
- Input files exceed the repository's provided example size (example2 inputs >100 Mb are not shipped; contact the Edison lab for raw files)

## Inputs

- Filtered networks of picked peaks from INADEQUATE spectra (PyINETA cluster/find output)
- Simulated INADEQUATE database of metabolite networks (database entry name used via --dbname)
- config.ini file with matching parameters (CSMT, NCMT, DQMT thresholds)

## Outputs

- Matching module output table listing each sample network's matched database metabolite with hit score and coverage score
- Optionally network-vs-database plots (singleplot step with -n and -d options)

## How to apply

Load the filtered peak networks produced by the PyINETA clustering/finding module together with a simulated INADEQUATE database of metabolite networks. For each sample network, compare every peak against each database entry applying three user-tuned thresholds: CSMT (carbon chemical shift matching tolerance, ppm), NCMT (network connectivity matching tolerance, Hz), and DQMT (double-quantum shift matching tolerance, ppm). A network is declared a match when chemical shift matches exceed NCMT and the double-quantum differences fall below DQMT. Each matched network is then scored with a hit score (how well the network matches the database compound) and a coverage score (the fraction of the database network represented in the sample), and results are written to the matching module's output table. The match step can be run standalone or chained from earlier steps using the run_pyineta.py -s option.

## Related tools

- **PyINETA** (Runs the matching step: compares filtered peak networks to a simulated INADEQUATE database of metabolites and scores hits by coverage) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for run_pyineta.py, the CLI entry point that exposes the match step and its thresholds)
- **NMRbox** (Hosted environment where PyINETA (including the matching workflow) is pre-installed and publicly available)

## Examples

```
python run_pyineta.py -c config.ini -s match -o example1_output
```

## Evaluation signals

- Every matched network in the output table carries both a hit score and a coverage score — absence of either score indicates the match step did not run
- Matched database compounds should be confirmed against known compound annotations; PyINETA's automatic parameter tracking (config.ini echo) lets you reproduce the exact CSMT/NCMT/DQMT settings used
- Cross-check against the example1_default_output reference results shipped in the repo after running example1 with a fresh config.ini
- Networks failing all three thresholds (CSMT, NCMT, DQMT) should appear unmatched, not silently dropped — verify unmatched networks remain in the output table
- Total number of matched networks should be ≤ total number of filtered networks fed into the matching step

## Limitations

- Thresholds CSMT, NCMT, and DQMT are user-configurable in config.ini — poor choices produce false matches or missed identifications; the paper does not prescribe universal defaults
- Matching is limited to metabolites present in the simulated INADEQUATE database supplied to the pipeline
- Example2 in the repository requires large (>100 Mb) raw input files that must be requested separately from the Edison lab, so reproducibility of large-scale runs depends on external access
- The README says networks are 'ideally' from the same compound — ambiguous clustering propagates into ambiguous matching

## Evidence

- [intro] Network matching rationale (intro): "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [methods] Match thresholds and scoring (source card): [evidence span withheld — anchor status: not_found]
- [readme] Pipeline steps and options (readme): [evidence span withheld — anchor status: not_found]
- [abstract] Transparent parameter tracking (abstract): "PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [readme] Example outputs for verification (readme): [evidence span withheld — anchor status: external_doc]

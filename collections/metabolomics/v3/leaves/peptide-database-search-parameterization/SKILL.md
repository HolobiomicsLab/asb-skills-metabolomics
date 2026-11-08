---
name: peptide-database-search-parameterization
description: 'Use when a simulated proteomics run must be seeded with real identifications:
  you have a public MS/MS deposit (PRIDE/ProteomeXchange) and need a 1% FDR peptide
  list with retention times as simulation input.'
when_to_use_negative:
- When the goal is novel peptide discovery rather than seeding a simulated run — this
  workflow's output is deliberately restricted to a 1% FDR curated list, not a maximal
  identification set.
- When the input already contains a peptide/retention-time csv compatible with the
  SMITER example_data layout — skip the search and go straight to smiter.lib.csv_to_peak_properties.
- When simulating non-peptide biomolecules (nucleosides, lipids) — no proteomics database
  search is needed; SMITER builds those simulations directly from chemical formulas.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: X!Tandem
  role: Database search engine that identifies peptides from the Bruderer et al. reference
    run spectra, executed via the ursgal framework with the specified mass tolerances
    and modifications
- name: ursgal
  role: Unified framework through which the X!Tandem search of the reference run is
    parameterized and executed
- name: Percolator
  role: PSM validation tool (v3.4.0) used to re-score identifications and enforce
    the 1% FDR cutoff
- name: SMITER
  role: Consumes the resulting peptide list (as csv) to build the peak-properties
    dictionary for the simulated mzML run
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Glue language for writing the csv peptide list and invoking smiter.lib.csv_to_peak_properties
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:0c13f39c113023455fcd6d0c01e2b5fc26ea12b3c6bef43219b0faaf9c464ad0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Percolator
  - Python
  - SMITER
  - X!Tandem
  - ursgal
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# peptide-database-search-parameterization

## Summary

Parameterize a peptide database search (X!Tandem via ursgal, Percolator validation at 1% FDR) to convert a real LC-MS/MS run — here the Bruderer et al. reference dataset — into a curated peptide list with retention times that feeds SMITER's simulation pipeline as a peak-properties csv.

## When to use

Use when a simulated proteomics run must be seeded with real identifications: you have a public MS/MS deposit (PRIDE/ProteomeXchange) and need a 1% FDR peptide list with retention times as simulation input.

## When NOT to use

- When the goal is novel peptide discovery rather than seeding a simulated run — this workflow's output is deliberately restricted to a 1% FDR curated list, not a maximal identification set.
- When the input already contains a peptide/retention-time csv compatible with the SMITER example_data layout — skip the search and go straight to smiter.lib.csv_to_peak_properties.
- When simulating non-peptide biomolecules (nucleosides, lipids) — no proteomics database search is needed; SMITER builds those simulations directly from chemical formulas.

## Inputs

- Raw MS/MS spectra from the Bruderer et al. reference run (public PRIDE/ProteomeXchange deposit)
- Search parameter set: 5 ppm precursor tolerance, 0.02 Da fragment tolerance, fixed carbamidomethylation, optional methionine oxidation, optional N-terminal acetylation
- SMITER example_data csv layout (template for the output peptide list)

## Outputs

- Percolator-validated PSM set filtered to 1% FDR (peptide list with retention times)
- peak_properties.csv in the SMITER example_data layout
- SMITER peak-properties dictionary produced by smiter.lib.csv_to_peak_properties

## How to apply

Retrieve the reference LC-MS/MS run (Bruderer et al.) from its public PRIDE/ProteomeXchange deposit, then search the spectra with X!Tandem through the ursgal framework using 5 ppm precursor tolerance and 0.02 Da fragment tolerance, with fixed carbamidomethylation plus optional methionine oxidation and N-terminal acetylation. Validate the resulting PSMs with Percolator v3.4.0 and filter to 1% FDR to obtain the peptide list. Write the peptide list as a csv in the same layout as the SMITER example_data files, and convert it into a SMITER peak-properties dictionary with smiter.lib.csv_to_peak_properties. The 1% FDR threshold is the key quality gate — it ensures the simulated run contains only identifications whose ground truth is known, which is exactly the gold-standard property SMITER simulations are designed for.

## Related tools

- **X!Tandem** (Database search engine that identifies peptides from the Bruderer et al. reference run spectra, executed via the ursgal framework with the specified mass tolerances and modifications)
- **ursgal** (Unified framework through which the X!Tandem search of the reference run is parameterized and executed)
- **Percolator** (PSM validation tool (v3.4.0) used to re-score identifications and enforce the 1% FDR cutoff)
- **SMITER** (Consumes the resulting peptide list (as csv) to build the peak-properties dictionary for the simulated mzML run) — https://github.com/LeidelLab/SMITER
- **Python** (Glue language for writing the csv peptide list and invoking smiter.lib.csv_to_peak_properties)

## Examples

```
peak_properties = smiter.lib.csv_to_peak_properties('peak_properties.csv')
```

## Evaluation signals

- peak_properties.csv exists and parses as a valid csv
- Peptide entries in peak_properties.csv correspond exactly to the 1% FDR PSM set from the Percolator-validated X!Tandem search
- csv layout matches the SMITER example_data files so smiter.lib.csv_to_peak_properties can parse it
- Search parameters recorded (5 ppm precursor, 0.02 Da fragment, fixed carbamidomethylation, optional Met oxidation and N-terminal acetylation) match the intended ursgal/X!Tandem configuration

## Limitations

- The workflow depends on availability and integrity of the public Bruderer et al. PRIDE/ProteomeXchange deposit; it is specific to that reference dataset, not a general search recipe.
- SMITER's simulation quality depends on retention times in the csv; the article's README/discussion notes no changelog and missing discussion prose, so edge-case guidance on RT prediction is limited.
- The article extracted only the csv_to_peak_properties conversion step in detail; Percolator v3.4.0 re-scoring details beyond the 1% FDR filter are documented only at the level of the workflow summary.

## Evidence

- [other] Peptide identifications for the SMITER proteomics simulation were obtained by searching the Bruderer et al. reference run with X!Tandem via the ursgal framework, using 5 ppm precursor and 0.02 Da fragment tolerance.: [evidence span withheld — anchor status: not_found]
- [methods] create the peak properties dict from a csv file using smiter.lib.csv_to_peak_properties: "create the peak properties dict (You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`)"
- [readme] Such gold standards, where the ground truth is known, are required in computational mass spectrometry to test new algorithms and to improve parameters for existing ones.: [evidence span withheld — anchor status: external_doc]

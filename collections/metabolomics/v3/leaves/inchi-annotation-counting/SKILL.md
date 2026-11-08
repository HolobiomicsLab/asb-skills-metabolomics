---
name: inchi-annotation-counting
description: Use when you have an mzmine-processed FBMN feature table with MS/MS spectra
  and need to benchmark how many features get an InChI from GNPS strict vs iterative
  weighted analog search.
when_to_use_negative:
- Your goal is de novo structure elucidation or molecular formula prediction rather
  than benchmarking library-search InChI coverage.
- Your input is raw LC-MS/MS data not yet processed into an mzmine FBMN feature table
  and MGF export — run MZmine 4 feature-based molecular networking first.
- You need matched MS/MS spectra with exact library hits only (strict precision matters
  more than return rate); the iterative analog search deliberately trades precision
  for coverage.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Executes both the strict single-query library search and the iterative weighted
    analog search (GNPS I) as FBMN jobs; annotations are parsed for returned InChIs
  repo: https://github.com/MejriY/MS2DECIDE
- name: MS2DECIDE
  role: Python package that launches the GNPS strict or iterative library search workflows
    via its K_estimation function and can report empty annotations to complete missing
    InChIs or SMILES
  repo: https://github.com/MejriY/MS2DECIDE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:0c04a2e3efdfaa0da2f327547ff0b9787489c555f935ddb559bc476ca768d121
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - MS2DECIDE
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# InChI annotation counting

## Summary

Quantifies how many features in an FBMN feature table receive an InChI identifier from a GNPS spectral library search, comparing the strict single-query baseline against the GNPS iterative weighted analog search (GNPS I). Applied to the P. mutica LC-MS/MS case (MassIVE MSV95596), it verifies the reported return-rate improvement from 187/754 (25%) to 650/754 (89%).

## When to use

Use when you have an mzmine-processed FBMN feature table with MS/MS spectra and need to benchmark how many features get an InChI from GNPS strict vs iterative weighted analog search.

## When NOT to use

- Your goal is de novo structure elucidation or molecular formula prediction rather than benchmarking library-search InChI coverage.
- Your input is raw LC-MS/MS data not yet processed into an mzmine FBMN feature table and MGF export — run MZmine 4 feature-based molecular networking first.
- You need matched MS/MS spectra with exact library hits only (strict precision matters more than return rate); the iterative analog search deliberately trades precision for coverage.

## Inputs

- mzmine 4-processed quantitative feature table (.csv, FBMN export format) for the P. mutica dataset (MassIVE MSV95596)
- MS/MS spectra in MGF format (.mgf) for the 754 features
- GNPS account credentials (username, password, email) for launching FBMN jobs

## Outputs

- gnps1_inchi_counts.csv with strict vs GNPS I InChI counts and rates (187/754, 25% vs 650/754, 89%)
- Per-mode GNPS FBMN annotation tables with InChI identifiers per feature
- Optional empty.tsv report of features with empty annotations under the iterative search

## How to apply

Download the deposited mzmine 4-processed feature table and MS/MS spectra for the P. mutica dataset from MassIVE accession MSV95596. Run the GNPS spectral library search twice on the 754 features: once in `strict` mode (mass difference tolerance 0.02 Da, threshold 0.001, a single FBMN job) and once in `iterative` mode (the iterative weighted analog search, which launches 27 FBMN jobs and can take up to three hours). Parse both result annotation tables and count, per job, the number of features for which an InChI identifier is returned. Compute the return rate over the 754 features and record the strict baseline (187, 25%) alongside the GNPS I result (650, 89%) in gnps1_inchi_counts.csv. Rationale: the iterative analog search relaxes matching so that features whose exact spectra are absent from the library still receive an InChI via analogs, which is why MS2DECIDE integrates 'GNPS (enhanced with a iterative weighted analog search)'.

## Related tools

- **GNPS** (Executes both the strict single-query library search and the iterative weighted analog search (GNPS I) as FBMN jobs; annotations are parsed for returned InChIs) — https://github.com/MejriY/MS2DECIDE
- **MS2DECIDE** (Python package that launches the GNPS strict or iterative library search workflows via its K_estimation function and can report empty annotations to complete missing InChIs or SMILES) — https://github.com/MejriY/MS2DECIDE

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Strict single-query baseline returns an InChI for 187 of 754 features (25%); a large deviation from this indicates a misconfigured strict search (check the 0.02 Da tolerance and 0.001 threshold).
- GNPS I iterative search raises the InChI return count to 650 of 754 (89%); a substantially lower rate suggests the iterative workflow did not complete all jobs.
- The iterative workflow launches 27 FBMN jobs from the GNPS account, while strict launches exactly one — verify the job count to confirm the correct mode ran.
- Output file gnps1_inchi_counts.csv contains counts and rates consistent with the reported 650/754 (89%) vs 187 (25%) comparison.
- The strict-mode FBMN job completes without the workflow failures the README warns about at the 0.001 library score threshold.

## Limitations

- Both workflows use a library score threshold of 0.001, which can cause FBMN workflow failures; the README recommends increasing it (e.g., to 0.1) at the cosine-score prompt.
- The iterative weighted analog search can take up to three hours and launches 27 FBMN jobs, requiring GNPS authentication and internet connectivity.
- Features with empty annotations under the iterative search may still lack InChIs; the optional empty.tsv report helps identify spectra missing InChIs or SMILES on GNPS.
- Counts are dataset-specific (754 P. mutica features, MSV95596); rates will differ on other datasets and library versions.

## Evidence

- [readme] strict: Uses a typical mass difference tolerance of 0.02 Da and threshold value of 0.001: [evidence span withheld — anchor status: not_found]
- [readme] iterative workflow launches 27 FBMN jobs: [evidence span withheld — anchor status: external_doc]
- [readme] MS2DECIDE integrates GNPS with iterative weighted analog search: [evidence span withheld — anchor status: external_doc]
- [readme] empty.tsv helps complete missing InChIs or SMILES: [evidence span withheld — anchor status: external_doc]
- [abstract] aggregate outputs of three annotation tools: "aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"

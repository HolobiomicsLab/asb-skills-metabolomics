---
name: gnps-library-search-execution
description: Use when an mzmine (>2.53) FBMN feature table (CSV) and MGF MS/MS spectra
  need GNPS library annotations, and the goal is to compare strict single-query vs
  iterative weighted analog search InChI return rates.
when_to_use_negative:
- The goal is aggregating GNPS annotations with ISDB-LOTUS and Sirius into a K-ranked
  prioritization (that is the K_estimation step, not the raw library search).
- Input data is not FBMN-formatted (e.g. not an mzmine export feature table/MGF pair),
  since the strict/iterative GNPS workflows are launched as FBMN jobs.
- 'Only exact library matches are needed and runtime matters: the iterative search
  takes up to three hours and 27 FBMN jobs versus one strict job.'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the FBMN spectral library search workflows; strict mode uses 0.02 Da
    tolerance / 0.001 threshold, iterative mode runs the weighted analog search as
    27 FBMN jobs
  repo: https://ccms-ucsd.github.io/GNPSDocumentation/librarysearch/
- name: MS2DECIDE
  role: Python library whose K_estimation function launches the GNPS strict or iterative
    FBMN library search jobs from the user's GNPS account
  repo: https://github.com/MejriY/MS2DECIDE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:c7553ea6be5f252901ce2fde918cc321424d5572c181bc028ff6bb61aec52523
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MS2DECIDE
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# GNPS library search execution

## Summary

Executes GNPS FBMN spectral library searches against a GNPS spectral library, either as a single strict query or as the iterative weighted analog search (GNPS I), to assign InChIs/SMILES to MS/MS features such as those from the P. mutica extract (754 mzmine 4 features).

## When to use

Use when an mzmine (>2.53) FBMN feature table (CSV) and MGF MS/MS spectra need GNPS library annotations, and the goal is to compare strict single-query vs iterative weighted analog search InChI return rates.

## When NOT to use

- The goal is aggregating GNPS annotations with ISDB-LOTUS and Sirius into a K-ranked prioritization (that is the K_estimation step, not the raw library search).
- Input data is not FBMN-formatted (e.g. not an mzmine export feature table/MGF pair), since the strict/iterative GNPS workflows are launched as FBMN jobs.
- Only exact library matches are needed and runtime matters: the iterative search takes up to three hours and 27 FBMN jobs versus one strict job.

## Inputs

- mzmine (>2.53) FBMN quantitative feature table (.csv)
- MGF file of tandem mass spectrometry data (.mgf)
- GNPS account credentials (username, password, email) for launching FBMN jobs
- Deposited dataset files (e.g. MassIVE accession MSV95596)

## Outputs

- GNPS FBMN library search annotations per feature (one job for strict, 27 jobs for iterative)
- InChI/SMILES identifiers per matched MS/MS spectrum
- gnps1_inchi_counts.csv recording InChI counts and return rates for strict vs GNPS I
- empty.tsv report of empty annotations from the iterative search (missing InChIs/SMILES for existing spectra)

## How to apply

Download the mzmine 4-processed feature table and MS/MS spectra (e.g. P. mutica, MassIVE MSV95596, 754 features) and run the GNPS spectral library search workflow. Choose `strict` (mass difference tolerance 0.02 Da, threshold 0.001, one FBMN job) for precise matches, or `iterative` for the weighted analog search, which launches 27 FBMN jobs and can take up to three hours. Because the workflows use a library score threshold of 0.001, FBMN failures can occur, so a higher minimum cosine score (e.g. 0.1) is recommended when prompted. Parse the result annotations, count features with a returned InChI over the 754 features, and record the strict baseline (187, 25%) against GNPS I (650, 89%) for comparison.

## Related tools

- **GNPS** (Hosts the FBMN spectral library search workflows; strict mode uses 0.02 Da tolerance / 0.001 threshold, iterative mode runs the weighted analog search as 27 FBMN jobs) — https://ccms-ucsd.github.io/GNPSDocumentation/librarysearch/
- **MS2DECIDE** (Python library whose K_estimation function launches the GNPS strict or iterative FBMN library search jobs from the user's GNPS account) — https://github.com/MejriY/MS2DECIDE

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()  # then choose 'iterative' GNPS library search (27 FBMN jobs) or 'strict' (1 job)
```

## Evaluation signals

- On the real P. mutica case the strict query returns InChIs for 187/754 features (25%) while GNPS I returns 650/754 (89%).
- Job counts: strict mode launches exactly 1 FBMN job; iterative mode launches 27 FBMN jobs on the user's GNPS account.
- GNPS job parameters match the chosen mode: strict = 0.02 Da mass difference tolerance and 0.001 threshold; recommended minimum cosine score raised to ~0.1 to avoid FBMN failures.
- Output counts/rates are written to gnps1_inchi_counts.csv with counts computed over the 754 features.
- The iterative search optionally produces empty.tsv listing features with empty annotations (spectra lacking InChIs/SMILES on GNPS).

## Limitations

- The default library score threshold of 0.001 can cause FBMN GNPS workflow failures; increasing to e.g. 0.1 is recommended.
- The iterative weighted analog search is slow: it can take up to three hours and launches 27 FBMN jobs.
- Iterative search may return empty annotations for some existing MS/MS spectra, requiring a report of missing InChIs/SMILES.

## Evidence

- [readme] strict mode parameters and recommendation to raise threshold: [evidence span withheld — anchor status: not_found]
- [readme] strict launches one job, iterative launches 27 FBMN jobs: [evidence span withheld — anchor status: external_doc]
- [readme] threshold 0.001 can cause FBMN failures; 0.1 recommended: [evidence span withheld — anchor status: external_doc]
- [readme] GNPS integrated with iterative weighted analog search within MS2DECIDE: [evidence span withheld — anchor status: external_doc]
- [other] P. mutica case: InChI return rate improvement from 187 (25%) to 650 (89%) over 754 features: [evidence span withheld — anchor status: not_found]

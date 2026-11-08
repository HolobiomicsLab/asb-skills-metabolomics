---
name: public-dataset-overdetection-screening
description: Use when curating propagated drug-analog MS/MS matches and needing to
  remove signals whose detection is too widespread across public metabolomics data
  to plausibly reflect real drug exposure.
when_to_use_negative:
- The analog spectrum already receives a full library match in the GNPS library —
  these are direct spectral hits, not propagated analogs, and are removed by the library-match
  filter before dataset testing.
- The parent drug carries an endogenous or dietary-source exposure annotation — such
  analogs are excluded by the source filter regardless of detection prevalence.
- The analog's mass offset is not among the 156 curated, chemically explained mass
  offsets — dataset testing cannot rescue chemically unexplained offsets.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3823
- http://edamontology.org/topic_0091
tools:
- name: GNPS MASST (Fast Search API)
  role: Searches each analog MS/MS spectrum against currently indexed public metabolomics
    datasets to quantify detection prevalence and flag over-detected analogs
  repo: https://github.com/mwang87/GNPS_MASST
- name: robinschmid/microbe_masst (jobs.py batch search)
  role: Executes batch MASST searches of multiple analog spectra against all indexed
    public datasets and generates the _datasets.tsv output used to judge over-detection
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:427d67c39cc53d1d0a43421f4e997a040892a0b854de58359d687dc76a2a43b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/GNPS_MASST
  tools:
  - GNPS MASST (Fast Search API)
  - robinschmid/microbe_masst (jobs.py batch search)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3823
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# public-dataset-overdetection-screening

## Summary

The fourth curation filter of the GNPS Drug Library propagated-analog workflow: analog spectra surviving the endogenous/food-source, mass-offset, and library-match filters are searched against 20 public test metabolomics datasets, and analogs detected too ubiquitously to indicate true drug exposure are excluded before propagating the final library.

## When to use

Use when curating propagated drug-analog MS/MS matches and needing to remove signals whose detection is too widespread across public metabolomics data to plausibly reflect real drug exposure.

## When NOT to use

- The analog spectrum already receives a full library match in the GNPS library — these are direct spectral hits, not propagated analogs, and are removed by the library-match filter before dataset testing.
- The parent drug carries an endogenous or dietary-source exposure annotation — such analogs are excluded by the source filter regardless of detection prevalence.
- The analog's mass offset is not among the 156 curated, chemically explained mass offsets — dataset testing cannot rescue chemically unexplained offsets.

## Inputs

- Raw drug-analog match table from the GNPS Drug Library spectra (analogs surviving prior filters)
- Curated drug metadata (exposure sources, pharmacologic classes) used to flag endogenous or dietary-source drugs
- Analog MS/MS spectra as .mgf files (e.g., from MZmine/GNPS molecular networking) or as USI lists (.csv/.tsv)
- 20 public test metabolomics datasets indexed in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN

## Outputs

- Filtered propagated analog library table (analog_library_filtered.csv) with 12,455 clustered non-duplicated MS/MS spectra covering 1277 drugs
- Per-spectrum _datasets.tsv files reporting the number of unique matching samples per indexed dataset
- _matches.tsv, _library.tsv, and _count_domain.tsv companion outputs from the batch MASST search

## How to apply

Take the analog spectra that already passed the first three filters (parent drug not endogenous/dietary; mass offset among the 156 curated, chemically explained offsets; no full GNPS library match). Batch-search each analog spectrum against currently indexed public metabolomics data (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN) using the GNPS Fast Search API via jobs.py from the microbe_masst repository, and read per-dataset sample-match counts from the generated _datasets.tsv files. Flag any analog that is over-detected — appearing in too many distinct samples/datasets across the 20 public test metabolomics datasets to plausibly indicate a specific drug exposure — and exclude it. Propagate the surviving spectra into the final clustered analog library (analog_library_filtered.csv). Validate the output against the reported result: 12,455 clustered non-duplicated MS/MS spectra representing analogs of 1277 drugs.

## Related tools

- **GNPS MASST (Fast Search API)** (Searches each analog MS/MS spectrum against currently indexed public metabolomics datasets to quantify detection prevalence and flag over-detected analogs) — https://github.com/mwang87/GNPS_MASST
- **robinschmid/microbe_masst (jobs.py batch search)** (Executes batch MASST searches of multiple analog spectra against all indexed public datasets and generates the _datasets.tsv output used to judge over-detection) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # after adding entries as ("input_directory/input_file", "output_directory/output_prefix"); run repeatedly with skip_existing=True until no new output
```

## Evaluation signals

- Final library contains exactly 12,455 clustered non-duplicated MS/MS spectra representing analogs of 1277 drugs.
- No surviving analog spectrum in analog_library_filtered.csv receives a full GNPS library match (filter 4 invariant).
- Every surviving analog's mass offset is among the 156 curated, chemically explained mass offsets.
- Excluded analogs show ubiquitous detection across the 20 public test datasets, visible as large per-dataset unique-sample counts in the _datasets.tsv outputs.
- Re-running jobs.py with skip_existing=True produces no new matches, confirming the batch search completed without silent Fast Search API failures.

## Limitations

- The Fast Search API can silently fail individual entries; jobs.py must be run a couple of times with skip_existing=True until no new output is generated (per the README).
- Over-detection judgments depend on the current public dataset index (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN), which grows over time and may shift prevalence estimates.
- Search outputs include matches from samples not part of curated domain-specific MASSTs, so raw match counts must be interpreted carefully before declaring over-detection.
- The resulting GNPS Drug Library analog readouts are not intended for immediate clinical decision-making.

## Evidence

- [methods] Exclude analogs that are over-detected when searched against 20 public test metabolomics datasets (analogs detected too ubiquitously to indicate true exposure).: [evidence span withheld — anchor status: not_found]
- [results] After applying four filters (endogenous/food-source removal, mass-offset filtering, library-match filtering, and dataset testing), the propagated analog library: [evidence span withheld — anchor status: not_found]
- [readme] A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum in each currently indexed dataset.: [evidence span withheld — anchor status: external_doc]
- [readme] execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN: [evidence span withheld — anchor status: external_doc]

---
name: mgf-file-parsing
description: Use when you need per-dataset tandem mass spectrum counts from MGF files,
  e.g. to reproduce or verify a compareMS2 quality-control figure or sanity-check
  that MGF conversion preserved all spectra.
when_to_use_negative:
- Input files are raw instrument formats (e.g. mzML, RAW) rather than MGF — spectrum
  counts must be taken after MGF conversion or with a different parser.
- The goal is pairwise dataset similarity/distance computation between spectra sets
  — that is compareMS2's distance metric, not spectrum counting.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Source of the default QC metric and of the PXD034932 serum MGF test data whose
    spectrum counts are being reproduced
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:033690364938e6e168d818a5f96ed9d62a0adf32ada7c123ffe10c3649f079ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mgf-file-parsing

## Summary

Count tandem mass spectra in MGF files (BEGIN IONS records) from ProteomeXchange dataset PXD034932 serum datasets to reproduce the default compareMS2 quality-control metric of 42,629–47,626 MS2 spectra per dataset.

## When to use

Use when you need per-dataset tandem mass spectrum counts from MGF files, e.g. to reproduce or verify a compareMS2 quality-control figure or sanity-check that MGF conversion preserved all spectra.

## When NOT to use

- Input files are raw instrument formats (e.g. mzML, RAW) rather than MGF — spectrum counts must be taken after MGF conversion or with a different parser.
- The goal is pairwise dataset similarity/distance computation between spectra sets — that is compareMS2's distance metric, not spectrum counting.

## Inputs

- MGF files (five serum datasets from ProteomeXchange PXD034932)
- Dataset identifiers mapping MGF filenames to PXD accession IDs

## Outputs

- ms2_spectrum_counts.csv: table of MS2 spectrum count per dataset identifier
- Validation flag per dataset: pass if count is in [42,629, 47,626]

## How to apply

Retrieve the five serum MGF files belonging to ProteomeXchange dataset PXD034932, which compareMS2 provides as test data. Parse each MGF file by counting spectrum records, i.e. occurrences of the 'BEGIN IONS' delimiter, one per tandem mass spectrum. Tabulate the MS2 spectrum count alongside each dataset identifier and write the table to ms2_spectrum_counts.csv. Validate that each of the five counts falls within the reported interval of 42,629 to 47,626 tandem mass spectra; a count outside this range indicates an incomplete download or a misparsed MGF file (e.g. dropped or truncated spectrum blocks). These counts correspond to the default compareMS2 quality-control metric reported for the five new serum datasets.

## Related tools

- **compareMS2** (Source of the default QC metric and of the PXD034932 serum MGF test data whose spectrum counts are being reproduced) — https://github.com/524D/compareMS2.git

## Examples

```
grep -c 'BEGIN IONS' PXD034932_serum_*.mgf  # per-file MS2 spectrum counts; expect 42,629–47,626 each
```

## Evaluation signals

- Each of the five per-dataset counts lies within the reported interval 42,629–47,626 tandem mass spectra.
- All five datasets yield five distinct counts — fewer than five rows indicates an incomplete download or misparsed file.
- Total BEGIN IONS matches per file equal the number of END IONS blocks (balanced spectrum records).
- Written CSV contains one count row per dataset identifier, matching values reproduced from PXD034932 files.

## Limitations

- Counts are specific to the five PXD034932 serum datasets; other datasets will have different spectrum totals, so the 42,629–47,626 range is only a validation window for this data.
- The metric depends on the MGF conversion/peak-picking upstream; misparsed or truncated MGF blocks silently undercount spectra.
- compareMS2 documentation notes no explicit software version pinning in the text — record the compareMS2/PXD034932 retrieval date used.

## Evidence

- [results] five serum datasets contain between 42,629 and 47,626 tandem mass spectra each: "each containing between 42,629 and 47,626 tandem mass spectra"
- [methods] Retrieve the five serum MGF files belonging to the ProteomeXchange dataset PXD034932: [evidence span withheld — anchor status: not_found]
- [readme] README PXD034932 pointer: [evidence span withheld — anchor status: not_found]

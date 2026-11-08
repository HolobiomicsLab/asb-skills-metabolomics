---
name: spectral-file-parsing-mgf
description: Use when input data are MS/MS spectra in .mgf format (e.g., from MZmine
  feature-based molecular networking) and you need to read scan records, attach or
  filter metadata, subset spectra by condition, and write a new .mgf for a batch search.
when_to_use_negative:
- Input is already a processed feature table or a list of USIs in a .csv/.tsv file
  — the microbeMASST batch script accepts USIs directly and does not need .mgf re-parsing.
- The spectra are in a vendor raw format or mzML; convert through MZmine first rather
  than attempting .mgf-style parsing.
- You only need to view or search a single spectrum; the standalone web apps (e.g.,
  microbeMASST) accept one spectrum at a time without file parsing.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Source of the .mgf files generated during feature-based molecular networking;
    its exported files are the ones parsed and re-filtered by this skill.
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Implementation language used by the microbe_masst batch scripts to parse and
    filter .mgf spectra before submitting them to the Fast Search API.
  repo: https://github.com/robinschmid/microbe_masst
- name: microbeMASST
  role: Downstream consumer of the filtered .mgf file; the parsed spectra are submitted
    as a batch input against the microbial metabolomics database.
  repo: https://masst.gnps2.org/microbemasst/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:3821b535d139475206aeb3b801731d0fca06cf9125cce858bf1443b0e7ebb9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spectral-file-parsing-mgf

## Summary

Parse and manipulate MS/MS spectra stored in .mgf ( Mascot Generic Format) files—such as those exported from MZmine or the GNPS molecular networking workflow—so they can be filtered, subset, and re-serialized for downstream batch tools like the microbeMASST Fast Search API.

## When to use

Use when input data are MS/MS spectra in .mgf format (e.g., from MZmine feature-based molecular networking) and you need to read scan records, attach or filter metadata, subset spectra by condition, and write a new .mgf for a batch search.

## When NOT to use

- Input is already a processed feature table or a list of USIs in a .csv/.tsv file — the microbeMASST batch script accepts USIs directly and does not need .mgf re-parsing.
- The spectra are in a vendor raw format or mzML; convert through MZmine first rather than attempting .mgf-style parsing.
- You only need to view or search a single spectrum; the standalone web apps (e.g., microbeMASST) accept one spectrum at a time without file parsing.

## Inputs

- .mgf file of MS/MS spectra exported from MZmine (containing PEPMASS, RTINSECONDS, SCANS metadata)
- molecular networking edges file (e.g., from a GNPS job) linking scan IDs by cosine similarity and mass difference
- sample metadata identifying which scans occur in which biological condition (e.g., GF vs SPF)

## Outputs

- filtered .mgf file containing only the retained spectra (e.g., exactly 10,047 SPF-unique consensus spectra in the GF/SPF workflow)
- intermediate in-memory list of parsed spectrum records for downstream subsetting

## How to apply

Parse the .mgf file record-by-record, keyed on the 'BEGIN IONS'/'END IONS' block boundaries, carrying the PEPMASS, RTINSECONDS, and SCANS fields that MZmine writes. Use the SCANS identifiers to join each spectrum to its molecular-networking metadata and edge table, which is how GF-vs-SPF presence is determined. Apply filtering logic (e.g., retain spectra present in SPF animals and drop any spectrum whose edges connect to GF-only spectra with cosine > 0.7 and delta parent mass within ±0.02 Da) on the parsed records, then write the surviving spectra to a new .mgf preserving the original peak list format. The README notes that the microbeMASST batch script expects either such a single .mgf file generated via MZmine from the GNPS molecular networking workflow or a list of USIs in a .csv/.tsv file, and requires Python 3.10.

## Related tools

- **MZmine** (Source of the .mgf files generated during feature-based molecular networking; its exported files are the ones parsed and re-filtered by this skill.) — https://github.com/mzmine/mzmine
- **Python 3.10** (Implementation language used by the microbe_masst batch scripts to parse and filter .mgf spectra before submitting them to the Fast Search API.) — https://github.com/robinschmid/microbe_masst
- **microbeMASST** (Downstream consumer of the filtered .mgf file; the parsed spectra are submitted as a batch input against the microbial metabolomics database.) — https://masst.gnps2.org/microbemasst/

## Examples

```
python jobs.py  # with ("input_directory/input_file", "output_directory/output_prefix") entries added to the files list, using Python 3.10 and skip_existing=True
```

## Evaluation signals

- The filtered .mgf file contains exactly the expected number of spectra (e.g., 10,047 for the GF/SPF microbeMASST workflow).
- Every retained record still carries valid PEPMASS, RTINSECONDS, and SCANS metadata so the Fast Search API and molecular networking joins stay intact.
- No retained spectrum has an edge with cosine > 0.7 and delta parent mass within ±0.02 Da to a GF-only spectrum, matching the paper's removal rule.
- Re-parsing the output .mgf yields the same scan set and peak lists as the input (lossless round-trip apart from the filtered-out records).

## Limitations

- Fast Search API batch calls can fail intermittently; the README instructs re-running jobs.py a couple of times with skip_existing=True until no new output appears.
- Python version must be 3.10 as stated in the README for the batch scripts.
- The .mgf produced by MZmine from GNPS molecular networking is the documented input format; other spectral formats require conversion first.

## Evidence

- [results] cosine threshold and mass-difference filtering rule for GF/SPF subsetting: [evidence span withheld — anchor status: not_found]
- [readme] input format expectation for the batch script: [evidence span withheld — anchor status: not_found]
- [readme] Python version requirement: "Please make user to use **_Python 3.10_**"
- [readme] need to re-run the batch to catch failed entries: "Due to the Fast Search API some of the entries will fail. Nevertheless sequent re-runs should catch all the possible matches."

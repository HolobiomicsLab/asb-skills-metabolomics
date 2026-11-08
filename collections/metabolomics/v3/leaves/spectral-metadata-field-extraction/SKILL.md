---
name: spectral-metadata-field-extraction
description: Use when MS/MS spectra arrive as MGF-format library files and you need
  to extract per-spectrum metadata (precursor m/z, adduct, InChIKey/SMILES) to tabulate
  spectrum counts and unique-compound counts.
when_to_use_negative:
- Input is raw instrument data (mzML/mzXML) rather than curated MGF library files
  that already carry harmonized metadata
- The goal is spectral similarity scoring or denoising rather than extracting and
  counting metadata fields from library records
- You need to re-run the curation filters themselves (e.g., library_cleaning_1.yaml
  / library_cleaning_2.yaml) rather than extract fields from already-curated spectra
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Parses MGF-format MS/MS library files into spectrum objects and exposes metadata
    fields (structure annotations, precursor m/z, adduct, charge) for extraction;
    the authors used a modified matchms zip with a fixed _get_neutral_mass function
  repo: https://github.com/matchms/matchms
- name: Python
  role: Runtime environment (Python 3.10 or higher) in which the parsing and counting
    scripts are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:c1a86a2892a93bdac5c191fa92076ceceda2710a59ac304f13c5ea2ebbd6e3a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# spectral-metadata-field-extraction

## Summary

Parses MGF-format MS/MS library records with matchms and extracts standardized metadata fields—peak lists plus structure annotations such as InChIKey/SMILES, precursor m/z, adduct, and charge—so library contents can be counted, deduplicated, and verified against reported totals.

## When to use

Use when MS/MS spectra arrive as MGF-format library files and you need to extract per-spectrum metadata (precursor m/z, adduct, InChIKey/SMILES) to tabulate spectrum counts and unique-compound counts.

## When NOT to use

- Input is raw instrument data (mzML/mzXML) rather than curated MGF library files that already carry harmonized metadata
- The goal is spectral similarity scoring or denoising rather than extracting and counting metadata fields from library records
- You need to re-run the curation filters themselves (e.g., library_cleaning_1.yaml / library_cleaning_2.yaml) rather than extract fields from already-curated spectra

## Inputs

- MGF-format MS/MS library files from the Spectraverse v1.0.1 Zenodo archive (doi:10.5281/zenodo.17870921)
- Spectra records carrying standardized structure annotations (InChIKey/SMILES), PRECURSOR_MZ, adduct, and charge metadata

## Outputs

- matchms spectrum objects with parsed peaks and metadata
- Total spectrum count (expected 488,630)
- Unique-compound count after de-duplication on structure identifier (expected 44,237)
- summary_statistics.csv with verified counts

## How to apply

Download the Spectraverse v1.0.1 archive from Zenodo (10.5281/zenodo.17870921) and parse all MS/MS records from the MGF-format library files into matchms spectrum objects under Python 3.10+. From each spectrum object, read the standardized structure annotations (InChIKey/SMILES), precursor and adduct metadata (including [M]+ / [M]− adducts with a reported charge of zero, which the paper's patched _get_neutral_mass function handles specially), and the peak/fragment arrays. Count total parsed spectra records, then de-duplicate compounds on the structure identifier to obtain unique small-molecule counts. Compare computed counts against reported library totals—here 488,630 spectra and 44,237 unique small molecules—and write the verified numbers to a summary table such as summary_statistics.csv.

## Related tools

- **matchms** (Parses MGF-format MS/MS library files into spectrum objects and exposes metadata fields (structure annotations, precursor m/z, adduct, charge) for extraction; the authors used a modified matchms zip with a fixed _get_neutral_mass function) — https://github.com/matchms/matchms
- **Python** (Runtime environment (Python 3.10 or higher) in which the parsing and counting scripts are executed)

## Examples

```
python -c "from matchms.importing import load_from_mgf; s = list(load_from_mgf('spectraverse_v1.0.1.mgf')); print(len(s), len({x.get('inchikey') for x in s}))"
```

## Evaluation signals

- Total parsed spectrum count must equal 488,630
- Unique-compound count after InChIKey/SMILES de-duplication must equal 44,237
- Every parsed spectrum record must carry a parseable standardized structure identifier (InChIKey or SMILES)
- Adduct/charge metadata must be consistent: spectra annotated with [M]+ or [M]− adducts with reported charge of zero are handled by the patched _get_neutral_mass logic rather than crashing mass computation
- Verified counts are written to summary_statistics.csv; any discrepancy flags a parsing or counting error

## Limitations

- Count checks only validate against the reported v1.0.1 totals; the paper notes no changelog is available for the resource
- Curation relied on manual review of only several thousand spectra, so some metadata pitfalls may remain beyond what was reviewed
- Script version numbers are not stated beyond the Python 3.10 requirement, so exact parser behavior may vary with tool versions

## Evidence

- [intro] Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules and is freely available from Zenodo: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small
molecules and is freely available from Zenodo"
- [other] Parse all MS/MS records from the MGF-format library files into spectrum objects using matchms under Python 3.10+: [evidence span withheld — anchor status: not_found]
- [methods] this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero: "this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS): "Matchms is a versatile open-source Python package developed for
importing, processing, cleaning, and comparing mass spectrometry data
(MS/MS)"

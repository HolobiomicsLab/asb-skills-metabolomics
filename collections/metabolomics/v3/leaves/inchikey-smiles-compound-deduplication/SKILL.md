---
name: inchikey-smiles-compound-deduplication
description: Use when a curated MS/MS library needs unique-compound counts for reporting
  or validation, and each spectrum carries structure metadata (InChIKey or SMILES)
  that can serve as the de-duplication key.
when_to_use_negative:
- Input spectra lack standardized structure annotations (no InChIKey/SMILES); de-duplicating
  on precursor m/z or adduct would conflate distinct isomers and compounds.
- The goal is per-spectrum bookkeeping (adducts, ionization modes, charge states)
  rather than compound-level statistics.
- Structure representations have not been harmonized yet; unstandardized SMILES variants
  (different tautomers or stereochemistry encodings) inflate the unique-compound count.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Parses MGF MS/MS records and provides metadata harmonization so structure
    annotations are in a standardized form before de-duplication counting.
  repo: https://github.com/matchms/matchms
- name: Python
  role: Runtime for counting unique structure identifiers across parsed spectra and
    writing summary statistics.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:47a76971758f2832a50172cfc3489e3df806de4a1a7618fc6b569fc92af6443a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# InChIKey/SMILES compound deduplication

## Summary

De-duplicates MS/MS spectra by mapping each record's standardized chemical structure annotations (InChIKey/SMILES) to unique compound identifiers, so that library statistics report molecules rather than spectra. Used in Spectraverse to verify that 488,630 curated spectra collapse onto 44,237 unique small molecules.

## When to use

Use when a curated MS/MS library needs unique-compound counts for reporting or validation, and each spectrum carries structure metadata (InChIKey or SMILES) that can serve as the de-duplication key.

## When NOT to use

- Input spectra lack standardized structure annotations (no InChIKey/SMILES); de-duplicating on precursor m/z or adduct would conflate distinct isomers and compounds.
- The goal is per-spectrum bookkeeping (adducts, ionization modes, charge states) rather than compound-level statistics.
- Structure representations have not been harmonized yet; unstandardized SMILES variants (different tautomers or stereochemistry encodings) inflate the unique-compound count.

## Inputs

- Curated MS/MS library records in MGF format with per-spectrum structure annotations (InChIKey/SMILES)
- Spectraverse v1.0.1 archive from Zenodo (doi:10.5281/zenodo.17870921)

## Outputs

- Unique-compound count keyed on InChIKey/SMILES
- Spectrum count vs. unique-compound count summary (e.g. summary_statistics.csv)

## How to apply

Parse the curated library records (MGF-format MS/MS spectra, e.g. via matchms) and extract each record's standardized structure annotation. Standardize representations first, as the Spectraverse pipeline explicitly harmonizes 'inconsistent representations of chemical structures' before counting. De-duplicate on the structure identifier (InChIKey first-layer preferred, or canonical SMILES) and count unique keys to obtain the molecule count; keep the raw spectrum count separately so both totals (spectra vs. unique compounds) are reported. Compare the computed unique-compound count against the paper-reported figure (44,237 unique small molecules over 488,630 spectra) to confirm that filtering and structure standardization have not silently introduced duplicates or lost annotations.

## Related tools

- **matchms** (Parses MGF MS/MS records and provides metadata harmonization so structure annotations are in a standardized form before de-duplication counting.) — https://github.com/matchms/matchms
- **Python** (Runtime for counting unique structure identifiers across parsed spectra and writing summary statistics.)

## Examples

```
python -c "import matchms; spectra = list(matchms.importing.load_from_mgf('data/spectraverse.mgf')); keys = {s.get('inchikey') or s.get('smiles') for s in spectra}; print(len(spectra), len(keys))"
```

## Evaluation signals

- Tabulated unique-compound count equals 44,237 for the final curated Spectraverse library (488,630 spectra); any discrepancy flags a parsing or counting error.
- Total spectrum count equals 488,630 and no records are lost or double-counted between spectrum-level and compound-level aggregation.
- Every spectrum carries a non-empty, standardized InChIKey or SMILES before counting; spectra with missing structure annotations are reported separately.
- De-duplication is idempotent: re-running the count on the same parsed library yields an identical unique-compound count.

## Limitations

- Counts are only as accurate as the upstream structure standardization; the paper identifies 'inconsistent representations of chemical structures' as a pitfall that must be resolved before compound counting.
- Structure annotations may be absent or incoherent for some public library entries, so the unique-compound count can underestimate true chemical diversity if such spectra are excluded rather than annotated.
- Near-duplicate spectra removal is reported for MassSpecGym comparisons, but the completeness of de-duplication for Spectraverse itself is not explicitly stated in the discussion section.

## Evidence

- [intro] Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules and is freely available from Zenodo"
- [intro] preprocessing pipeline standardizes chemical structure representations: "preprocessing pipeline to identify high-quality MS/MS spectra, curate and harmonize their associated metadata, and standardize the representations of their"
- [discussion] inconsistent representations of chemical structures: "the presence of low-resolution spectra, fragment ions with zero or uniform intensities, or inconsistent representations of chemical structures"

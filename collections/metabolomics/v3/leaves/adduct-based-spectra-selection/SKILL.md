---
name: adduct-based-spectra-selection
description: Use when building a benchmarking set from concatenated GNPS spectral
  libraries (e.g. 66,646 entries) and you need to restrict spectra to protonated [M+H]+
  adducts that carry structural identifiers before InChIKey conversion and de-duplication.
when_to_use_negative:
- Input spectra are [M-H]− or other adduct types and you need a negative-mode benchmarking
  set — this skill only retains protonated [M+H]+ entries.
- Library entries lack structural identifiers, so InChIKey conversion and de-duplication
  are impossible and adduct-based selection alone is insufficient.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mgf_filterer
  role: Filters .mgf spectral files by peak-count and intensity criteria, feeding
    the parsed entries that are then selected by adduct.
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: GNPS_parser_cleaner
  role: Parses each library entry and cleans metadata fields, applying the [M+H]+
    adduct filter and structural-identifier requirement.
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: RDKit
  role: Converts retained structures of adduct-selected entries to InChIKeys for de-duplication.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:0fd5cc4d73e97f2ef44275f345b74354416f6b433c138be363f3e86eab990438
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taxonomicallyinformedannotation/tima
  tools:
  - GNPS_parser_cleaner
  - RDKit
  - mgf_filterer
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# adduct-based-spectra-selection

## Summary

Filters GNPS library .mgf spectral entries by adduct type — keeping only [M+H]+ protonated adducts with structural identifiers — so that benchmarking spectra are chemically interpretable and comparable across annotation tools.

## When to use

Use when building a benchmarking set from concatenated GNPS spectral libraries (e.g. 66,646 entries) and you need to restrict spectra to protonated [M+H]+ adducts that carry structural identifiers before InChIKey conversion and de-duplication.

## When NOT to use

- Input spectra are [M-H]− or other adduct types and you need a negative-mode benchmarking set — this skill only retains protonated [M+H]+ entries.
- Library entries lack structural identifiers, so InChIKey conversion and de-duplication are impossible and adduct-based selection alone is insufficient.

## Inputs

- Concatenated GNPS and third-party spectral library .mgf files (66,646 entries before preprocessing)
- Library entry metadata including adduct annotation and structural identifiers (SMILES/InChI)

## Outputs

- Subset of .mgf entries restricted to [M+H]+ adducts with structural identifiers, feeding RDKit InChIKey conversion and downstream mass/peak filters
- gnps_library_filtered.tsv with 40,138 spectral entries over 8,558 unique first-layer InChIKeys

## How to apply

Parse each cleaned library entry from the .mgf file and inspect its metadata adduct field: retain only entries annotated as protonated [M+H]+ adducts that also possess structural identifiers (e.g. SMILES or InChI usable by RDKit). The rationale is that only protonated adducts make the parent-mass and fragmentation behavior directly comparable to computational annotation tools and allow exact-mass checking; entries lacking structural identifiers cannot be converted to InChIKeys and cannot be de-duplicated. After adduct selection, convert retained structures to InChIKeys with RDKit and continue the downstream filters (parent mass 100–1,500 Da, ≥6 fragments, top 500 most intense peaks). Validation of the adduct step is indirect: the final gnps_library_filtered.tsv must contain 40,138 spectral entries over 8,558 unique first-layer InChIKeys, all protonated, with parent masses within 100–1,500 Da.

## Related tools

- **mgf_filterer** (Filters .mgf spectral files by peak-count and intensity criteria, feeding the parsed entries that are then selected by adduct.) — https://github.com/taxonomicallyinformedannotation/tima
- **GNPS_parser_cleaner** (Parses each library entry and cleans metadata fields, applying the [M+H]+ adduct filter and structural-identifier requirement.) — https://github.com/taxonomicallyinformedannotation/tima
- **RDKit** (Converts retained structures of adduct-selected entries to InChIKeys for de-duplication.)

## Examples

```
Rscript parse_gnps_library.R --input gnps_library.mgf --adduct '[M+H]+' --output gnps_library_filtered.tsv
```

## Evaluation signals

- After adduct selection, every retained entry's adduct field equals [M+H]+ and has a parseable structural identifier convertible by RDKit.
- Downstream output gnps_library_filtered.tsv contains exactly 40,138 spectral entries over 8,558 unique first-layer (first 14 characters) InChIKeys.
- All retained entries have parent masses within 100–1,500 Da and spectra with ≥6 fragments (≤500 most intense peaks).
- No duplicate or stereoisomeric structures remain after first-layer InChIKey de-duplication.

## Limitations

- Only [M+H]+ protonated adducts are kept, so the resulting benchmarking set does not represent negative-mode or in-source fragment spectra.
- 197 entries of the benchmarking dataset were later found to have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass, which adduct filtering alone does not catch.
- Entries lacking structural identifiers are dropped even if spectrally valid, reducing coverage.

## Evidence

- [discussion] keep only [M+H]+ protonated adducts with structural identifiers: [evidence span withheld — anchor status: not_found]
- [discussion] parent masses between 100 and 1,500 Da: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] retain entries with ≥6 fragments, 500 most intense peaks: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] benchmarking dataset of 2107 distinct entries: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [readme] README: mgf backend handles Mascot Generic Format files: "Mass Spectrometry Data Backend for Mascot Generic Format (mgf) Files"

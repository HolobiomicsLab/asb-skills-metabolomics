---
name: adduct-annotation-coherence-checking
description: Use when curating a reference MS/MS library (e.g. MGF-format spectra
  with SMILES, adduct, ionization mode, and precursor m/z metadata) where chemically
  incoherent annotations must be detected and removed before library cleaning or machine-learning
  model training.
when_to_use_negative:
- Query spectra in a library-search or spectral-annotation workflow — this is a library-cleaning
  filter, not a search-time annotation validator
- Spectra lacking SMILES or parent-mass metadata, since the 0.1 Da parent-mass-vs-SMILES
  check cannot be evaluated and spectra will simply be dropped by REQUIRE_COMPLETE_METADATA/REQUIRE_COMPLETE_ANNOTATION
- Input that has already passed library_cleaning_2.yaml; re-running the coherence
  checks on the already-filtered output is the wrong stage in the prescribed pipeline
  order
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: matchms
  role: Executes the two-round library cleaning; the patched version supplies the
    REQUIRE_COMPLETE_ANNOTATION coherence filter and the modified _get_neutral_mass
  repo: https://github.com/matchms/matchms
- name: spectraverse-analysis
  role: Provides the declarative filter configurations library_cleaning_1.yaml and
    library_cleaning_2.yaml and the patched matchms zip used in the coherence-checking
    round
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: Python
  role: Runtime environment (Python 3.10 or higher) in which the matchms cleaning
    rounds and per-filter tallying scripts are run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:922b666e0ff88a7b67d08059581fae64081c102c56574ee0f3b99cc8b118d68a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# adduct-annotation-coherence-checking

## Summary

Validates that MS/MS library spectrum annotations are chemically coherent with their own metadata: the adduct must match the precursor m/z within 0.1 Da and the ionization mode, and the reported parent mass must match the SMILES-derived neutral mass within 0.1 Da. In Spectraverse this is implemented as the matchms REQUIRE_COMPLETE_ANNOTATION filter in the second cleaning round, together with a patched _get_neutral_mass that ignores [M]+/[M]− adducts reporting a charge of zero.

## When to use

Use when curating a reference MS/MS library (e.g. MGF-format spectra with SMILES, adduct, ionization mode, and precursor m/z metadata) where chemically incoherent annotations must be detected and removed before library cleaning or machine-learning model training.

## When NOT to use

- Query spectra in a library-search or spectral-annotation workflow — this is a library-cleaning filter, not a search-time annotation validator
- Spectra lacking SMILES or parent-mass metadata, since the 0.1 Da parent-mass-vs-SMILES check cannot be evaluated and spectra will simply be dropped by REQUIRE_COMPLETE_METADATA/REQUIRE_COMPLETE_ANNOTATION
- Input that has already passed library_cleaning_2.yaml; re-running the coherence checks on the already-filtered output is the wrong stage in the prescribed pipeline order

## Inputs

- Merged MS/MS spectral library in MGF format (with precursor m/z, ionization mode, adduct, and SMILES metadata)
- matchms/library_cleaning_1.yaml declarative filter configuration (round 1)
- matchms/library_cleaning_2.yaml declarative filter configuration (round 2, including REQUIRE_COMPLETE_METADATA and REQUIRE_COMPLETE_ANNOTATION)
- Patched matchms installation in which _get_neutral_mass ignores [M]+/[M]− spectra with reported charge of zero

## Outputs

- Filtered, metadata-repaired MGF written to the data/ directory feeding the subsequent config_step2/config_step3 preprocessing stages
- Per-filter removal-count report tallying spectra removed during both cleaning rounds

## How to apply

Load the merged MS/MS library in MGF format and first run matchms round 1 with library_cleaning_1.yaml (HARMONIZE_METADATA_FIELD_NAMES, DERIVE_METADATA_IN_WRONG_FIELD, HARMONIZE_METADATA_ENTRIES, DERIVE_MISSING_METADATA, REPAIR_ANNOTATION, NORMALIZE_INTENSITY) so that metadata fields and annotations are harmonized before coherence testing. Then run round 2 with matchms/library_cleaning_2.yaml, which retains the round-1 filters and adds REQUIRE_COMPLETE_METADATA (valid precursor m/z and ionization mode) and REQUIRE_COMPLETE_ANNOTATION: the parent mass is verified against the SMILES within 0.1 Da, the adduct against the precursor m/z within 0.1 Da, and the adduct against the ionization mode. Use a patched matchms whose _get_neutral_mass ignores spectra annotated with [M]+ or [M]− adducts that report a charge of zero, since such net-charge-zero spectra make neutral-mass derivation unreliable. The rationale is that public libraries contain chemically incoherent annotations that would otherwise confound model training; spectra failing any check are dropped, and removal counts are tallied per filter.

## Related tools

- **matchms** (Executes the two-round library cleaning; the patched version supplies the REQUIRE_COMPLETE_ANNOTATION coherence filter and the modified _get_neutral_mass) — https://github.com/matchms/matchms
- **spectraverse-analysis** (Provides the declarative filter configurations library_cleaning_1.yaml and library_cleaning_2.yaml and the patched matchms zip used in the coherence-checking round) — https://github.com/skinniderlab/spectraverse-analysis
- **Python** (Runtime environment (Python 3.10 or higher) in which the matchms cleaning rounds and per-filter tallying scripts are run)

## Examples

```
matchms run matchms/library_cleaning_2.yaml
```

## Evaluation signals

- Every retained spectrum passes REQUIRE_COMPLETE_ANNOTATION: parent mass matches the SMILES-derived neutral mass within 0.1 Da, and the adduct matches the precursor m/z within 0.1 Da
- Adduct charge state and polarity are consistent with the recorded ionization mode for all retained spectra
- Spectra annotated with [M]+ or [M]− adducts reporting a charge of zero are absent from the output MGF (caught by the patched _get_neutral_mass)
- A per-filter removal-count report is produced, with non-zero counts for REQUIRE_COMPLETE_METADATA and REQUIRE_COMPLETE_ANNOTATION where incoherent spectra existed
- The output MGF loads cleanly into the subsequent config_step2/config_step3 preprocessing stages in the prescribed order

## Limitations

- Curation relied on manual review of only several thousand MS/MS spectra, so annotation pitfalls beyond those reviewed may remain
- The 0.1 Da tolerance for the parent-mass-vs-SMILES and adduct-vs-precursor checks is pipeline-specific and may discard valid spectra with imprecise precursor m/z or nonstandard adduct annotations
- The check requires the patched matchms; the stock _get_neutral_mass does not ignore [M]+/[M]− spectra with a reported charge of zero, so results are not reproducible with unpatched matchms

## Evidence

- [methods] Round 2 matchms applies REQUIRE_COMPLETE_METADATA and REQUIRE_COMPLETE_ANNOTATION filters: "Run matchms again using filters specified in matchms/library_cleaning_2.yaml."
- [methods] Patched _get_neutral_mass ignores zero-charge [M]+/[M]− spectra: "this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero."
- [readme] matchms provides metadata cleaning and validation functionality underlying the filter: [evidence span withheld — anchor status: external_doc]
- [abstract] Incoherent annotations in public libraries motivated the filtering: "public spectral libraries are scattered across disparate databases and contain spectra that are of low resolution or quality, missing critical metadata, or which have chemically incoherent annotations"

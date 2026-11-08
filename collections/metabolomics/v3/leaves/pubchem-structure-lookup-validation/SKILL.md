---
name: pubchem-structure-lookup-validation
description: Use when library spectra carry only a compound name (no SMILES), or when
  validating existing SMILES/InChI/InChIKey annotations against each other in a mass
  spectral library cleaning pipeline.
when_to_use_negative:
- The spectrum already has a complete, validated SMILES, InChI, and InChIKey that
  were all derived from the same original structure — the lookup adds nothing and
  could overwrite a good annotation
- The compound name is missing, empty, or so generic (e.g. a trivial abbreviation)
  that a unique PubChem hit cannot be expected; then the filter would yield no candidate
  or a wrong one
- You need to check whether measured fragments actually match the annotation — the
  article notes such plausibility checks over both metadata and fragments are not
  yet implemented in matchms
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3418
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides the 'derive annotation from compound name' filter (v0.26.4) that
    queries PubChem and enforces parent-mass consistency before accepting a structure
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads the recovered SMILES, InChI, and InChIKey and compares them to detect
    incomplete or mismatched annotations
- name: PubChem
  role: External database queried for canonical SMILES, InChI, and InChIKey based
    on the compound name; its mass is used as a filter criterion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:af051e6908b17c3f8406155d3312bcc4bbd4d6e9d98e621aef4d1d7ae78a2c42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3418
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pubchem-structure-lookup-validation

## Summary

Derives canonical SMILES, InChI, and InChIKey from PubChem using the compound name in spectrum metadata, accepting a candidate only when its PubChem mass matches the spectrum's parent mass, then validates the recovered structures by loading them with RDKit and cross-comparing the three identifier forms.

## When to use

Use when library spectra carry only a compound name (no SMILES), or when validating existing SMILES/InChI/InChIKey annotations against each other in a mass spectral library cleaning pipeline.

## When NOT to use

- The spectrum already has a complete, validated SMILES, InChI, and InChIKey that were all derived from the same original structure — the lookup adds nothing and could overwrite a good annotation
- The compound name is missing, empty, or so generic (e.g. a trivial abbreviation) that a unique PubChem hit cannot be expected; then the filter would yield no candidate or a wrong one
- You need to check whether measured fragments actually match the annotation — the article notes such plausibility checks over both metadata and fragments are not yet implemented in matchms

## Inputs

- MS/MS spectrum metadata records from mass spectral libraries (e.g. GNPS public library .msp/.mgf entries) containing a compound name and optionally SMILES/InChI/InChIKey
- Spectrum parent mass (monoisotopic) from metadata
- Pre-computed compound-name-to-structure lookup file (cache of PubChem queries)

## Outputs

- Canonical SMILES, InChI, and InChIKey strings attached to each spectrum whose name lookup succeeded and passed the parent-mass consistency check
- Spectra removed or flagged where annotation is incomplete or identifier forms disagree
- Error-rate estimates: fraction of names successfully derived (e.g. 72.4%), fraction not derivable (27.6%), and fraction with incorrect 2D structures (1.62%)

## How to apply

For each spectrum, look up canonical SMILES, InChI, and InChIKey in PubChem from the compound name field; accept the structure only if the PubChem mass matches the parent mass recorded in the spectrum metadata, so wrong hits with similar names are rejected. Load the recovered SMILES, InChI, and InChIKey with RDKit and cross-compare them — if any of the three is incomplete or inconsistent with the others, discard the spectrum rather than keep a partially repaired annotation. The article reports this filter in matchms 0.26.4 recovered SMILES from compound names for 72.4% of a GNPS test set, leaving 27.6% un-derivable, and found 1.62% of successfully annotated spectra had a different 2D structure than the original annotation. Reuse a stored compound-name-to-SMILES lookup file to minimize repeated PubChem queries when processing large libraries.

## Related tools

- **matchms** (Provides the 'derive annotation from compound name' filter (v0.26.4) that queries PubChem and enforces parent-mass consistency before accepting a structure) — https://github.com/matchms/matchms
- **RDKit** (Loads the recovered SMILES, InChI, and InChIKey and compares them to detect incomplete or mismatched annotations)
- **PubChem** (External database queried for canonical SMILES, InChI, and InChIKey based on the compound name; its mass is used as a filter criterion)

## Examples

```
from matchms import Pipeline; p = Pipeline('library-cleaning.yaml'); p.run()
```

## Evaluation signals

- On a GNPS error-estimation subset, the derived-from-name rate should land near 72.4% and the not-derivable rate near 27.6%
- Among successfully re-annotated spectra, the fraction flagged with a different 2D structure should be about 1.62% (matching the article's reported estimate)
- No spectrum is accepted whose PubChem-derived mass does not match the recorded parent mass — check that all accepted hits pass the mass-consistency rule
- For every accepted spectrum, RDKit must successfully parse all three identifier forms (SMILES, InChI, InChIKey) and they must agree with each other; any failure means the spectrum is removed, not partially patched
- The processing report should record how many spectra were changed or removed by this filter compared to the input library count

## Limitations

- Only 72.4% of compound names yield a usable structure; 27.6% of names fail to produce a SMILES at all, so this filter cannot complete annotations for all entries
- Even among successful lookups, 1.62% of annotations recover a different 2D structure than the original — the method cannot detect when the wrong compound was retrieved if its mass happens to match the parent mass
- The filter checks only name-to-mass consistency; it performs no plausibility check on whether the measured fragments match the retrieved annotation
- Repeated PubChem queries can be slow for large libraries; the pipeline mitigates this by storing a compound-name lookup file for reuse

## Evidence

- [methods] PubChem lookup constrained by parent mass: "This filter derives the canonical SMILES, InChI and InChIKey from PubChem."
- [discussion] Mass-match acceptance rule: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
- [methods] RDKit cross-validation of identifier forms: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
- [methods] Discard incomplete annotations: "If one of these is not complete or does not match the spectrum will be removed."
- [results] Error-rate estimate from source task card: [evidence span withheld — anchor status: not_found]

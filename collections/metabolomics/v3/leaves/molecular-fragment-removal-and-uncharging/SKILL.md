---
name: molecular-fragment-removal-and-uncharging
description: Use when raw natural-product structure tables (e.g. smiles.tsv.gz from
  LOTUS sources) contain salts, charged species or dimeric SMILES entries that must
  be reduced to single, neutral, parent molecules before structure deduplication or
  standardization to InChI.
when_to_use_negative:
- Input structures are already single-component neutral SMILES with no salts, charges,
  or dimers — applying MolVS sanitization can still degrade them ('correct salts,
  charged or dimeric molecules in the input data will suffer an unwanted sanitization
  step'), so skipping the step preserves fidelity.
- The goal is name-to-structure conversion of chemical names rather than cleaning
  existing SMILES; in LOTUS that is handled separately by translating names to InChI
  with OPSIN before/independent of sanitization.
- You need a broad alternative natural-product cleaning pipeline rather than the LOTUS
  three-stage MolVS approach; the article notes 'the current processing pipeline still
  removes a lot of correct entries and misses or induces some incorrect ones', so
  results are not interchangeable with other curation workflows.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: MolVS
  role: Python package that performs the three-stage sanitization (standardizing,
    fragment removal, uncharging) on RDKit molecule objects in the LOTUS pipeline.
  repo: https://github.com/lotusnprod
- name: sanitizing.py
  role: Script in the lotus-processor 3_cleaningAndEnriching module that orchestrates
    the MolVS sanitization stages.
  repo: https://github.com/lotusnprod/lotus-processor
- name: smiles.py
  role: Script handling the SMILES-side processing of the structure tables prior to
    sanitization.
  repo: https://github.com/lotusnprod/lotus-processor
- name: RDKit
  role: Used to parse SMILES strings into molecule objects on which MolVS sanitization
    operates.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:ba0cbbce09fc8e3bff007dabcb6eefa54588b198f63933c8178dc0d4f1a9bef8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MolVS
  - RDKit
  - sanitizing.py
  - smiles.py
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# molecular-fragment-removal-and-uncharging

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A chemical sanitization sub-step of the LOTUS processing pipeline in which SMILES-derived molecule objects are standardized, stripped of salts/fragments, and uncharged via the MolVS package before being written into the cleaned structure table with standardized InChI identifiers.

## When to use

Use when raw natural-product structure tables (e.g. smiles.tsv.gz from LOTUS sources) contain salts, charged species or dimeric SMILES entries that must be reduced to single, neutral, parent molecules before structure deduplication or standardization to InChI.

## When NOT to use

- Input structures are already single-component neutral SMILES with no salts, charges, or dimers — applying MolVS sanitization can still degrade them ('correct salts, charged or dimeric molecules in the input data will suffer an unwanted sanitization step'), so skipping the step preserves fidelity.
- The goal is name-to-structure conversion of chemical names rather than cleaning existing SMILES; in LOTUS that is handled separately by translating names to InChI with OPSIN before/independent of sanitization.
- You need a broad alternative natural-product cleaning pipeline rather than the LOTUS three-stage MolVS approach; the article notes 'the current processing pipeline still removes a lot of correct entries and misses or induces some incorrect ones', so results are not interchangeable with other curation workflows.

## Inputs

- raw natural-product structure table smiles.tsv.gz (SMILES strings)
- compound-name table nominal.tsv.gz (chemical names, translated to InChI via OPSIN when no structural identifier is present)
- RDKit molecule objects parsed from SMILES

## Outputs

- sanitized RDKit molecule objects (standardized, fragment-removed, uncharged)
- standardized InChI strings for each sanitized structure
- cleaned structure table interim/tables/2_cleaned/structure/cleaned.tsv.gz (gzipped TSV with InChI column)

## How to apply

Parse each SMILES string from the raw structure table (smiles.tsv.gz) into an RDKit molecule object, discarding dimeric entries (all structures containing a '.' in their SMILES) as the LOTUS pipeline does before sanitization. Then run the MolVS sanitization routine in three main stages — standardizing, fragment removal, and uncharging — as implemented in the sanitizing.py script of the lotus-processor 3_cleaningAndEnriching module. The rationale is that 'correct salts, charged or dimeric molecules in the input data will suffer an unwanted sanitization step' if these cases are not systematically reduced, and fragment removal collapses multi-component entries to the parent natural-product molecule. After sanitization, generate the standardized InChI for each molecule and write the records as the cleaned structure table interim/tables/2_cleaned/structure/cleaned.tsv.gz in gzipped TSV format. Note the pipeline is convoluted with many dependencies; the sanitizing.py helper is invoked from the lotus-processor repository rather than reimplemented.

## Related tools

- **MolVS** (Python package that performs the three-stage sanitization (standardizing, fragment removal, uncharging) on RDKit molecule objects in the LOTUS pipeline.) — https://github.com/lotusnprod
- **sanitizing.py** (Script in the lotus-processor 3_cleaningAndEnriching module that orchestrates the MolVS sanitization stages.) — https://github.com/lotusnprod/lotus-processor
- **smiles.py** (Script handling the SMILES-side processing of the structure tables prior to sanitization.) — https://github.com/lotusnprod/lotus-processor
- **RDKit** (Used to parse SMILES strings into molecule objects on which MolVS sanitization operates.)

## Examples

```
python 3_cleaningAndEnriching/sanitizing.py --input raw/tables/1_noEdit/structure/smiles.tsv.gz --output interim/tables/2_cleaned/structure/cleaned.tsv.gz
```

## Evaluation signals

- Check that no retained record in cleaned.tsv.gz contains a '.' in its SMILES field (the dimer-discard rule).
- Verify the cleaned output file interim/tables/2_cleaned/structure/cleaned.tsv.gz exists, is gzipped TSV, and has a populated InChI column.
- Confirm that each output molecule is uncharged (no net formal charge) after the uncharging stage.
- Confirm that fragment-removed structures no longer carry leftover counterions/salts (single-component parent molecule).
- Cross-check that a sample of sanitized structures still round-trips to valid InChI when re-parsed by RDKit/InChI tooling.

## Limitations

- The sanitization is systematic, so correct salts, charged or dimeric molecules in the input data suffer an unwanted sanitization step.
- The LOTUS processing pipeline is heavy, includes many dependencies, and is convoluted; it still removes a lot of correct entries and misses or induces some incorrect ones.
- Only structures were dimer-discarded by the '.'-in-SMILES rule; name-based (OPSIN-translated) entries follow a separate path and are not covered by this sanitization step alone.

## Evidence

- [results] dimeric structures discarded before cleaning: [evidence span withheld — anchor status: not_found]
- [results] OPSIN used for names without structural identifiers: [evidence span withheld — anchor status: not_found]

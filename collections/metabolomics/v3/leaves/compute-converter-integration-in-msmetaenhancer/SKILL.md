---
name: compute-converter-integration-in-msmetaenhancer
description: Use when annotating .msp mass spectral library records that already carry
  SMILES (or similar existing metadata) and a derived attribute such as molecular
  weight should be computed locally instead of querying external services.
when_to_use_negative:
- Records lack SMILES or any source attribute the converter is keyed on — the conversion
  method must return an empty dict and no metadata is gained.
- The target metadata must come from an authoritative external source rather than
  being locally derivable — use the web converters (CIR, CTS, PubChem, IDSM, BridgeDb)
  instead.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSMetaEnhancer
  role: Hosts the Application, Job system, ComputeConverter base class, ConverterBuilder
    registration, and matchms-based .msp load/save used to apply the local conversion
    to every record
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: RDKit
  role: Parses SMILES locally (MolFromSmiles) and computes molecular weight from the
    molecule object inside the ComputeConverter conversion method
  repo: https://doi.org/10.5281/zenodo.6961488
- name: matchms
  role: Provides the data IO used by MSMetaEnhancer to load and save the .msp records
    being annotated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:c98a2ab36f478029b824898b6d2ac8b670419cedb9699c0d5bafbcdc73392e91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - MSMetaEnhancer
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# compute-converter-integration-in-msmetaenhancer

## Summary

Integrate a local ComputeConverter (e.g. RDKit) into MSMetaEnhancer so identifiers such as molecular weight are computed in place from existing record metadata (SMILES) rather than fetched from external web services, applied via annotation jobs over .msp mass spectral library records.

## When to use

Use when annotating .msp mass spectral library records that already carry SMILES (or similar existing metadata) and a derived attribute such as molecular weight should be computed locally instead of querying external services.

## When NOT to use

- Records lack SMILES or any source attribute the converter is keyed on — the conversion method must return an empty dict and no metadata is gained.
- The target metadata must come from an authoritative external source rather than being locally derivable — use the web converters (CIR, CTS, PubChem, IDSM, BridgeDb) instead.

## Inputs

- Annotated .msp mass spectral library file (matchms-readable), with SMILES metadata present on records
- SMILES strings on individual spectra records

## Outputs

- annotated_spectra_with_mw.msp — .msp file whose records carry a molecular_weight field
- Per-record computed molecular weights stored via the MSMetaEnhancer Job system

## How to apply

Load the annotated .msp file with MSMetaEnhancer's matchms-based load routine (app.load_data(..., file_format='msp')). Create a ComputeConverter subclass whose conversions list maps ('smiles', 'molecular_weight', 'conversion_method') and call create_top_level_conversion_methods(conversions, asynch=False) so a top-level method like smiles_to_molecular_weight() is auto-generated, following the same pattern that auto-generates methods like compound_name_to_inchi() for web converters. In the conversion method, parse the SMILES string locally with RDKit (MolFromSmiles) and compute the molecular weight from the molecule object; return {'molecular_weight': value} so the Job system stores it on the record, returning an empty dict when SMILES is missing. Register the converter with ConverterBuilder (ConverterBuilder.register([...RDKit])), include it in the services list, and run the annotation job over all records, then export with app.save_data(..., file_format='msp'). The rationale is the paper's explicit support for computing identifiers locally instead of querying external databases, which avoids network dependence and rate limiting for derivable metadata.

## Related tools

- **MSMetaEnhancer** (Hosts the Application, Job system, ComputeConverter base class, ConverterBuilder registration, and matchms-based .msp load/save used to apply the local conversion to every record) — https://github.com/RECETOX/MSMetaEnhancer
- **RDKit** (Parses SMILES locally (MolFromSmiles) and computes molecular weight from the molecule object inside the ComputeConverter conversion method) — https://doi.org/10.5281/zenodo.6961488
- **matchms** (Provides the data IO used by MSMetaEnhancer to load and save the .msp records being annotated)

## Examples

```
from MSMetaEnhancer import Application; from MSMetaEnhancer.libs.converters.compute import RDKit; from MSMetaEnhancer.libs.utils.ConverterBuilder import ConverterBuilder; ConverterBuilder.register([RDKit]); app = Application(); app.load_data('sample.msp', file_format='msp')
```

## Evaluation signals

- Output .msp file exists and parses as valid .msp via matchms IO
- Every record with a non-empty SMILES carries a molecular_weight field in the output
- Computed molecular weights for a sampled subset of records exactly match weights recomputed from the same SMILES with RDKit
- Records without SMILES have no molecular_weight added (empty-dict return path exercised)
- Expert review for chemical plausibility of sampled computed weights

## Limitations

- Computation only applies where the source attribute (SMILES) exists on the record; missing metadata still requires external annotation or post-processing.
- Validation of obtained metadata relies on matchms validators; attribute validation tracking in logs was still an open issue (#68).
- The tool iteratively executes all steps per entry until no new metadata is found, so adding compute jobs interacts with the overall annotation loop.

## Evidence

- [abstract] Computed identifiers are supported in addition to external database queries, e.g. molecular weight from SMILES: "instead of querying external databases, computing the identifiers is also supported (e.g.molecular weight from SMILES)"
- [abstract] RDKit is the compute engine used for this conversion: "we employed computation of molecular weight from SMILES using RDKit (Landrum et al., 2022)"
- [methods] Converter pattern for auto-generated conversion methods: "Converters automatically generate methods like `compound_name_to_inchi()`"
- [readme] README shows the compute converter import and registration path: [evidence span withheld — anchor status: external_doc]
- [readme] README shows ConverterBuilder registration of the RDKit compute converter: [evidence span withheld — anchor status: external_doc]

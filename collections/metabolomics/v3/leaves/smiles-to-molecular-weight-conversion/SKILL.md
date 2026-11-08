---
name: smiles-to-molecular-weight-conversion
description: Use when annotated .msp spectra records already carry SMILES metadata
  but lack molecular weight, and you need a locally computed identifier without external
  database queries.
when_to_use_negative:
- Records lack SMILES metadata — the compute converter returns an empty dict and no
  molecular weight can be derived; fetch SMILES from an external service (CIR, CTS,
  IDSM, PubChem, BridgeDb) first
- You need identifiers that cannot be computed locally (e.g. CAS numbers, InChI from
  names) — those require MSMetaEnhancer's web converters querying external services
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: MSMetaEnhancer
  role: Framework providing ComputeConverter, Job system, and .msp load/save (load_data,
    save_data, annotate_spectra) used to apply the SMILES→molecular_weight conversion
    to library records
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: RDKit
  role: Parses SMILES locally (MolFromSmiles) and computes molecular weight from the
    molecule object without network access
  repo: https://doi.org/10.5281/zenodo.6961488
- name: matchms
  role: Underlying data IO and validators for the .msp format handled by MSMetaEnhancer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:da5cfb32096fdc488730a774e210ae16b027c190be4408eef625e72f0abcac50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - MSMetaEnhancer
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# smiles-to-molecular-weight-conversion

## Summary

Computes molecular weight locally from existing SMILES metadata in .msp mass spectral library records using RDKit inside MSMetaEnhancer's ComputeConverter framework, instead of querying external chemical databases.

## When to use

Use when annotated .msp spectra records already carry SMILES metadata but lack molecular weight, and you need a locally computed identifier without external database queries.

## When NOT to use

- Records lack SMILES metadata — the compute converter returns an empty dict and no molecular weight can be derived; fetch SMILES from an external service (CIR, CTS, IDSM, PubChem, BridgeDb) first
- You need identifiers that cannot be computed locally (e.g. CAS numbers, InChI from names) — those require MSMetaEnhancer's web converters querying external services

## Inputs

- Annotated .msp mass spectral library file whose records contain SMILES metadata
- SMILES strings per spectrum record

## Outputs

- annotated_spectra_with_mw.msp — .msp file with a molecular_weight metadata field added to each record that had SMILES
- Per-record computed molecular weight values matching RDKit calculations from the input SMILES

## How to apply

Load the annotated .msp file with MSMetaEnhancer's load_data (matchms-based .msp IO), then define a ComputeConverter subclass whose conversions list maps ('smiles', 'molecular_weight', 'conversion_method') and call create_top_level_conversion_methods(conversions, asynch=False) to auto-generate the top-level method. In the conversion method, parse the SMILES string locally with RDKit's MolFromSmiles and compute the molecular weight from the molecule object. Return the computed value as {'molecular_weight': value} so the Job system stores it on the record, following the key-principles pattern of returning a dictionary and returning an empty dict when SMILES is missing. Run the annotation job over all records that have SMILES, then save the result with save_data using file_format='msp'. Register the RDKit compute converter alongside web converters via ConverterBuilder.register before annotating.

## Related tools

- **MSMetaEnhancer** (Framework providing ComputeConverter, Job system, and .msp load/save (load_data, save_data, annotate_spectra) used to apply the SMILES→molecular_weight conversion to library records) — https://github.com/RECETOX/MSMetaEnhancer
- **RDKit** (Parses SMILES locally (MolFromSmiles) and computes molecular weight from the molecule object without network access) — https://doi.org/10.5281/zenodo.6961488
- **matchms** (Underlying data IO and validators for the .msp format handled by MSMetaEnhancer)

## Examples

```
from MSMetaEnhancer import Application; from MSMetaEnhancer.libs.converters.compute import RDKit; app = Application(); app.load_data('annotated.msp', file_format='msp'); asyncio.run(app.annotate_spectra(['RDKit'], [('smiles', 'molecular_weight', 'RDKit')])); app.save_data('annotated_spectra_with_mw.msp', file_format='msp')
```

## Evaluation signals

- Output .msp file exists and parses as valid .msp format
- Every output record that had SMILES in the input contains a molecular_weight field
- Computed molecular weights match independently recomputed RDKit values for a sample of SMILES strings
- Expert review confirms chemical plausibility of computed weights for sampled compounds
- Records missing SMILES are left without a molecular_weight field (empty dict returned), with issues captured in the optional detailed log file

## Limitations

- Conversion only works for records that already contain SMILES; records without SMILES yield no molecular weight
- Caveats in annotation (e.g. failed parsing, missing metadata) should be recorded in the optional detailed log file; attribute validation tracking was not fully implemented (open issue #68)
- Computed identifiers are an alternative to external queries, so no cross-check against a database value is performed within this step

## Evidence

- [abstract] Computed identifiers supported alongside web queries: "instead of querying external databases, computing the identifiers is also supported (e.g.molecular weight from SMILES)"
- [abstract] RDKit molecular weight computation: "we employed computation of molecular weight from SMILES using RDKit (Landrum et al., 2022)"
- [readme] README usage: register RDKit compute converter: "from MSMetaEnhancer.libs.converters.compute import RDKit
from MSMetaEnhancer.libs.utils.ConverterBuilder import ConverterBuilder

ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit])"
- [readme] README load/export pattern: [evidence span withheld — anchor status: external_doc]
- [abstract] Validation of metadata: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [abstract] Iterative per-record annotation: "The tool processes the spectral library by iteratively executing all steps for each entry until no new metadata is found."

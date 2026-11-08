---
name: python-converter-class-implementation
description: Use when a .msp mass spectral library record already carries an identifier
  (e.g. SMILES) from which a missing attribute (e.g. molecular weight) can be derived
  locally, without any external database query.
when_to_use_negative:
- The target attribute requires data from an external chemical database (CIR, CTS,
  PubChem, IDSM, BridgeDb) rather than local computation — use a Web converter instead
- Records lack the source attribute (e.g. no SMILES present), so nothing can be computed
  locally
- The attribute's form needs remote validation rather than the local matchms validators
  employed by MSMetaEnhancer
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: MSMetaEnhancer
  role: Provides the ComputeConverter base class, Job system, ConverterBuilder registration,
    and matchms-based .msp load/save used to integrate and run the new conversion
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: RDKit
  role: Parses SMILES strings locally (MolFromSmiles) and computes molecular weight
    from the resulting molecule object
  repo: https://doi.org/10.5281/zenodo.6961488
- name: matchms
  role: Used by MSMetaEnhancer for .msp data IO and for validating that obtained metadata
    have the correct form
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:04b54cc0838280d942adec90e6b98d556e3f890318b353215412879ab2f169da
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# python-converter-class-implementation

## Summary

Implement a new converter class in MSMetaEnhancer that computes metadata attributes locally (e.g. molecular weight from SMILES via RDKit) instead of querying external services, following the package's ComputeConverter pattern and Job/annotation pipeline.

## When to use

Use when a .msp mass spectral library record already carries an identifier (e.g. SMILES) from which a missing attribute (e.g. molecular weight) can be derived locally, without any external database query.

## When NOT to use

- The target attribute requires data from an external chemical database (CIR, CTS, PubChem, IDSM, BridgeDb) rather than local computation — use a Web converter instead
- Records lack the source attribute (e.g. no SMILES present), so nothing can be computed locally
- The attribute's form needs remote validation rather than the local matchms validators employed by MSMetaEnhancer

## Inputs

- Annotated .msp mass spectral library file (matchms-format) with SMILES metadata on records
- ComputeConverter subclass source code specifying conversions tuples ('source', 'target', 'conversion_method')

## Outputs

- annotated_spectra_with_mw.msp containing a molecular_weight metadata field per annotated record
- Optional detailed log file recording annotation issues

## How to apply

Load the annotated .msp file with MSMetaEnhancer's load_spectra (matchms-based .msp IO), then write a ComputeConverter subclass whose conversions list maps ('smiles', 'molecular_weight', 'conversion_method') and call create_top_level_conversion_methods(conversions, asynch=False) so the top-level method (e.g. smiles_to_molecular_weight) is auto-generated, mirroring how web converters generate methods like compound_name_to_inchi(). In the conversion method, parse the SMILES locally with RDKit's MolFromSmiles and compute the molecular weight from the molecule object; return {'molecular_weight': value} so the Job system stores it on the record, returning an empty dict when SMILES is missing. Register the converter (e.g. via ConverterBuilder.register), run the annotation job over all records having SMILES, and save the output as annotated_spectra_with_mw.msp with save_spectra(file_format='msp'). Validate that the output parses as .msp, contains the molecular_weight field, and that computed weights match independently computed RDKit values for a sample of SMILES strings.

## Related tools

- **MSMetaEnhancer** (Provides the ComputeConverter base class, Job system, ConverterBuilder registration, and matchms-based .msp load/save used to integrate and run the new conversion) — https://github.com/RECETOX/MSMetaEnhancer
- **RDKit** (Parses SMILES strings locally (MolFromSmiles) and computes molecular weight from the resulting molecule object) — https://doi.org/10.5281/zenodo.6961488
- **matchms** (Used by MSMetaEnhancer for .msp data IO and for validating that obtained metadata have the correct form)

## Examples

```
from MSMetaEnhancer import Application; from MSMetaEnhancer.libs.converters.compute import RDKit; from MSMetaEnhancer.libs.utils.ConverterBuilder import ConverterBuilder; ConverterBuilder.register([RDKit]); app = Application(); app.load_data('annotated.msp', file_format='msp'); asyncio.run(app.annotate_spectra(['RDKit'], [('smiles', 'molecular_weight', 'RDKit')])); app.save_data('annotated_spectra_with_mw.msp', file_format='msp')
```

## Evaluation signals

- Output .msp file exists and parses correctly as .msp format via matchms-based IO
- Every record with SMILES carries a molecular_weight field in the output .msp
- Computed molecular weights match RDKit values recomputed independently for a sample of SMILES strings
- Records lacking SMILES are skipped without errors (conversion returns an empty dict, Job system stores nothing)
- Expert review confirms chemical plausibility of computed weights relative to annotated formulas

## Limitations

- Compute converters only derive attributes from existing metadata; records missing the source attribute (e.g. SMILES) yield no new fields
- Obtained metadata must pass matchms validators to ensure correct form; attribute validation tracking in the log was not yet implemented (issue #68)
- The converter returns a single computed value per attribute; no remote cross-checking against databases like PubChem or ChemIDplus occurs in this local path

## Evidence

- [abstract] instead of querying external databases, computing the identifiers is also supported (e.g.molecular weight from SMILES): "instead of querying external databases, computing the identifiers is also supported (e.g.molecular weight from SMILES)"
- [abstract] we employed computation of molecular weight from SMILES using RDKit: "we employed computation of molecular weight from SMILES using RDKit (Landrum et al., 2022)"
- [abstract] the obtained metadata are validated to ensure their correct form: "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [methods] Converters automatically generate methods like compound_name_to_inchi(): "Converters automatically generate methods like `compound_name_to_inchi()`"
- [readme] from MSMetaEnhancer.libs.converters.compute import RDKit: "from MSMetaEnhancer.libs.converters.compute import RDKit
from MSMetaEnhancer.libs.utils.ConverterBuilder import ConverterBuilder

ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit])"

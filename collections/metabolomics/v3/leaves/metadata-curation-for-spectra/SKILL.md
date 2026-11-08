---
name: metadata-curation-for-spectra
description: Use when a .msp mass spectral library file has records missing metadata
  such as SMILES, InChI, InChIKey, formula, or CAS number, and those identifiers must
  be fetched or derived before downstream annotation.
when_to_use_negative:
- The spectral records already contain complete, correctly formed metadata (SMILES,
  InChI, CAS number) and no post-processing enrichment is needed.
- You need offline annotation with no network access — the web converters (CIR, CTS,
  IDSM, PubChem, BridgeDb) query external services over HTTP, so only computed converters
  like RDKit would work.
- The input is not a mass spectral library in .msp (or supported) format but an unrelated
  feature table or raw instrument file.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0605
tools:
- name: MSMetaEnhancer
  role: core Python package that loads the .msp library, orchestrates asynchronous
    annotation jobs against services, validates metadata, and saves the enriched file
  repo: github.com/RECETOX/MSMetaEnhancer
- name: matchms
  role: handles data IO of the .msp format and provides validators used to check the
    correct form of obtained metadata
- name: CIR
  role: Chemical Identifier Resolver web service used for identifier conversion, e.g.
    inchikey to inchi
  repo: mcs07/CIRpy
- name: CTS
  role: chemical translation service used for conversions such as inchikey to iupac_name
- name: IDSM
  role: Integrated Database for Small Molecules used for conversions such as inchi
    to canonical_smiles and inchi to formula
- name: RDKit
  role: computed converter that derives identifiers locally, e.g. molecular weight
    from SMILES, without querying external databases
  repo: 10.5281/zenodo.6961488
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:e42eb8a0d396a5acea71d37e73fbad07c07db9519e12a0789e24f5088018adfe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIR
  - CTS
  - IDSM
  - MSMetaEnhancer
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metadata-curation-for-spectra

## Summary

Annotate mass spectral library records in .msp files with missing metadata (InChI, IUPAC name, canonical SMILES, formula, CAS number) by querying external identifier-conversion services (CIR, CTS, IDSM, PubChem, BridgeDb) through MSMetaEnhancer's asynchronous workflow.

## When to use

Use when a .msp mass spectral library file has records missing metadata such as SMILES, InChI, InChIKey, formula, or CAS number, and those identifiers must be fetched or derived before downstream annotation.

## When NOT to use

- The spectral records already contain complete, correctly formed metadata (SMILES, InChI, CAS number) and no post-processing enrichment is needed.
- You need offline annotation with no network access — the web converters (CIR, CTS, IDSM, PubChem, BridgeDb) query external services over HTTP, so only computed converters like RDKit would work.
- The input is not a mass spectral library in .msp (or supported) format but an unrelated feature table or raw instrument file.

## Inputs

- mass spectral library file in open text-based .msp format (e.g. input_spectra_file.msp)
- list of supported services (e.g. ['CIR', 'CTS', 'IDSM', 'PubChem', 'BridgeDb', 'RDKit'])
- list of annotation jobs as (source attribute, target attribute, service) tuples

## Outputs

- enriched .msp spectral library file (e.g. annotated_spectra_file.msp) with new metadata fields (inchi, iupac_name, canonical_smiles, formula) populated per record
- optional detailed log file recording annotation issues

## How to apply

Instantiate MSMetaEnhancer's Application object and load the spectral library with load_data (or load_spectra in the paper's version) specifying file_format='msp'. Register converters via ConverterBuilder (e.g. CTS, CIR, IDSM, PubChem, BridgeDb for web lookups, RDKit for computed identifiers like molecular weight from SMILES), optionally run curate_metadata() first (e.g. to fix CAS numbers), then specify a services list and a jobs list of (source_attribute, target_attribute, service) tuples such as ('inchikey', 'inchi', 'CIR') or ('inchi', 'canonical_smiles', 'IDSM'). Execute asyncio.run(app.annotate_spectra(services, jobs)); the asynchronous implementation with per-compound caching and throttling of rate-limited services provides optimal fetching speed, and the tool iteratively executes all steps per entry until no new metadata is found. Finally export the enriched library with save_data/save_spectra to a new .msp file, optionally writing a detailed log file of annotation issues.

## Related tools

- **MSMetaEnhancer** (core Python package that loads the .msp library, orchestrates asynchronous annotation jobs against services, validates metadata, and saves the enriched file) — github.com/RECETOX/MSMetaEnhancer
- **matchms** (handles data IO of the .msp format and provides validators used to check the correct form of obtained metadata)
- **CIR** (Chemical Identifier Resolver web service used for identifier conversion, e.g. inchikey to inchi) — mcs07/CIRpy
- **CTS** (chemical translation service used for conversions such as inchikey to iupac_name)
- **IDSM** (Integrated Database for Small Molecules used for conversions such as inchi to canonical_smiles and inchi to formula)
- **RDKit** (computed converter that derives identifiers locally, e.g. molecular weight from SMILES, without querying external databases) — 10.5281/zenodo.6961488

## Examples

```
import asyncio; from MSMetaEnhancer import Application; app = Application(); app.load_spectra('input_spectra_file.msp', file_format='msp'); asyncio.run(app.annotate_spectra(['CIR','CTS','IDSM'], [('inchikey','inchi','CIR'),('inchikey','iupac_name','CTS'),('inchi','canonical_smiles','IDSM'),('inchi','formula','IDSM')])); app.save_spectra('annotated_spectra_file.msp', file_format='msp')
```

## Evaluation signals

- The output .msp file contains all input records, with newly derived inchi, iupac_name, canonical_smiles, and formula metadata fields populated per compound.
- Obtained metadata pass matchms validators, ensuring their correct form.
- Annotation proceeds iteratively until no new metadata can be found for any entry (fixpoint reached).
- Annotation issues (e.g. failed service queries) are recorded in the optional detailed log file rather than silently dropped.
- Results are cached per compound so repeated identifiers do not trigger duplicate service queries.

## Limitations

- Metadata such as chemical formula, SMILES, InChI, and spectrum origin are not always present in mass spectral library records and require this post-processing step.
- CIR behavior for multiple hits is not fully specified — an open issue proposes taking only the first result when there are multiple hits in CIR conversions (issue #69).
- PubChem SMILES metadata type handling is incomplete — an open issue proposes supporting ISOMERIC_SMILES and CANONICAL_SMILES instead of generic SMILES (issue #67).
- Attribute validation tracking in the log is not yet implemented (issue #68).
- Rate-limited external services require throttling; availability of CIR, CTS, IDSM, PubChem, and BridgeDb endpoints is a runtime dependency.

## Evidence

- [abstract] It annotates given mass spectra records in the library file by adding missing metadata such as SMILES, In ChI, and CAS numbers to the individual entries.: "It annotates given mass spectra records in the library file by adding missing metadata such as SMILES, In ChI, and CAS numbers to the individual entries."
- [abstract] this task is suitable for the asynchronous approach, making the tool computationally efficient. Additionally, results containing all metadata related to a compound are cached: "this task is suitable for the asynchronous approach, making the tool computationally efficient. Additionally, results containing all metadata related to a compound are cached"
- [abstract] jobs = [('inchikey', 'inchi', 'CIR'), ('inchikey', 'iupac_name', 'CTS'), ('inchi', 'canonical_smiles', 'IDSM'), ('inchi', 'formula', 'IDSM')]: "jobs = [('inchikey', 'inchi', 'CIR'), ('inchikey', 'iupac_name', 'CTS'), ('inchi', 'canonical_smiles', 'IDSM'), ('inchi', 'formula', 'IDSM')]"
- [abstract] the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task): "the obtained metadata are validated to ensure their correct form (currently, matchms validators are employed for this task)"
- [readme] The app uses asynchronous implementation of annotation process allowing for optimal fetching speed.: "The app uses asynchronous implementation of annotation process allowing for optimal fetching speed."
- [readme] ConverterBuilder.register([CTS, CIR, IDSM, PubChem, BridgeDb, RDKit]): [evidence span withheld — anchor status: external_doc]

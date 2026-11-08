---
name: strain-identifier-mapping
description: Use when GCF–spectrum link scoring requires strain co-occurrence vectors
  but genome/molecule objects carry heterogeneous identifiers (antiSMASH BGC ids,
  GNPS spectrum ids, mzXML filenames) that must first be reconciled to a shared strain
  namespace.
when_to_use_negative:
- Spectrum-to-strain mappings are keyed by mzXML filename and are not first passed
  through extract_mappings_ms_filename_spectrum_id, so filename-to-spectrum-id translation
  is still pending.
- The analysis uses a link-scoring method that does not depend on strain co-occurrence
  (e.g. non-strain-based scoring), in which case strain reconciliation adds nothing.
- No strain_mapping.json is available for the data set and no manual mapping can be
  supplied, since NPLinker requires this file to map strain to genomics and metabolomics
  data.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3361
tools:
- name: NPLinker
  role: Framework whose load_config/load_data pipeline reads strain_mapping.json to
    associate each BGC and spectrum identifier with its strain before link scoring
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of the molecular networking data whose spectrum identifiers must be
    mapped to strains
- name: AntiSMASH
  role: Source of the BGC regions whose identifiers must be mapped to strains
- name: BigScape
  role: Provides the GCF clustering whose member BGCs inherit strain labels from the
    mapping
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:5b8dec17e2a46eec90c2503b1ead023533751c4046a7ea288b026b519bf3608e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - AntiSMASH
  - BigScape
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# strain-identifier-mapping

## Summary

Maps BGC and spectrum identifiers to their source microbial strain using NPLinker's strain_mapping.json file, building the strain co-occurrence vectors for GCFs and molecular families required by strain-correlation scoring (e.g. Metcalf).

## When to use

Use when GCF–spectrum link scoring requires strain co-occurrence vectors but genome/molecule objects carry heterogeneous identifiers (antiSMASH BGC ids, GNPS spectrum ids, mzXML filenames) that must first be reconciled to a shared strain namespace.

## When NOT to use

- Spectrum-to-strain mappings are keyed by mzXML filename and are not first passed through extract_mappings_ms_filename_spectrum_id, so filename-to-spectrum-id translation is still pending.
- The analysis uses a link-scoring method that does not depend on strain co-occurrence (e.g. non-strain-based scoring), in which case strain reconciliation adds nothing.
- No strain_mapping.json is available for the data set and no manual mapping can be supplied, since NPLinker requires this file to map strain to genomics and metabolomics data.

## Inputs

- strain_mapping.json (NPLinker-required strain mappings file)
- GNPS molecular networking data (spectrum ids and mzXML filenames)
- AntiSMASH BGC data in the antismash subdirectory
- BiG-SCAPE GCF clustering output
- NPLinker config file (nplinker.toml)

## Outputs

- Strain-labelled BGC and spectrum objects inside the loaded NPLinker session
- Strain co-occurrence vectors for GCFs and molecular families consumable by metcalf scoring

## How to apply

After loading the NPLinker working directory (config via load_config, data via npl.load_data()) so the antiSMASH BGC regions, BiG-SCAPE GCF clustering and GNPS molecular networking files are in memory, read the strain_mappings.json file that NPLinker requires to map the strain to genomics and metabolomics data. Associate each BGC identifier and each spectrum identifier with its strain, so that every GCF and molecular family inherits a binary strain-presence vector. If spectra are keyed by mzXML filenames rather than spectrum ids, apply nplinker.metabolomics.utils.extract_mappings_ms_filename_spectrum_id to translate filename-level mappings to spectrum ids. The resulting per-strain co-occurrence vectors are the direct input to npl.get_links(..., 'metcalf'), whose raw and hypergeometric-standardised strain correlation scores depend on these vectors being correctly aligned across the genomics and metabolomics sides.

## Related tools

- **NPLinker** (Framework whose load_config/load_data pipeline reads strain_mapping.json to associate each BGC and spectrum identifier with its strain before link scoring) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of the molecular networking data whose spectrum identifiers must be mapped to strains)
- **AntiSMASH** (Source of the BGC regions whose identifiers must be mapped to strains)
- **BigScape** (Provides the GCF clustering whose member BGCs inherit strain labels from the mapping)

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data()  # strain_mapping.json read during load to map BGCs/spectra to strains
```

## Evaluation signals

- Every GCF and molecular family used in subsequent npl.get_links calls has a non-empty strain set derived from strain_mapping.json.
- Filename-keyed metabolomics mappings have been converted so that spectrum ids, not mzXML filenames, carry strain associations.
- Downstream metcalf raw scores are reproducible: recomputing get_links on all GCFs yields identical per-link scores for the same mapping file.
- For the Crüsemann data set, group means after scoring should match the reference values (all links 83.514 raw / -0.006 standardised; validated links 14.667 raw / 3.672 standardised), which fails if strain mappings are misaligned.
- NPLinker loads without errors complaining about the missing or malformed strain_mapping.json.

## Limitations

- The strain_mappings.json file is user-supplied; NPLinker requires it but does not generate strain labels automatically, so identifier inconsistencies across platforms are the curator's responsibility.
- When spectra are referenced by mzXML filename rather than spectrum id, an extra extraction step (extract_mappings_ms_filename_spectrum_id) is needed, and skipping it silently misassigns strains.

## Evidence

- [methods] The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data.: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
- [methods] you may need the function extract_mappings_ms_filename_spectrum_id to extract the mappings from mzXML files to the spectrum ids: "you may need the function [extract_mappings_ms_filename_spectrum_id][nplinker.metabolomics.utils.extract_mappings_ms_filename_spectrum_id] to extract the mappings from mzXML files to the spectrum ids"
- [methods] NPLinker requires AntiSMASH BGC data as input, which are organized in the `antismash` subdirectory of the working directory.: "NPLinker requires AntiSMASH BGC data as input, which are organized in the `antismash` subdirectory of the working directory."
- [other] Read strain_mappings.json to associate each BGC and spectrum identifier with its strain, building the strain co-occurrence vectors for GCFs and molecular families.: [evidence span withheld — anchor status: not_found]

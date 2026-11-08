---
name: compound-record-grouping-by-instrument-class
description: Use when handling multiple ESI-MS2 MassBank records of the same compound
  measured at different collision energies and you need to partition records into
  per-compound, per-instrument-class groups before spectrum merging.
when_to_use_negative:
- Records are EI-MS or FAB-MS data rather than ESI-MS2 measured under multiple CID
  collision energies
- Each compound already has a single merged mass spectrum, so no per-energy grouping
  is needed
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassBank/MassBank3
  role: Maintains the MassBank spectral reference library web app and REST API from
    which ESI-MS2 record files can be obtained and served
  repo: https://github.com/MassBank/MassBank3
- name: MassBank/MassBank-cli-tools
  role: MassBank Java command line tools, including the validator, for handling record
    files
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:60c4c56a17d3be375e6cdbcfe4e1d2d7de650cb4eff014dafa15818fb0526ada
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank/MassBank-cli-tools
  - MassBank/MassBank3
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# compound-record-grouping-by-instrument-class

## Summary

Groups parsed MassBank ESI-MS2 record files by compound identity and instrument class (QqQ vs QqTOF) so that all CID collision-energy scans (10-50 V) of one compound fall into one group, enabling construction of artificially merged reference spectra per compound and instrument class.

## When to use

Use when handling multiple ESI-MS2 MassBank records of the same compound measured at different collision energies and you need to partition records into per-compound, per-instrument-class groups before spectrum merging.

## When NOT to use

- Records are EI-MS or FAB-MS data rather than ESI-MS2 measured under multiple CID collision energies
- Each compound already has a single merged mass spectrum, so no per-energy grouping is needed

## Inputs

- MassBank ESI-MS2 record files (.txt, MassBank record format) with peak lists, accession IDs, precursor/compound identifiers, and collision-energy annotations
- Instrument class metadata distinguishing QqQ from QqTOF records

## Outputs

- Grouped record sets keyed by compound identity and instrument class (QqQ vs QqTOF)
- Merged_spectra.csv containing one artificial merged spectrum per compound per instrument class ([Merged QqQ] and [Merged QqTOF] target sets, 856 entries each)

## How to apply

Parse the MassBank ESI-MS2 record files (in MassBank record format with accession ID, peaks, and collision-energy annotations), extracting peak lists, precursor/compound identifiers, and the collision-energy annotation for each record. Group the parsed records by compound identity and by instrument class (QqQ vs QqTOF) so that all MS2 scans of one compound measured at different CID energies (e.g. 10-50 V) fall into the same group. Each group is then superimposed by merging peaks at matching m/z and summing/integrating intensities into one artificial merged spectrum, yielding one merged record per compound for each instrument class ([Merged QqQ] and [Merged QqTOF] target sets of 856 entries each). The rationale is that merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions improved identification precision by 21-23% at a similarity score of 0.6, where similarity is a weighted cosine correlation with exponents optimized to ESI-MS2 data.

## Related tools

- **MassBank/MassBank3** (Maintains the MassBank spectral reference library web app and REST API from which ESI-MS2 record files can be obtained and served) — https://github.com/MassBank/MassBank3
- **MassBank/MassBank-cli-tools** (MassBank Java command line tools, including the validator, for handling record files) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
cd MassBank3/compose && cp env.dist .env && docker compose up -d
```

## Evaluation signals

- Output merged_spectra.csv contains exactly 856 merged entries per target set ([Merged QqQ] and [Merged QqTOF] each)
- Every grouped set contains records of a single compound identity and a single instrument class (QqQ or QqTOF), spanning the 10-50 V collision-energy range
- Merging preserves peak structure such that downstream weighted cosine correlation search reproduces the reported 21-23% precision improvement at a similarity score of 0.6
- Each merged record has one entry per compound per instrument class, not one per collision energy

## Limitations

- Merged-spectrum precision gains (21-23% at similarity score 0.6) were reported for ESI-MS2 data; the grouping rationale does not transfer to EI-MS or FAB-MS records
- Only two instrument classes (QqQ and QqTOF) form the [Merged QqQ] and [Merged QqTOF] target sets; records from other instrument types are outside these sets
- MassBank-web is deprecated; MassBank3 is the current web app and MassBank-cli-tools the current command-line/validator tooling

## Evidence

- [abstract] Merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions improved the precision of compound identification by 21-23% at a similarity score of 0.6.: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral search similarity scores are calculated by a weighted cosine correlation with weighting exponents on peak intensity and m/z optimized to ESI-MS2 data.: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [readme] MassBank3 is the spectral reference library's next generation software product.: [evidence span withheld — anchor status: not_found]
- [readme] The MassBank files need to be in the specified MassBank format.: [evidence span withheld — anchor status: not_found]
- [intro] The validator and other command line tools are maintained in MassBank-cli-tools.: "The validator and other command line tools are maintained in https://github.com/MassBank/MassBank-cli-tools"

---
name: bgc-spectrum-pairing
description: 'Use when building an evaluation set of known BGC–metabolite–MS2 spectrum
  links: MIBiG records contain no MS2 data and GNPS spectra lack BGC provenance, so
  pairing must be done through shared compound structural identifiers.'
when_to_use_negative:
- Input is raw metabolomics (mzML/mzXML) or GNPS molecular networking output rather
  than GNPS library spectrum annotations — this skill links library entries to BGCs,
  it does not score links in new datasets.
- Compounds lacking InChIKeys or with only partially specified structures, since the
  join depends entirely on first-block InChIKey equality.
- Cases where stereochemistry of the metabolite matters for the analysis — the first-block
  truncation deliberately discards chirality information.
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: NPLinker
  role: Framework whose paired-genomics/metabolomics data model and MIBiG download
    support the BGC–spectrum pairing used for evaluation
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of library spectrum annotations (InChIKeys and compound names) joined
    against MIBiG records
- name: Python
  role: Implements the InChIKey truncation, matching, and CSV export steps (NPLinker
    requires Python >=3.11, installable via pip or conda)
  repo: https://github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:95cc9e896e716d275ba6eb83da6a506af4c89ca5a06ccd1fb4d8cb900c7ebb74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bgc-spectrum-pairing

## Summary

Constructs a gold-standard set of known BGC–spectrum pairs by joining MIBiG 2.0 biosynthetic gene cluster metadata with GNPS library spectra via first-block InChIKey matching, producing the 2966 validated pairs used to evaluate link-scoring in NPLinker.

## When to use

Use when building an evaluation set of known BGC–metabolite–MS2 spectrum links: MIBiG records contain no MS2 data and GNPS spectra lack BGC provenance, so pairing must be done through shared compound structural identifiers.

## When NOT to use

- Input is raw metabolomics (mzML/mzXML) or GNPS molecular networking output rather than GNPS library spectrum annotations — this skill links library entries to BGCs, it does not score links in new datasets.
- Compounds lacking InChIKeys or with only partially specified structures, since the join depends entirely on first-block InChIKey equality.
- Cases where stereochemistry of the metabolite matters for the analysis — the first-block truncation deliberately discards chirality information.

## Inputs

- MIBiG 2.0 metadata records (BGC identifiers, compound structural annotations, InChIKeys)
- GNPS library spectrum annotations (spectrum identifiers, compound names, InChIKeys)

## Outputs

- bgc_spectrum_pairs.csv with columns: BGC identifier, spectrum identifier, InChIKey, metabolite name (2966 BGC–spectrum pairs)

## How to apply

Parse MIBiG 2.0 metadata to extract each BGC's compound structural annotation and its InChIKey, and parse GNPS library spectrum annotations to obtain InChIKeys (and compound names) for each library spectrum. Normalize all InChIKeys by truncating to the first skeleton block, which collapses stereochemistry/chirality variants to the same key — a necessary step because MS2 spectra cannot distinguish stereoisomers. Match MIBiG BGCs to GNPS library spectra sharing the same first InChIKey block and emit the pair table (BGC identifier, spectrum identifier, InChIKey, metabolite name) to bgc_spectrum_pairs.csv. Rationale: pairing on full InChIKeys would wrongly split pairs that MS2 cannot discriminate, and the structural-annotation join is the only available bridge since MIBiG itself contains no spectra.

## Related tools

- **NPLinker** (Framework whose paired-genomics/metabolomics data model and MIBiG download support the BGC–spectrum pairing used for evaluation) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of library spectrum annotations (InChIKeys and compound names) joined against MIBiG records)
- **Python** (Implements the InChIKey truncation, matching, and CSV export steps (NPLinker requires Python >=3.11, installable via pip or conda)) — https://github.com/NPLinker/nplinker

## Examples

```
python scripts/build_bgc_spectrum_pairs.py --mibig mibig/ --gnps gnps_library.tsv --out bgc_spectrum_pairs.csv  # expect 2966 rows
```

## Evaluation signals

- The emitted bgc_spectrum_pairs.csv must contain exactly 2966 BGC–spectrum pairs, matching the number reported in the paper.
- Row-level match: pair table rows must equal the corresponding rows of the deposited S1 Data file.
- Every emitted pair must have a non-empty BGC identifier, spectrum identifier, first-block InChIKey, and metabolite name.
- All InChIKeys in the pair table must be truncated to the first skeleton block (13 characters), with stereochemistry blocks removed.
- No pair should join records whose full InChIKeys share only a stereoisomer distinction — verify each pair's first blocks are identical.

## Limitations

- Pairing relies on InChIKeys present in both MIBiG and GNPS annotations; compounds without InChIKeys or with non-canonical annotations are silently missed.
- First-block InChIKey matching cannot distinguish stereoisomers, so pairs are compound-skeleton-level rather than stereochemically exact.
- The pair count is tied to MIBiG 2.0 and the GNPS library snapshot; other MIBiG or GNPS versions will not reproduce the 2966 pairs.

## Evidence

- [other] Pairing is performed by matching structural annotations using only the first part of the InChIKey of each metabolite (avoiding chirality-based distinctions absent from MS2 spectra), which yields 2966 BGC-spectrum pairs: [evidence span withheld — anchor status: not_found]
- [methods] MIBiG directory handling: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"
- [methods] Python version requirement: "Python version ≥3.11"
- [methods] NPLinker data integration: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data"

---
name: inchikey-skeleton-matching
description: 'Use when building a ground-truth set of known BGC–spectrum links: you
  have MIBiG BGC records with structural annotations and GNPS library spectra with
  InChIKey annotations, but no direct spectral link, and need a chirality-insensitive
  structural join.'
when_to_use_negative:
- The goal is to distinguish stereoisomers or chirality variants — first-block skeleton
  matching deliberately collapses them, so a full InChIKey match is required instead
- The metabolomics side lacks InChIKey annotations (e.g., unannotated GNPS molecular
  networking nodes) — no structural join key exists and the pairing cannot be performed
- Input spectra are experimental MS2 runs rather than GNPS library spectra with curated
  compound annotations
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Framework in which the paired BGC–spectrum ground truth is used to evaluate
    genomic–metabolomic link scoring; auto-downloads the MIBiG metadata
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Scripting language used to parse metadata, truncate InChIKeys, and emit the
    pair table (requires Python ≥3.11)
- name: GNPS
  role: Source of the library spectrum annotations (compound InChIKeys and names)
    that are matched against MIBiG
- name: MIBiG
  role: Curated BGC database whose compound structural annotations supply one side
    of each pair
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:446a7fd93fd75ba06377f1e754f96ced50ce6a839efab322ebf29576c846c97e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - MIBiG
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# InChIKey skeleton matching for BGC–spectrum pairing

## Summary

Links MIBiG biosynthetic gene clusters (BGCs) to GNPS library MS2 spectra by matching metabolite InChIKeys truncated to their first (skeleton) block, since MS2 spectra cannot resolve stereochemistry. This reconstructs the known BGC–spectrum pair set used to evaluate NPLinker's link-scoring methods.

## When to use

Use when building a ground-truth set of known BGC–spectrum links: you have MIBiG BGC records with structural annotations and GNPS library spectra with InChIKey annotations, but no direct spectral link, and need a chirality-insensitive structural join.

## When NOT to use

- The goal is to distinguish stereoisomers or chirality variants — first-block skeleton matching deliberately collapses them, so a full InChIKey match is required instead
- The metabolomics side lacks InChIKey annotations (e.g., unannotated GNPS molecular networking nodes) — no structural join key exists and the pairing cannot be performed
- Input spectra are experimental MS2 runs rather than GNPS library spectra with curated compound annotations

## Inputs

- MIBiG 2.0 metadata records with compound structural annotations and InChIKeys per BGC
- GNPS library spectrum annotation table with compound InChIKeys and names per spectrum
- GNPS molecular networking data (GNPS1 or GNPS2 workflow export) as consumed by NPLinker

## Outputs

- bgc_spectrum_pairs.csv with columns: BGC identifier, spectrum identifier, InChIKey, and metabolite name (2966 BGC–spectrum pairs for MIBiG 2.0)
- Known BGC–spectrum pair set used as ground truth for evaluating NPLinker link-scoring functions

## How to apply

Parse the MIBiG 2.0 metadata records to extract each BGC's compound structural annotation and its InChIKey (NPLinker downloads MIBiG metadata automatically into the `mibig` directory). Parse the GNPS library spectrum annotations to obtain the compound InChIKeys and names for each library spectrum. Normalize every InChIKey by truncating it to the first skeleton block so that stereochemistry/chirality variants collapse to the same key — this is essential because MS2 spectra carry no chirality information. Match MIBiG BGCs to GNPS library spectra sharing the same first InChIKey block and write the pair table (BGC identifier, spectrum identifier, InChIKey, metabolite name) to bgc_spectrum_pairs.csv. This pairing underpins the NPLinker evaluation framework that scores links between genomic and metabolomic data with functions such as Metcalf and the Input-Output Kernel Regression score.

## Related tools

- **NPLinker** (Framework in which the paired BGC–spectrum ground truth is used to evaluate genomic–metabolomic link scoring; auto-downloads the MIBiG metadata) — https://github.com/NPLinker/nplinker
- **Python** (Scripting language used to parse metadata, truncate InChIKeys, and emit the pair table (requires Python ≥3.11))
- **GNPS** (Source of the library spectrum annotations (compound InChIKeys and names) that are matched against MIBiG)
- **MIBiG** (Curated BGC database whose compound structural annotations supply one side of each pair)

## Examples

```
bgc_key = bgc_inchikey.split('-')[0]; pairs = [(b.id, s.id, bgc_key, s.name) for b in mibig_bgcs for s in gnps_library if s.inchikey.split('-')[0] == bgc_key]
```

## Evaluation signals

- The emitted pair count must equal the 2966 BGC–spectrum pairs reported in the paper
- Rows of bgc_spectrum_pairs.csv must match the corresponding rows of the deposited S1 Data file
- Every emitted pair carries an associated metabolite name alongside BGC and spectrum identifiers
- All InChIKeys are truncated to the first skeleton block before matching (no stereochemistry-based distinctions remain)
- Every matched pair shares an identical first InChIKey block between its MIBiG compound and GNPS library entry

## Limitations

- MIBiG itself contains no MS2 spectra, so pairing relies entirely on structural annotation overlap rather than spectral evidence
- Truncating to the first InChIKey block collapses chirality- and stereochemistry-based distinctions, so pairs may correspond to multiple stereoisomers that MS2 spectra cannot distinguish
- Pair count is sensitive to the MIBiG version — the 2966 figure is specific to MIBiG 2.0

## Evidence

- [methods] Pairing is performed by matching structural annotations using only the first part of the InChIKey of each metabolite: [evidence span withheld — anchor status: not_found]
- [methods] which yields 2966 BGC-spectrum pairs, each with an associated metabolite: [evidence span withheld — anchor status: not_found]
- [methods] NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data.: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
- [other] `mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"

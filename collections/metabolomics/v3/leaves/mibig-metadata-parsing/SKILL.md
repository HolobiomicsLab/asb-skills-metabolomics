---
name: mibig-metadata-parsing
description: Use when building a validated ground-truth table of known BGC-spectrum
  pairs from MIBiG structural annotations and GNPS library spectra, e.g. to benchmark
  metabologenomic link scoring methods.
when_to_use_negative:
- Input is already a BGC-spectrum pairs table (e.g. the deposited S1 Data file) —
  no re-parsing is needed
- The goal is scoring candidate GCF-spectrum links (e.g. metcalf, Rosetta, or Input-Output
  Kernel Regression scores) rather than constructing evaluation ground truth
- Full stereochemical resolution of metabolite identity is required — first-InChIKey-block
  matching deliberately collapses chirality variants because MS2 spectra cannot distinguish
  them
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2250
tools:
- name: NPLinker
  role: Framework that automatically downloads and stores MIBiG metadata for parsing,
    and integrates the resulting BGC-spectrum pairs into genomics–metabolomics link
    evaluation
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Language used to implement the metadata parsing, InChIKey truncation, and
    pair-matching script
- name: GNPS
  role: Source of library spectrum annotations (compound InChIKeys and names) that
    are paired with MIBiG BGCs
  repo: https://gnps.ucsd.edu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:2cd2d45fdaea6513369d51d77f7aeffda7585d520949ecf425fa0762ea25ee1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mibig-metadata-parsing

## Summary

Parses MIBiG 2.0 BGC metadata records to extract compound structural annotations and InChIKeys, then matches them against GNPS library spectrum InChIKeys (truncated to the first skeleton block) to build a ground-truth set of 2966 known BGC-spectrum pairs for evaluating metabologenomic link scores.

## When to use

Use when building a validated ground-truth table of known BGC-spectrum pairs from MIBiG structural annotations and GNPS library spectra, e.g. to benchmark metabologenomic link scoring methods.

## When NOT to use

- Input is already a BGC-spectrum pairs table (e.g. the deposited S1 Data file) — no re-parsing is needed
- The goal is scoring candidate GCF-spectrum links (e.g. metcalf, Rosetta, or Input-Output Kernel Regression scores) rather than constructing evaluation ground truth
- Full stereochemical resolution of metabolite identity is required — first-InChIKey-block matching deliberately collapses chirality variants because MS2 spectra cannot distinguish them

## Inputs

- MIBiG 2.0 metadata records (BGC entries with compound structural annotations and InChIKeys; auto-downloaded by NPLinker into the `mibig` directory)
- GNPS library spectrum annotations (compound InChIKeys and names for each library spectrum)

## Outputs

- bgc_spectrum_pairs.csv — table of 2966 BGC-spectrum pairs with columns: BGC identifier, spectrum identifier, InChIKey, and metabolite name

## How to apply

Parse the MIBiG 2.0 metadata records, extracting each BGC's compound structural annotation and its InChIKey, and parse the GNPS library spectrum annotations to obtain compound InChIKeys and names for each library spectrum. Normalize all InChIKeys by truncating to the first skeleton block so that stereochemistry/chirality variants collapse to the same key — this is essential because MS2 spectra cannot distinguish stereoisomers. Match MIBiG BGCs to GNPS library spectra sharing the same first InChIKey block to form BGC-spectrum pairs, and write the pair table (BGC identifier, spectrum identifier, InChIKey, and metabolite name) to bgc_spectrum_pairs.csv. In the NPLinker context the `mibig` directory containing MIBiG metadata is automatically created and downloaded by NPLinker, so the parser can consume that pre-fetched metadata rather than re-downloading MIBiG manually.

## Related tools

- **NPLinker** (Framework that automatically downloads and stores MIBiG metadata for parsing, and integrates the resulting BGC-spectrum pairs into genomics–metabolomics link evaluation) — https://github.com/NPLinker/nplinker
- **Python** (Language used to implement the metadata parsing, InChIKey truncation, and pair-matching script)
- **GNPS** (Source of library spectrum annotations (compound InChIKeys and names) that are paired with MIBiG BGCs) — https://gnps.ucsd.edu

## Evaluation signals

- The emitted pair count must equal the 2966 BGC-spectrum pairs reported in the paper
- Each row of bgc_spectrum_pairs.csv must match the corresponding rows of the deposited S1 Data file
- Every emitted pair carries all four fields: BGC identifier, spectrum identifier, InChIKey, and metabolite name
- All InChIKeys in the output are truncated to the first skeleton block, so stereochemistry/chirality variants collapse to the same key
- No emitted pair has a MIBiG first-block InChIKey that differs from its GNPS spectrum's first-block InChIKey

## Limitations

- MIBiG itself contains no MS2 spectra, so pairing relies entirely on structural annotations (InChIKey matching) rather than spectral evidence
- Truncating InChIKeys to the first block means chirality/stereochemistry distinctions are lost — a pair may link a BGC to a stereoisomer indistinguishable by MS2
- The 2966-pair validation count is tied to MIBiG 2.0 and the specific GNPS library snapshot used; other MIBiG versions will yield different counts

## Evidence

- [methods] MIBiG metadata location: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"
- [methods] GNPS input requirement: "NPLinker requires GNPS molecular networking data as input"
- [other] InChIKey skeleton-block normalization: [evidence span withheld — anchor status: not_found]
- [other] Pairing rationale: [evidence span withheld — anchor status: not_found]
- [other] Pair count and output: [evidence span withheld — anchor status: not_found]

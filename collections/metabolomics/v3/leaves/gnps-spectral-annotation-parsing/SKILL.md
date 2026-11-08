---
name: gnps-spectral-annotation-parsing
description: Use when building validated BGC-molecule links from MIBiG and GNPS library
  spectra, or when structural annotations must be matched across databases despite
  missing stereochemistry information in MS2 data.
when_to_use_negative:
- Input spectra already carry direct BGC-spectrum link annotations (e.g. from PODP
  paired datasets) — no InChIKey-based reconstruction is needed
- The task requires stereochemically resolved compound matching; first-block InChIKey
  truncation deliberately discards chirality information absent from MS2 spectra
- The goal is de novo spectral annotation or molecular networking (GNPS library search
  / FBMN), rather than constructing a ground-truth evaluation set
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: NPLinker
  role: Framework within which the BGC-spectrum pairing and downstream link scoring
    are performed; consumes the resulting known-link set for evaluation
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of library spectrum annotations (InChIKeys and compound names per spectrum);
    NPLinker accepts GNPS1 and GNPS2 molecular networking data
  repo: https://gnps.ucsd.edu
- name: MIBiG
  role: Source of BGC compound structural annotations and InChIKeys; metadata is automatically
    downloaded by NPLinker
- name: Python
  role: Implementation language for parsing, InChIKey truncation, and pair-table generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:86c7b4085d853867294c2f7aecb658df6e94dd2fb33795f227f4d7ce070ce979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - MIBiG
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gnps-spectral-annotation-parsing

## Summary

Parses GNPS library spectrum annotations to extract compound InChIKeys and names, then pairs them with MIBiG 2.0 BGC structural annotations via first-block InChIKey matching to build a gold-standard set of known BGC-spectrum links (2966 pairs in the NPLinker paper).

## When to use

Use when building validated BGC-molecule links from MIBiG and GNPS library spectra, or when structural annotations must be matched across databases despite missing stereochemistry information in MS2 data.

## When NOT to use

- Input spectra already carry direct BGC-spectrum link annotations (e.g. from PODP paired datasets) — no InChIKey-based reconstruction is needed
- The task requires stereochemically resolved compound matching; first-block InChIKey truncation deliberately discards chirality information absent from MS2 spectra
- The goal is de novo spectral annotation or molecular networking (GNPS library search / FBMN), rather than constructing a ground-truth evaluation set

## Inputs

- MIBiG 2.0 metadata records (BGC identifiers with compound structural annotations and InChIKeys)
- GNPS library spectrum annotations (spectrum identifiers with compound InChIKeys and names)

## Outputs

- bgc_spectrum_pairs.csv — pair table with BGC identifier, spectrum identifier, InChIKey, and metabolite name (~2966 pairs)
- Normalized first-block InChIKey index collapsing stereochemistry variants

## How to apply

Parse the MIBiG 2.0 metadata records to extract each BGC's compound structural annotation and InChIKey, and parse the GNPS library spectrum annotations to obtain compound InChIKeys and names for each library spectrum. Normalize all InChIKeys by truncating to the first skeleton block so that stereochemistry/chirality variants collapse to the same key — this is essential because MS2 spectra cannot distinguish chirality-based differences. Match MIBiG BGCs to GNPS library spectra sharing the same first InChIKey block to form BGC-spectrum pairs. Write the pair table (BGC identifier, spectrum identifier, InChIKey, and metabolite name) to bgc_spectrum_pairs.csv. Validate by confirming the emitted pair count equals the 2966 BGC-spectrum pairs reported in the paper and matches the deposited S1 Data rows.

## Related tools

- **NPLinker** (Framework within which the BGC-spectrum pairing and downstream link scoring are performed; consumes the resulting known-link set for evaluation) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of library spectrum annotations (InChIKeys and compound names per spectrum); NPLinker accepts GNPS1 and GNPS2 molecular networking data) — https://gnps.ucsd.edu
- **MIBiG** (Source of BGC compound structural annotations and InChIKeys; metadata is automatically downloaded by NPLinker)
- **Python** (Implementation language for parsing, InChIKey truncation, and pair-table generation)

## Examples

```
python build_bgc_spectrum_pairs.py --mibig mibig_json/ --gnps gnps_library.tsv --out bgc_spectrum_pairs.csv  # expect 2966 rows
```

## Evaluation signals

- Number of emitted BGC-spectrum pairs must equal 2966 as reported in the paper
- Pair rows must match the corresponding rows of the deposited S1 Data file
- All InChIKeys in the output must be truncated to the first skeleton block (13-character prefix before the '-' separator)
- Each emitted pair must carry a non-empty BGC identifier, spectrum identifier, InChIKey, and metabolite name in bgc_spectrum_pairs.csv
- Every matched pair must share an identical first InChIKey block between the MIBiG and GNPS records

## Limitations

- Matching on the first InChIKey block collapses stereoisomers into one key, so pairs differing only in stereochemistry are treated as the same metabolite — appropriate for MS2 but loses stereochemical specificity
- Pairing is limited to compounds present in both the MIBiG annotations and the GNPS library spectra; BGCs whose metabolites lack GNPS library spectra are unrepresented
- Validation depends on the exact MIBiG 2.0 and GNPS library snapshot; newer database versions may change the pair count

## Evidence

- [other] InChIKey first-block matching rationale: [evidence span withheld — anchor status: not_found]
- [other] pair count: [evidence span withheld — anchor status: not_found]
- [methods] MIBiG metadata in NPLinker: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"
- [methods] GNPS input requirement: "NPLinker requires GNPS molecular networking data as input. It currently accepts data from both GNPS1 (https://gnps.ucsd.edu) and GNPS2 (https://gnps2.org) workflows."

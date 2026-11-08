---
name: natural-product-structure-informatics
description: Use when you need ground-truth linkages between gene clusters and MS2
  spectra for evaluating linking algorithms, and both MIBiG BGC annotations and GNPS
  library InChIKeys are available but the two databases contain no direct cross-references.
when_to_use_negative:
- MS2 spectra or reference annotations already contain full stereochemical InChIKeys
  with a reliable link table — first-block truncation would needlessly discard chirality
  information
- The goal is scoring candidate GCF–molecular-family links (e.g. metcalf scoring via
  npl.get_links), not constructing the ground-truth pair table itself
edam_operation: http://edamontology.org/operation_0349
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: NPLinker
  role: Framework whose validated evaluation set these BGC–spectrum pairs feed; it
    automatically downloads MIBiG metadata and requires GNPS molecular networking
    data as input
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Parsing MIBiG and GNPS annotations, truncating InChIKeys to the first skeleton
    block, and writing the pair table (NPLinker itself is a python framework)
- name: GNPS
  role: Source of the library spectrum annotations whose compound InChIKeys are matched
    against MIBiG structural annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:b729d9529e807537d9b11c83c6b6e0305887b81048eebb3a1b238553cdd6630f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_0349
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# natural-product-structure-informatics

## Summary

Builds a validated set of known BGC–spectrum pairs by joining MIBiG 2.0 biosynthetic gene cluster metadata (compound names and InChIKeys) against GNPS library spectrum annotations, using first-block InChIKey matching so stereoisomers collapse onto the same chemical skeleton.

## When to use

Use when you need ground-truth linkages between gene clusters and MS2 spectra for evaluating linking algorithms, and both MIBiG BGC annotations and GNPS library InChIKeys are available but the two databases contain no direct cross-references.

## When NOT to use

- MS2 spectra or reference annotations already contain full stereochemical InChIKeys with a reliable link table — first-block truncation would needlessly discard chirality information
- The goal is scoring candidate GCF–molecular-family links (e.g. metcalf scoring via npl.get_links), not constructing the ground-truth pair table itself

## Inputs

- MIBiG 2.0 metadata records with per-BGC compound structural annotations and InChIKeys
- GNPS library spectrum annotations with compound InChIKeys and names
- GNPS molecular networking data (required NPLinker input)

## Outputs

- bgc_spectrum_pairs.csv: BGC identifier, spectrum identifier, InChIKey, and metabolite name for each of the 2966 known BGC–spectrum pairs

## How to apply

Parse MIBiG 2.0 metadata records to extract each BGC's compound structural annotation and InChIKey, and parse the GNPS library spectrum annotations to obtain InChIKeys (and names) for each library spectrum. Normalize every InChIKey by truncating to the first skeleton block — MS2 spectra do not encode stereochemistry, so chirality-based distinctions must be collapsed before matching. Join the two sides on equal first-block keys to form BGC–spectrum pairs, each carrying a metabolite name, and write the pair table (BGC identifier, spectrum identifier, InChIKey, metabolite name) to bgc_spectrum_pairs.csv. In the NPLinker context, the `mibig` directory of metadata is automatically created and downloaded by NPLinker, and GNPS molecular networking/library data is a required input. Validate by counting emitted pairs: the procedure must reproduce the 2966 BGC–spectrum pairs reported in the paper and match the deposited S1 Data file row-for-row.

## Related tools

- **NPLinker** (Framework whose validated evaluation set these BGC–spectrum pairs feed; it automatically downloads MIBiG metadata and requires GNPS molecular networking data as input) — https://github.com/NPLinker/nplinker
- **Python** (Parsing MIBiG and GNPS annotations, truncating InChIKeys to the first skeleton block, and writing the pair table (NPLinker itself is a python framework))
- **GNPS** (Source of the library spectrum annotations whose compound InChIKeys are matched against MIBiG structural annotations)

## Examples

```
skel = lambda ik: ik.split('-')[0]; pairs = [(b, s, skel(ikb), name) for b, ikb, name in mibig_records for s, iks in gnps_lib if skel(ikb) == skel(iks)]
```

## Evaluation signals

- Pair count invariant: the emitted table must contain exactly 2966 BGC–spectrum pairs
- Row-level diff against the corresponding rows of the deposited S1 Data file
- Every emitted pair must carry a non-empty metabolite name and a first-block (skeleton) InChIKey
- No pair exists where the two sides differ beyond the first InChIKey block (stereo suffixes must have been stripped before matching)
- Each BGC and spectrum identifier in the table must resolve to a record in the parsed MIBiG 2.0 metadata and GNPS library annotations respectively

## Limitations

- First-block InChIKey matching deliberately collapses stereoisomers, so pairs are structural-skeleton-level, not stereochemically exact
- MIBiG itself contains no MS2 spectra; pairs exist only insofar as GNPS library annotations cover MIBiG compounds
- The pair table is tied to MIBiG 2.0 and the GNPS library snapshot used; updating either source changes the 2966-pair count

## Evidence

- [full_text] InChIKey first-block matching rationale: [evidence span withheld — anchor status: not_found]
- [full_text] Pair count and output: [evidence span withheld — anchor status: not_found]
- [methods] MIBiG metadata provenance in NPLinker: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"
- [methods] GNPS input requirement: "NPLinker requires GNPS molecular networking data as input"

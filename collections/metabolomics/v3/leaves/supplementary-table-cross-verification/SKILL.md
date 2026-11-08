---
name: supplementary-table-cross-verification
description: Reference for validating that a curated inventory of tool- or database-specific
  lipid annotation styles is complete and correctly attributed against a paper's supplementary
  file and repository configuration files.
when_to_use_negative:
- The task is to convert actual lipid abbreviations between annotation formats — use
  LipidLynxX Converter directly instead of auditing its style list
- The software or database of interest is not among the 26 styles and no configuration
  file in lynx/configurations/rules/input fits it — the README states users should
  contact the developers rather than assume coverage
- Verification target is a non-tabular artifact (e.g. source code behavior or API
  responses) rather than a compiled list of named styles
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: LipidLynxX
  role: Source of the JSON configuration files defining the 26 predefined input styles
    that the compiled list must match
  repo: https://github.com/SysMedOs/LipidLynxX
- name: RefMet
  role: Reference converter/database whose annotation style is one of the 26 predefined
    input styles used to check list completeness
- name: Goslin
  role: Reference converter whose annotation style is one of the predefined input
    styles used in the cross-check
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:0c1557446dec71b24560855727f7e9965b7698b8c1251e3b7d0952b359a7f8b0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SysMedOs/LipidLynxX
  repo_url: https://github.com/SysMedOs/LipidLynxX
  tools:
  - Goslin
  - Lion
  - LipidID mapper
  - LipidLynxX
  - RefMet
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# supplementary-table-cross-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-verify a compiled list of lipid annotation input styles (17 identification programs plus 5 lipid-oriented databases, 26 styles total) against the authoritative source lists in the LipidLynxX JSON configuration files and the article's supplementary/reference lists, ensuring each style appears exactly once with a correct tool/database assignment.

## When to use

Reference for validating that a curated inventory of tool- or database-specific lipid annotation styles is complete and correctly attributed against a paper's supplementary file and repository configuration files.

## When NOT to use

- The task is to convert actual lipid abbreviations between annotation formats — use LipidLynxX Converter directly instead of auditing its style list
- The software or database of interest is not among the 26 styles and no configuration file in lynx/configurations/rules/input fits it — the README states users should contact the developers rather than assume coverage
- Verification target is a non-tabular artifact (e.g. source code behavior or API responses) rather than a compiled list of named styles

## Inputs

- JSON configuration files from lynx/configurations/rules/input in the SysMedOs/LipidLynxX repository
- Supplementary File 1 reference list of identification tools and lipid-oriented databases from the LipidLynxX article (DOI 10.1101/2020.04.09.033894)
- Repository README section 'Supported lipid notation styles'

## Outputs

- CSV table with columns: style name, source tool/database, and tool category (identification software vs. lipid database), one row per input style (26 rows expected)
- Discrepancy report of missing, extra, or misattributed styles relative to Supplementary File 1

## How to apply

Load the predefined input-style configuration files from the LipidLynxX repository, located under `lynx/configurations/rules/input` (with aliases in `lynx/configurations/defined_alias.json`), and parse each JSON structure to extract the style name and the identification tool or lipid database whose annotation format it represents. Compile the extracted styles into a CSV table with columns for style name, source tool/database, and category (identification software vs. lipid-oriented database). Cross-check this table against the reference list in Supplementary File 1 and the README's authoritative lists — 5 databases (HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids) and 17 programs (ALEX123 lipid calculator, Greazy, LDA 2, LipidBlast, LipidCreator, LipiDex, LipidFrag, LipidHunter, LipidMatch, LipidPro, LipidSearch, Lipostar, LIQUID, LPPtiger, MetFrag, MS-DIAL, MZmine2). The rationale is that the README and JSON configurations are the executable ground truth for what LipidLynxX's Converter actually accepts, so the compiled table must agree with both them and the supplementary list.

## Related tools

- **LipidLynxX** (Source of the JSON configuration files defining the 26 predefined input styles that the compiled list must match) — https://github.com/SysMedOs/LipidLynxX
- **RefMet** (Reference converter/database whose annotation style is one of the 26 predefined input styles used to check list completeness)
- **Goslin** (Reference converter whose annotation style is one of the predefined input styles used in the cross-check)

## Examples

```
python -c "import json,glob,csv; rows=[(f.split('/')[-1], json.load(open(f)).get('name','')) for f in glob.glob('lynx/configurations/rules/input/*.json')]; csv.writer(open('input_styles.csv','w')).writerows([('style','source')]+rows)"
```

## Evaluation signals

- Row count invariant: the compiled CSV contains exactly 26 input styles (17 program styles + 5 database styles), each appearing exactly once
- Set equality: every style name in the CSV matches a configuration file in lynx/configurations/rules/input with no extras or omissions
- Attribution check: each style's source tool/database assignment agrees with the reference list in Supplementary File 1
- Category partition: every row is classified as either identification software or lipid-oriented database, and the database subset equals {HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids}
- Program subset check: the 17 identification-program rows equal the README list (ALEX123 lipid calculator through MZmine2) with no renamed or missing entries

## Limitations

- LipidLynxX coverage is limited to mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipid styles are not covered
- The list reflects the surveyed snapshot — common abbreviations are handled separately as 'defined alias' entries in defined_alias.json, not as the 26 input styles
- No changelog was available in the repository snapshot, so the style list may drift from the current master branch; new styles are proposed via feature request issues

## Evidence

- [readme] Databases (5): - `HMDB`, `LIPID MAPS LMSD & COMP_DB`, `LipidHome`, `RefMet`, `SwissLipids`: [evidence span withheld — anchor status: external_doc]
- [readme] Programs (17): ALEX123 lipid calculator ... MS-DIAL, MZmine2: [evidence span withheld — anchor status: external_doc]
- [readme] you can test if any of the configuration files located in `lynx/configurations/rules/input` would fit to your database / program: [evidence span withheld — anchor status: external_doc]

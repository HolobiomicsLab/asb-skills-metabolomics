---
name: substrate-selectivity-evidence-code-extraction
description: Use when reproducing or auditing how the MIBiG JSON schema encodes evidence
  for A-domain/NRPS substrate selectivity — e.g. enumerating the evidence codes the
  schema permits for substrate annotations.
when_to_use_negative:
- The goal is BGC detection from sequence — use dedicated tools such as antiSMASH,
  fungiSMASH, plantiSMASH, GECCO, DeepBGC, RiPPMiner or PRISM 4 instead of schema
  inspection
- The goal is predicting A-domain substrate specificity from sequence (e.g. with SANDPUMA
  or NRPSPredictor2) rather than enumerating how the schema records evidence for such
  annotations
- The input is not the MIBiG JSON schema/data standard (e.g. raw GenBank files or
  the mibig.secondarymetabolites.org web entries alone)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0154
tools:
- name: mibig-json
  role: Repository hosting the MIBiG JSON-formatted data standard and schema definitions
    that are traversed to collect evidence codes
  repo: https://github.com/mibig-secmet
- name: Python json
  role: Used to load and traverse the JSON schema's enumerated definitions to collect
    evidence codes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:47b1f2aec4e06683bc75ac31bd129f1eaa9674547d04434f474fcdbdbc832031
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python json
  - mibig-json
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# substrate-selectivity-evidence-code-extraction

## Summary

Extracts the enumerated evidence codes in the MIBiG JSON schema that govern NRPS substrate-selectivity annotations of protein domains (notably A-domains), and records them alongside literature references in a table mirroring Table 1 of the MIBiG 3.0 paper.

## When to use

Use when reproducing or auditing how the MIBiG JSON schema encodes evidence for A-domain/NRPS substrate selectivity — e.g. enumerating the evidence codes the schema permits for substrate annotations.

## When NOT to use

- The goal is BGC detection from sequence — use dedicated tools such as antiSMASH, fungiSMASH, plantiSMASH, GECCO, DeepBGC, RiPPMiner or PRISM 4 instead of schema inspection
- The goal is predicting A-domain substrate specificity from sequence (e.g. with SANDPUMA or NRPSPredictor2) rather than enumerating how the schema records evidence for such annotations
- The input is not the MIBiG JSON schema/data standard (e.g. raw GenBank files or the mibig.secondarymetabolites.org web entries alone)

## Inputs

- mibig-secmet/mibig-json repository (git clone) containing the MIBiG JSON schema definition files
- MIBiG 3.0 paper Table 1 (reference layout and expected count of 13 evidence codes)

## Outputs

- table1_evidence_codes.csv with one row per substrate-selectivity evidence code
- validation report confirming the CSV contains exactly 13 evidence codes

## How to apply

Clone the mibig-secmet/mibig-json repository, which hosts the MIBiG JSON-formatted data standard. Load the JSON schema definition file(s) with Python's json module and traverse their enumerated definitions, collecting every evidence code that governs substrate-selectivity annotations of protein domains (in MIBiG 3.0 this concerns A-domains of NRPSs, which recognise over 500 known substrates). Write each collected code as one row to table1_evidence_codes.csv so the file mirrors the layout of Table 1 of the MIBiG 3.0 paper. The paper reports exactly 13 evidence codes added for substrate selectivity, so validate the CSV row count against that threshold and flag any discrepancy; each code should be paired with the literature references supporting A-domain substrate annotations.

## Related tools

- **mibig-json** (Repository hosting the MIBiG JSON-formatted data standard and schema definitions that are traversed to collect evidence codes) — https://github.com/mibig-secmet
- **Python json** (Used to load and traverse the JSON schema's enumerated definitions to collect evidence codes)

## Examples

```
git clone https://github.com/mibig-secmet/mibig-json && python -c "import json,glob,csv; codes=[e for f in glob.glob('mibig-json/schema*.json') for e in json.load(open(f)).get('definitions',{}) if 'evidence' in e]; w=csv.writer(open('table1_evidence_codes.csv','w')); [w.writerow([c]) for c in codes]; print(len(codes))"
```

## Evaluation signals

- table1_evidence_codes.csv contains exactly 13 evidence code rows, matching the count the MIBiG 3.0 paper reports adding for substrate selectivity
- Every collected code is an enumerated value actually present in the schema's definitions governing substrate-selectivity annotations of protein domains
- Each evidence-code row carries its associated literature reference(s) as the paper does for A-domain substrate annotations
- Column layout of the CSV mirrors Table 1 of the MIBiG 3.0 paper
- Discrepancies between the extracted count and the paper-reported count of 13 are flagged rather than silently accepted

## Limitations

- The 13-code count is specific to the MIBiG 3.0 schema version; later schema revisions may add or remove evidence codes, invalidating the expected count
- Applies only to substrate-selectivity evidence codes; other MIBiG annotations (e.g. compound structures cross-linked to NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL, or 1002 bioactivity annotations) use different schema fields
- The genbanks directory in mibig-json only holds a handful of sequence files not available from NCBI GenBank/RefSeq, so evidence extraction must target the schema/data files, not GenBank inputs

## Evidence

- [methods] Clone the mibig-json repository from github.com/mibig-secmet/mibig-json, which hosts the MIBiG JSON-formatted data standard: [evidence span withheld — anchor status: not_found]
- [results] The MIBiG JSON schema records evidence for substrate selectivity via exactly 13 evidence codes (documented in Table 1): [evidence span withheld — anchor status: not_found]
- [supplementary] The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub (https://github.com/mibig-secmet)"
- [intro] for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"

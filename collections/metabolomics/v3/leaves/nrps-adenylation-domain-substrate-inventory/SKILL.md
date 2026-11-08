---
name: nrps-adenylation-domain-substrate-inventory
description: Use when you need to quantify how comprehensively a biosynthesis-visualisation
  tool (RAIChU) covers tailoring reactions, NRPS A-domain substrates/starter units,
  KS subtypes, or extender units before relying on it to depict fully maturated natural
  product pathways.
when_to_use_negative:
- You want RAIChU to predict the regio- and stereochemistry of tailoring enzymes —
  the package cannot do this; users must specify reaction targets themselves.
- You need mature natural product structures generated directly from raw antiSMASH
  output — RAIChU generates scaffolds but not fully maturated products, so coverage
  counts do not guarantee end-to-end maturation depiction.
- Your goal is pathway visualisation (spaghetti diagrams) rather than enumerating
  the tool's reaction-library coverage — use RAIChU's general.py functionality instead.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Python package whose shipped tailoring-reaction and PKS/NRPS module definition
    files are enumerated to build the coverage manifest
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Runtime environment (conda, python==3.9.12) in which RAIChU is installed and
    whose package files are programmatically enumerated
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:c57c8edc9b145303e91e14fbe289bb973da05c143db00cf24c15cd3ba31c6cff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nrps-adenylation-domain-substrate-inventory

## Summary

Systematically enumerates the reaction- and substrate-definition files shipped inside the RAIChU Python package to build a structured coverage manifest of its biosynthetic chemistry: 34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, and 44 KS subtypes.

## When to use

Use when you need to quantify how comprehensively a biosynthesis-visualisation tool (RAIChU) covers tailoring reactions, NRPS A-domain substrates/starter units, KS subtypes, or extender units before relying on it to depict fully maturated natural product pathways.

## When NOT to use

- You want RAIChU to predict the regio- and stereochemistry of tailoring enzymes — the package cannot do this; users must specify reaction targets themselves.
- You need mature natural product structures generated directly from raw antiSMASH output — RAIChU generates scaffolds but not fully maturated products, so coverage counts do not guarantee end-to-end maturation depiction.
- Your goal is pathway visualisation (spaghetti diagrams) rather than enumerating the tool's reaction-library coverage — use RAIChU's general.py functionality instead.

## Inputs

- Locally installed RAIChU Python package (pip install raichu in python==3.9.12 conda environment)
- Tailoring reaction definition files inside the installed RAIChU package
- NRPS module definition files (A-domain substrates, starter units)
- PKS module definition files (KS subtypes, extender units)

## Outputs

- coverage_manifest file in the working directory containing a structured dictionary of counts (34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, 44 KS subtypes) paired with enzyme/reaction class names, plus a 'discrepancy' field if reported figures mismatch

## How to apply

Install RAIChU locally so its shipped reaction definitions are available: create a conda environment with python==3.9.12 and run 'pip install raichu' (a biopython_independent branch exists for newer Python versions). Systematically enumerate the tailoring reaction definition files in the installed package, counting each implemented tailoring reaction and tallying the RiPP families covered by family-specific posttranslational modifications. Then enumerate the NRPS module definitions to count supported A-domain substrates (281) and the starter-unit set, and the PKS module definitions to count KS subtypes (44) and the supported extender-unit set. Record each count with its corresponding enzyme/reaction class names in a structured dictionary and write it to coverage_manifest in the working directory. Validate by checking the manifest's top-level counts against the reported figures — 34 tailoring reactions, 25 RiPP families, 281 A-domain substrates, 44 KS subtypes — flagging any mismatch in a 'discrepancy' field.

## Related tools

- **RAIChU** (Python package whose shipped tailoring-reaction and PKS/NRPS module definition files are enumerated to build the coverage manifest) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Runtime environment (conda, python==3.9.12) in which RAIChU is installed and whose package files are programmatically enumerated) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Manifest top-level count for tailoring reactions equals 34 (the figure reported in the paper abstract and README).
- Manifest top-level count for RiPP families covered by family-specific posttranslational modifications equals 25.
- NRPS module enumeration yields 281 supported A-domain substrates plus the complete starter-unit set.
- PKS module enumeration yields 44 supported KS subtypes plus the supported extender-unit set.
- The 'discrepancy' field is empty when all counts match reported figures; any mismatch is explicitly flagged rather than silently accepted.

## Limitations

- Regio- and stereochemistry of most tailoring enzymes cannot be predicted, so users must specify reaction targets; coverage counts do not imply automatic target selection.
- Cationic cascade reactions in terpenes require programmatic shortcuts, so some reaction intermediates are simplified in visualisations.
- Counts reflect the release under validation (pip install raichu, python==3.9.12); the development branch may add features not captured by the manifest.

## Evidence

- [readme] scaffold diversification in natural product chemistry through the implementation of 34 tailoring reactions. These reactions encompass family-specific posttranslational modifications across 25 RiPP families: "the implementation of 34 tailoring reactions. These reactions encompass family-specific posttranslational modifications across 25 RiPP families"
- [methods] First, create a new environment with python version 3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [other] Enumerate the NRPS module definitions to count the supported A-domain substrates: [evidence span withheld — anchor status: not_found]
- [abstract] RAIChU includes 34 prevalent tailoring reactions: "RAIChU includes 34 prevalent tailoring reactions to enable the visualisation of biosynthetic pathways of fully maturated natural products"
- [results] users need to specify their reaction targets: "As it is currently not possible to predict the regio and stereochemistry of most of these tailoring enzymes, users need to specify their reaction targets"

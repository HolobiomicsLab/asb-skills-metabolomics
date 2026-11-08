---
name: pks-ketosynthase-subtype-classification
description: Use when quantifying how comprehensively a PKS/NRPS visualisation tool
  covers biosynthetic chemistry — e.g. counting KS subtypes and extender units in
  RAIChU's installed PKS module definition files for a coverage manifest.
when_to_use_negative:
- You want RAIChU to predict regio- and stereochemistry of tailoring enzymes — users
  must specify reaction targets manually
- You need mature natural product structures directly from raw antiSMASH output —
  RAIChU generates scaffolds, not maturated products
- The goal is BGC detection rather than reaction-library coverage enumeration — use
  antiSMASH or PRISM instead
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Source of the PKS module definitions whose KS subtypes and extender units
    are enumerated and counted
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Runtime environment (3.9.12) and scripting language for enumerating definition
    files and writing coverage_manifest
- name: PIKAChU
  role: Underlying tool enforcing chemical correctness of the reaction intermediates
    implemented in RAIChU's elongation and tailoring reactions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:af5c533abd0c1cfdfd568a84e44a5983bd473b15d689d84563dc4fe5afed2cb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pks-ketosynthase-subtype-classification

## Summary

Enumerate and classify the ketosynthase (KS) subtypes and supported extender-unit set implemented in the RAIChU PKS module definitions, producing a structured coverage manifest that validates reported RAIChU reaction-library coverage alongside tailoring reactions (34), RiPP families (25), and NRPS A-domain substrates (281).

## When to use

Use when quantifying how comprehensively a PKS/NRPS visualisation tool covers biosynthetic chemistry — e.g. counting KS subtypes and extender units in RAIChU's installed PKS module definition files for a coverage manifest.

## When NOT to use

- You want RAIChU to predict regio- and stereochemistry of tailoring enzymes — users must specify reaction targets manually
- You need mature natural product structures directly from raw antiSMASH output — RAIChU generates scaffolds, not maturated products
- The goal is BGC detection rather than reaction-library coverage enumeration — use antiSMASH or PRISM instead

## Inputs

- Locally installed RAIChU Python package (pip install raichu in a Python 3.9.12 environment)
- RAIChU PKS module definition files shipped with the package (KS subtypes, extender-unit set)
- Reported coverage figures used for validation (34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, 44 KS subtypes)

## Outputs

- coverage_manifest file containing a structured dictionary of counts (KS subtypes, extender units, tailoring reactions, RiPP families, NRPS A-domain substrates) with enzyme/reaction class names
- Optional 'discrepancy' field flagging mismatches against reported figures

## How to apply

Create a conda environment with Python 3.9.12 and install the package with 'pip install raichu' so the shipped reaction definitions are locally available. Systematically enumerate the PKS module definition files inside the installed RAIChU package, counting each supported KS subtype and tallying the supported extender-unit set. Record each enumerated count together with the corresponding enzyme/reaction class names in a structured dictionary, alongside tailoring-reaction counts (34), RiPP families (25), and NRPS A-domain substrates (281). Write the dictionary to coverage_manifest and validate that the top-level counts equal the reported coverage figures — 34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, and 44 KS subtypes — flagging any mismatch in a 'discrepancy' field. The rationale is that RAIChU relies on these class-specific cluster frameworks and graph-based PIKAChU reaction implementations to depict assembly-line and discrete multi-enzymatic pathways.

## Related tools

- **RAIChU** (Source of the PKS module definitions whose KS subtypes and extender units are enumerated and counted) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Runtime environment (3.9.12) and scripting language for enumerating definition files and writing coverage_manifest)
- **PIKAChU** (Underlying tool enforcing chemical correctness of the reaction intermediates implemented in RAIChU's elongation and tailoring reactions)

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Manifest's top-level counts equal the reported coverage figures: 34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, and 44 KS subtypes
- Any count mismatch is explicitly flagged in the manifest's 'discrepancy' field rather than silently written
- Each enumerated count is paired with the corresponding enzyme/reaction class names in the structured dictionary
- Enumerated KS subtypes and extender units trace back one-to-one to PKS module definition files in the installed RAIChU package

## Limitations

- Counts are tied to a specific RAIChU release; the GitHub branch notes 'new features should be tested here first', so a development branch may shift subtype coverage
- Using a newer Python version than 3.9.12 requires the biopython_independent branch per the README
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, so some reaction intermediates are simplified representations
- Coverage figures verify library breadth, not prediction accuracy of regio- and stereochemistry, which RAIChU cannot predict

## Evidence

- [other] Systematically enumerate the tailoring reaction definition files in the installed RAIChU package: [evidence span withheld — anchor status: not_found]
- [other] 34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, and 44 KS subtypes: [evidence span withheld — anchor status: not_found]
- [methods] Create a conda environment with Python 3.9.12 and install raichu via pip: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [readme] RAIChU implements 34 tailoring reactions spanning 25 RiPP families: "These reactions encompass family-specific posttranslational modifications across 25 RiPP families, enabling the exploration of advanced RiPP chemistry."
- [abstract] RAIChU coverage of PKS, NRPS and hybrid systems is validated on random systems and MIBiG: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"

---
name: tailoring-reaction-library-enumeration
description: Use when you must quantify RAIChU's built-in biosynthetic reaction coverage
  — tailoring reactions, RiPP PTM families, NRPS A-domain substrates/starter units,
  and PKS KS subtypes/extender units — before relying on it to visualise fully maturated
  pathways.
when_to_use_negative:
- When the goal is automatically validating drawn products against MIBiG characterised
  structures — the authors only manually curated select MIBiG examples and explicitly
  rejected fingerprinting-based automatic validation
- When you need automatic prediction of tailoring regio- and stereochemistry — RAIChU
  requires users to specify reaction targets
- When raw antiSMASH output must be converted into fully maturated natural products
  — RAIChU generates scaffolds, not matured products, from raw antiSMASH output
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Source of the enumerated reaction definitions; installed as a Python pip package
    so tailoring, NRPS-module, and PKS-module definition files can be inspected locally
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Runtime environment (3.9.12 via conda) in which the package is installed and
    the enumeration script builds and writes the coverage_manifest dictionary
  repo: https://pypi.org/project/raichu
- name: PIKAChU
  role: Underlying reaction engine RAIChU uses to implement elongation and tailoring
    reactions while enforcing chemical correctness of intermediates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:1fc36ec99c17cc690060d2a0595a9a78a160d8a7f4969bb7b934b7736d8a603f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tailoring-reaction-library-enumeration

## Summary

Systematically enumerate the tailoring-reaction, RiPP-family, NRPS substrate, and KS-subtype definitions shipped inside the installed RAIChU Python package, and record them as a structured coverage manifest to verify the tool's reported reaction-library coverage (34 tailoring reactions, 25 RiPP families, 281 A-domain substrates, 44 KS subtypes).

## When to use

Use when you must quantify RAIChU's built-in biosynthetic reaction coverage — tailoring reactions, RiPP PTM families, NRPS A-domain substrates/starter units, and PKS KS subtypes/extender units — before relying on it to visualise fully maturated pathways.

## When NOT to use

- When the goal is automatically validating drawn products against MIBiG characterised structures — the authors only manually curated select MIBiG examples and explicitly rejected fingerprinting-based automatic validation
- When you need automatic prediction of tailoring regio- and stereochemistry — RAIChU requires users to specify reaction targets
- When raw antiSMASH output must be converted into fully maturated natural products — RAIChU generates scaffolds, not matured products, from raw antiSMASH output

## Inputs

- Locally installed RAIChU Python package (pip install raichu, Python 3.9.12 conda environment) containing reaction-definition files
- Reported reference coverage figures: 34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, 44 KS subtypes

## Outputs

- coverage_manifest file in the working directory containing a structured dictionary of counts per enzyme/reaction class (tailoring reactions, RiPP families, NRPS A-domain substrates, starter units, KS subtypes, extender units)
- Discrepancy field flagging mismatches against the published coverage figures

## How to apply

Create a conda environment pinned to Python 3.9.12 (conda create -n raichu python==3.9.12) and install the package with 'pip install raichu' so the shipped reaction definitions are locally available. Enumerate the tailoring reaction definition files in the installed package, counting each implemented tailoring reaction and tallying the RiPP families covered by family-specific posttranslational modifications. Next, enumerate the NRPS module definitions to count supported A-domain substrates and starter units, and the PKS module definitions to count KS subtypes and extender units. Record each count with its enzyme/reaction class names in a structured dictionary and write it to coverage_manifest in the working directory. Validate that the manifest's top-level counts equal the paper's reported figures (34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, 44 KS subtypes) and flag any mismatch in a 'discrepancy' field; RAIChU implements these reactions via PIKAChU, which enforces chemical correctness of reaction intermediates.

## Related tools

- **RAIChU** (Source of the enumerated reaction definitions; installed as a Python pip package so tailoring, NRPS-module, and PKS-module definition files can be inspected locally) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Runtime environment (3.9.12 via conda) in which the package is installed and the enumeration script builds and writes the coverage_manifest dictionary) — https://pypi.org/project/raichu
- **PIKAChU** (Underlying reaction engine RAIChU uses to implement elongation and tailoring reactions while enforcing chemical correctness of intermediates)

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- coverage_manifest top-level count equals 34 tailoring reactions, matching the README and abstract claim
- RiPP family count equals 25 family-specific posttranslational modification families
- NRPS module enumeration yields 281 supported A-domain substrates
- PKS module enumeration yields 44 KS subtypes and the full supported extender-unit set
- No mismatch is left unflagged: any divergence from published figures appears in the manifest's 'discrepancy' field

## Limitations

- Regio- and stereochemistry of most tailoring enzymes cannot be predicted; users must specify reaction targets manually
- Cationic cascade reactions in terpenes require programmatic shortcuts, so reaction intermediates may be simplified (RAIChU offers a summarised pathway view showing only initial and final intermediates)
- The installation path is pinned to Python 3.9.12; newer Python versions require the biopython_independent branch, and enumeration counts reflect the specific package version shipped

## Evidence

- [abstract] RAIChU includes 34 prevalent tailoring reactions to enable the visualisation of biosynthetic pathways of fully maturated natural products: "RAIChU includes 34 prevalent tailoring reactions to enable the visualisation of biosynthetic pathways of fully maturated natural products"
- [readme] implementation of 34 tailoring reactions across 25 RiPP families: "the implementation of 34 tailoring reactions. These reactions encompass family-specific posttranslational modifications across 25 RiPP families"
- [readme] create environment and install via pip: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] PIKAChU enforces chemical correctness: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates"
- [results] users must specify tailoring reaction targets: "As it is currently not possible to predict the regio and stereochemistry of most of these tailoring enzymes, users need to specify their reaction targets"

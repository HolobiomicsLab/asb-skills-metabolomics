---
name: coverage-manifest-construction
description: Use when you need to audit or reproduce the reported reaction-library
  coverage of a pip-installed biosynthesis visualisation package (e.g. RAIChU's 34
  tailoring reactions, 25 RiPP families, 281 A-domain substrates, 44 KS subtypes)
  from its shipped reaction definition files.
when_to_use_negative:
- You want to generate or visualise a specific biosynthetic pathway ('spaghetti diagram')
  rather than audit the reaction library's coverage — use RAIChU's depiction functionality
  instead.
- The package version under audit has not been installed into the environment, so
  the reaction definition files are not locally enumerable.
- Your goal is BGC detection or product structure prediction from nucleotide input
  — RAIChU is a visualisation tool, not a prediction platform like antiSMASH or PRISM.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Target package whose installed reaction definition files (tailoring reactions,
    RiPP family modifications, NRPS and PKS module definitions) are enumerated and
    counted
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Environment (Python 3.9.12 via conda) and scripting language used to enumerate
    definition files, tally counts, and write the manifest dictionary
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:98ab927f31a7804578269511722195656eb106354a73f60bba7425ed8e25327e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# -coverage-manifest-construction

## Summary

Constructs a structured coverage manifest that quantifies how comprehensively RAIChU's built-in reaction library covers natural product tailoring chemistry — enumerating tailoring reactions, RiPP families, NRPS A-domain substrates, and PKS KS subtypes directly from the installed package's reaction definition files.

## When to use

Use when you need to audit or reproduce the reported reaction-library coverage of a pip-installed biosynthesis visualisation package (e.g. RAIChU's 34 tailoring reactions, 25 RiPP families, 281 A-domain substrates, 44 KS subtypes) from its shipped reaction definition files.

## When NOT to use

- You want to generate or visualise a specific biosynthetic pathway ('spaghetti diagram') rather than audit the reaction library's coverage — use RAIChU's depiction functionality instead.
- The package version under audit has not been installed into the environment, so the reaction definition files are not locally enumerable.
- Your goal is BGC detection or product structure prediction from nucleotide input — RAIChU is a visualisation tool, not a prediction platform like antiSMASH or PRISM.

## Inputs

- pip-installed RAIChU package (Python 3.9.12 conda environment) with its shipped tailoring reaction definition files
- RAIChU NRPS module definition files (A-domain substrates, starter units)
- RAIChU PKS module definition files (KS subtypes, extender units)

## Outputs

- coverage_manifest. — a structured dictionary/manifest in the working directory recording counts of tailoring reactions, RiPP families, A-domain substrates, and KS subtypes with enzyme/reaction class names and a 'discrepancy' field

## How to apply

Create a conda environment with Python 3.9.12 and install the package with 'pip install raichu' so the shipped reaction definitions are locally available. Systematically enumerate the tailoring reaction definition files inside the installed RAIChU package, counting each implemented tailoring reaction and tallying the RiPP families covered by family-specific posttranslational modifications. Separately enumerate the NRPS module definitions to count supported A-domain substrates and the supported starter-unit set, and the PKS module definitions to count KS subtypes and supported extender units. Record each count together with the corresponding enzyme/reaction class names in a structured dictionary and write it to coverage_manifest. in the working directory. Validation: check that the manifest's top-level counts equal the reported coverage figures — 34 tailoring reactions, 25 RiPP families, 281 NRPS A-domain substrates, and 44 KS subtypes — flagging any mismatch in a 'discrepancy' field.

## Related tools

- **RAIChU** (Target package whose installed reaction definition files (tailoring reactions, RiPP family modifications, NRPS and PKS module definitions) are enumerated and counted) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Environment (Python 3.9.12 via conda) and scripting language used to enumerate definition files, tally counts, and write the manifest dictionary)

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu; python -c "import raichu, os; print(sorted(os.listdir(os.path.join(os.path.dirname(raichu.__file__),'reaction_definitions'))))"
```

## Evaluation signals

- Manifest's tailoring-reaction count equals 34, matching the figure reported in the article abstract and README.
- Manifest's RiPP-family count equals 25, matching the reported number of families with family-specific posttranslational modifications.
- Manifest's NRPS A-domain substrate count equals 281 and KS subtype count equals 44 per the source card's expected coverage figures.
- No 'discrepancy' field content is populated, i.e. all top-level counts match the reported values exactly.
- Each enumerated count is accompanied by the corresponding enzyme/reaction class names recorded in the structured dictionary.

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot currently be predicted, so reaction targets must be user-specified — coverage counts capture implemented reactions, not predictive coverage.
- Some reactions, such as cationic cascade reactions in terpenes, require programmatic shortcuts that may lead to simplified representations of reaction intermediates.
- Counts are version-dependent: the manifest reflects the shipped definitions of the installed package version (README targets the development branch; a biopython_independent branch exists for newer Python versions).

## Evidence

- [abstract] RAIChU includes 34 prevalent tailoring reactions to enable the visualisation of biosynthetic pathways of fully maturated natural products: "RAIChU includes 34 prevalent tailoring reactions to enable the visualisation of biosynthetic pathways of fully maturated natural products"
- [methods] First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [readme] implementation of 34 tailoring reactions... encompass family-specific posttranslational modifications across 25 RiPP families: "through the implementation of 34 tailoring reactions. These reactions encompass family-specific posttranslational modifications across 25 RiPP families"
- [readme] A unique feature of RAIChU is its capability to detect and visually highlight potential sites for these tailoring reactions on molecules: "A unique feature of RAIChU is its capability to detect and visually highlight potential sites for these tailoring reactions on molecules"

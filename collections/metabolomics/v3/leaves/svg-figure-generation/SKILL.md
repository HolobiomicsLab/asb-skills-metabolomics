---
name: svg-figure-generation
description: Use when you need to programmatically draw a biosynthetic gene cluster
  (e.g. a MIBiG modular PKS/NRPS entry) as a spaghetti diagram with a vertically aligned
  backbone, exported to an .svg file.
when_to_use_negative:
- Input is not a modular PKS/NRPS system or lacks a cluster definition (e.g. RiPP/terpene
  pathways needing custom user-specified enzymatic steps)
- The goal is predicted scaffold structures rather than diagram rendering — RAIChU
  cannot produce mature natural products from raw antiSMASH output without user-specified
  tailoring targets
- Regio- and stereochemistry of tailoring reactions must be predicted automatically;
  RAIChU requires users to specify reaction targets
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0780
tools:
- name: RAIChU
  role: Drawing engine whose spaghetti-diagram layout rules (central-chain 60/-60°,
    cyclic 90°, side-chain 0/180° bond angles) are reconstructed to emit the SVG diagram
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Runtime environment (Python 3.9.12) in which the raichu package is installed
    and the SVG rendering workflow is implemented
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:3234079a8d37bf34f5a50d33d361101f8786d839bec88552281c43c3faffad05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# svg-figure-generation

## Summary

Reconstruct RAIChU's spaghetti-diagram drawing engine to render modular PKS/NRPS cluster visualisations as SVG files, applying its central-chain, cyclic-bond, and side-chain angle rules so diagrams meet the paper's readability criteria.

## When to use

Use when you need to programmatically draw a biosynthetic gene cluster (e.g. a MIBiG modular PKS/NRPS entry) as a spaghetti diagram with a vertically aligned backbone, exported to an .svg file.

## When NOT to use

- Input is not a modular PKS/NRPS system or lacks a cluster definition (e.g. RiPP/terpene pathways needing custom user-specified enzymatic steps)
- The goal is predicted scaffold structures rather than diagram rendering — RAIChU cannot produce mature natural products from raw antiSMASH output without user-specified tailoring targets
- Regio- and stereochemistry of tailoring reactions must be predicted automatically; RAIChU requires users to specify reaction targets

## Inputs

- Modular PKS/NRPS BGC cluster definition (e.g. a MIBiG database entry)
- raichu Python package installed via pip in a Python 3.9.12 environment

## Outputs

- SVG spaghetti diagram file (e.g. mibig_spaghetti_diagram.svg) with domains as circles, genes as rectangles, and a vertically aligned backbone

## How to apply

Create a conda environment with Python 3.9.12 and install the raichu pip package. Load a modular PKS/NRPS cluster definition (e.g. a MIBiG entry) and implement the RaichuDrawer central-chain placement: alternate central-chain bond angles between 60 and -60 degrees, set cyclic-bond angles to 90 degrees, set side-chain angles to 0/180 degrees, and apply clash-resolution finetuning to remove overlaps. Render domains as circles and genes as rectangles around a vertically aligned backbone across modules, then export the result as an SVG file (e.g. mibig_spaghetti_diagram.svg) matching the separate .svg spaghetti-diagram files RAIChU produces. The rationale is that RAIChU achieved 100% drawing correctness and 97.66% drawing readability using these layout rules, so reproducing them preserves those quality characteristics.

## Related tools

- **RAIChU** (Drawing engine whose spaghetti-diagram layout rules (central-chain 60/-60°, cyclic 90°, side-chain 0/180° bond angles) are reconstructed to emit the SVG diagram) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Runtime environment (Python 3.9.12) in which the raichu package is installed and the SVG rendering workflow is implemented) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- Manually inspect the SVG against the paper's readability criteria: correctness of depicted structures, correct bond angles, and absence of overlap
- Central-chain bonds alternate between 60 and -60 degrees, cyclic bonds are at 90 degrees, and side-chain bonds are at 0/180 degrees
- The output is a valid .svg file matching the separate .svg spaghetti-diagram files RAIChU produces (compare against Zenodo 10.5281/zenodo.10987931 exemplars)
- The backbone is vertically aligned across modules, with domains drawn as circles and genes as rectangles
- Drawing completes without unexpected errors when run on MIBiG entries, as RAIChU did on the entire MIBiG database

## Limitations

- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may differ
- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, potentially leading to simplified representations of reaction intermediates

## Evidence

- [other] RAIChU's spaghetti diagram drawing logic renders bonds attached to the backbone at a 0° or 180° angle: [evidence span withheld — anchor status: not_found]
- [results] correctness of the depicted structures: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [abstract] drawing readability (97.66%) were validated: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [readme] create a new environment with python version 3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [readme] Then, install raichu with pip: pip install raichu: "Then, install raichu with pip: pip install raichu"

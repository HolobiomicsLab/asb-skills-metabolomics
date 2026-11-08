---
name: modular-pks-nrps-domain-gene-annotation-rendering
description: Use when you need to visualise a modular PKS, NRPS, or hybrid PKS/NRPS
  BGC (e.g. from MIBiG or antiSMASH output) as a module-resolution 'spaghetti diagram'
  in SVG format.
when_to_use_negative:
- Input is raw antiSMASH output and the goal is a mature natural product structure
  — RAIChU generates natural product scaffolds but not mature natural products from
  raw antiSMASH output
- The BGC belongs to a RiPP or terpene class where tailoring, cyclisation, and proteolytic
  cleavage occur in flexible order and the summarized pathway view applies rather
  than modular spaghetti rendering
- An automated structural validation (e.g. Morgan fingerprinting) is required instead
  of a visual diagram — RAIChU's fingerprinting-based validation was considered but
  not implemented
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3312
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Provides the RaichuDrawer drawing engine whose bond-angle placement rules
    and clash-resolution logic are reconstructed to emit the SVG spaghetti diagram
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of BGC detection and annotation results that can be uploaded and edited
    in RAIChU as cluster input
- name: Python
  role: Runtime environment (pinned to 3.9.12) into which raichu is installed via
    pip so the drawer can be invoked in Python pipelines
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:c64b9dc379b17b53d700e536dbb7c37ea2e7c46844fd580390c7852c0347c546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3312
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# modular-pks-nrps-domain-gene-annotation-rendering

## Summary

Reconstructs RAIChU's spaghetti-diagram drawing engine to render modular PKS/NRPS biosynthetic gene clusters as SVGs with domains as circles, genes as rectangles, and a vertically aligned backbone, applying RAIChU's specific bond-angle layout rules and clash-resolution to meet the paper's 97.66% drawing-readability criteria.

## When to use

Use when you need to visualise a modular PKS, NRPS, or hybrid PKS/NRPS BGC (e.g. from MIBiG or antiSMASH output) as a module-resolution 'spaghetti diagram' in SVG format.

## When NOT to use

- Input is raw antiSMASH output and the goal is a mature natural product structure — RAIChU generates natural product scaffolds but not mature natural products from raw antiSMASH output
- The BGC belongs to a RiPP or terpene class where tailoring, cyclisation, and proteolytic cleavage occur in flexible order and the summarized pathway view applies rather than modular spaghetti rendering
- An automated structural validation (e.g. Morgan fingerprinting) is required instead of a visual diagram — RAIChU's fingerprinting-based validation was considered but not implemented

## Inputs

- Modular PKS/NRPS (or hybrid) BGC cluster definition, e.g. a MIBiG database entry
- antiSMASH BGC detection/annotation results that can be uploaded and edited in RAIChU

## Outputs

- SVG spaghetti diagram (e.g. mibig_spaghetti_diagram.svg) with domains as circles, genes as rectangles, and a vertically aligned backbone
- GIF animation of generated clusters per validation set, as distributed via Zenodo 10.5281/zenodo.10987931

## How to apply

Set up a conda environment pinned to Python 3.9.12 and pip install raichu, then load a cluster definition — for example a modular PKS/NRPS entry from the MIBiG database — and drive the RaichuDrawer central-chain placement logic. Alternate central-chain bond angles between 60 and -60 degrees, set cyclic-bond angles to 90 degrees, and fix side-chain (backbone-attached) bonds at 0° or 180° with respect to the drawing plane, which RAIChU uses as a readability criterion. Apply clash-resolution finetuning to remove overlaps between drawn elements before rendering. Emit domains as circles, genes as rectangles, and the backbone vertically aligned across modules, writing the result to an SVG file such as mibig_spaghetti_diagram.svg, matching the separate .svg spaghetti-diagram files RAIChU distributes per validation set. Finally validate the SVG manually against the paper's three readability characteristics — correctness of the depicted structures, bond angles, and absence of overlap.

## Related tools

- **RAIChU** (Provides the RaichuDrawer drawing engine whose bond-angle placement rules and clash-resolution logic are reconstructed to emit the SVG spaghetti diagram) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of BGC detection and annotation results that can be uploaded and edited in RAIChU as cluster input)
- **Python** (Runtime environment (pinned to 3.9.12) into which raichu is installed via pip so the drawer can be invoked in Python pipelines) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- SVG shows domains as circles, genes as rectangles, and the backbone vertically aligned across modules
- Central-chain bonds alternate at 60/-60 degrees, cyclic bonds at 90 degrees, and backbone-attached side-chain bonds at 0°/180° to the drawing plane
- No overlapping elements after clash-resolution finetuning (absence of overlap is one of the paper's readability criteria)
- Depicted structures are chemically correct (structure correctness criterion, part of RAIChU's 100% correctness / 97.66% readability scores)
- Output matches the style of the separate .svg spaghetti-diagram files distributed per validation set on Zenodo (10.5281/zenodo.10987931)

## Limitations

- Requires Python 3.9.12 for the main branch; newer Python versions need the biopython_independent branch
- Double bond stereochemistry following dehydration reactions in PKS systems is a known source of discrepancy (e.g. observed in the CMC-thuggacins)
- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify reaction targets
- Validation relied on manual curation of select MIBiG examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters) rather than automated validation of all MIBiG BGCs

## Evidence

- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [results] We manually assessed each spaghetti diagram on three characteristics: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [results] Spaghetti diagrams are available as separate.svg files, and.gif files, showing an animation of all generated clusters per validation set: "Spaghetti diagrams are available as separate.svg files, and.gif files, showing an animation of all generated clusters per validation set"
- [readme] RAIChU is a chemoinformatics tool designed to illuminate the intricate processes of scaffold diversification in natural product chemistry: "RAIChU is a chemoinformatics tool designed to illuminate the intricate processes of scaffold diversification in natural product chemistry"
- [methods] Our tool is particularly adept at visualizing the sequence of tailoring reactions in both modular systems: "Our tool is particularly adept at visualizing the sequence of tailoring reactions in both modular systems"

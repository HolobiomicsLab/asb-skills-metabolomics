---
name: biosynthetic-gene-cluster-spaghetti-diagram-layout
description: Use when a predicted or MIBiG-derived modular PKS/NRPS BGC needs an automated,
  readable pathway visualisation at module resolution (spaghetti diagram in SVG),
  or when manually drawn biosynthetic models are too laborious/error-prone.
when_to_use_negative:
- The goal is BGC detection/annotation from nucleotide sequence — RAIChU visualises
  clusters but, unlike antiSMASH, does not predict gene clusters
- The task requires predicting regio- and stereochemistry of tailoring enzymes — users
  must specify reaction targets themselves, so untargeted tailoring prediction is
  out of scope
- A mature natural product must be generated directly from raw antiSMASH output —
  RAIChU can generate natural product scaffolds but not mature natural products in
  that setting
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Provides the drawing engine (RaichuDrawer) that lays out the central chain,
    cyclic bonds, and side-chain bonds and renders the spaghetti diagram to SVG
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of BGC detection/annotation results that users can upload and edit
    as RAIChU input clusters
- name: Python
  role: Runtime environment (Python 3.9.12) and scripting interface for integrating
    RAIChU's RaichuDrawer into pipelines
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:efb10483bcaf44e4d7b154b0a260d61ffe5d5c41b3420f828f257ed2ff9be8bd
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# biosynthetic-gene-cluster-spaghetti-diagram-layout

## Summary

Renders a modular PKS/NRPS biosynthetic gene cluster as a RAIChU 'spaghetti diagram' — domains as circles, genes as rectangles, and a vertically aligned backbone across modules — exported as an SVG, using RAIChU's bond-angle layout rules to maximise drawing readability.

## When to use

Use when a predicted or MIBiG-derived modular PKS/NRPS BGC needs an automated, readable pathway visualisation at module resolution (spaghetti diagram in SVG), or when manually drawn biosynthetic models are too laborious/error-prone.

## When NOT to use

- The goal is BGC detection/annotation from nucleotide sequence — RAIChU visualises clusters but, unlike antiSMASH, does not predict gene clusters
- The task requires predicting regio- and stereochemistry of tailoring enzymes — users must specify reaction targets themselves, so untargeted tailoring prediction is out of scope
- A mature natural product must be generated directly from raw antiSMASH output — RAIChU can generate natural product scaffolds but not mature natural products in that setting

## Inputs

- A modular PKS/NRPS (or hybrid) BGC cluster definition, e.g. a MIBiG database entry or an edited/uploaded antiSMASH result
- User-specified tailoring reaction targets (required where regio- and stereochemistry of tailoring enzymes cannot be predicted)

## Outputs

- A spaghetti diagram rendered as an SVG file (e.g. mibig_spaghetti_diagram.svg) with domains as circles, genes as rectangles, and a vertically aligned backbone
- Optionally animated .gif files showing all generated clusters per validation set (cf. RAIChU's Zenodo deposit 10.5281/zenodo.10987931)

## How to apply

Set up a conda environment with Python 3.9.12 and install the raichu package via pip. Load a modular PKS/NRPS cluster entry (e.g. from the MIBiG database, or an antiSMASH result edited/uploaded by the user) as the cluster definition. Drive RaichuDrawer's central-chain placement: alternate central-chain bond angles between 60 and -60 degrees, set cyclic-bond angles to 90 degrees, and set side-chain (spaghetti) bonds attached to the backbone to 0 or 180 degrees with respect to the drawing plane; then apply clash-resolution finetuning to remove overlaps. Draw the cluster with domains as circles, genes as rectangles, and a vertically aligned backbone across modules, and render the result to an SVG file (e.g. mibig_spaghetti_diagram.svg) matching the separate .svg spaghetti-diagram files RAIChU produces.

## Related tools

- **RAIChU** (Provides the drawing engine (RaichuDrawer) that lays out the central chain, cyclic bonds, and side-chain bonds and renders the spaghetti diagram to SVG) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of BGC detection/annotation results that users can upload and edit as RAIChU input clusters)
- **Python** (Runtime environment (Python 3.9.12) and scripting interface for integrating RAIChU's RaichuDrawer into pipelines) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Manually assess the diagram on the three validation characteristics: correctness of the depicted structures, bond angles, and absence of overlap — the criteria behind RAIChU's 97.66% drawing readability
- Backbone is vertically aligned across modules, with domains drawn as circles and genes as rectangles
- Side-chain bonds attached to the backbone sit at 0° or 180° with respect to the drawing plane; central-chain bonds alternate between 60° and -60°; cyclic bonds sit at 90°
- No clashes/overlaps remain after finetuning
- Output SVG is comparable to the separate .svg spaghetti-diagram files RAIChU releases (Zenodo 10.5281/zenodo.10987931)

## Limitations

- Regio- and stereochemistry of most tailoring enzymes cannot currently be predicted; users must specify reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (e.g. observed in the CMC-thuggacins)
- For some chemistry, such as cationic cascade reactions in terpenes, programmatic shortcuts may yield simplified representations of reaction intermediates

## Evidence

- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [results] side-chain bond angle rule: [evidence span withheld — anchor status: not_found]
- [results] validation criteria: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [results] SVG output format: "Spaghetti diagrams are available as separate.svg files, and.gif files, showing an animation of all generated clusters per validation set (10.5281/zenodo.10987931)"
- [methods] environment setup: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [readme] package install: "Then, install raichu with pip: pip install raichu"

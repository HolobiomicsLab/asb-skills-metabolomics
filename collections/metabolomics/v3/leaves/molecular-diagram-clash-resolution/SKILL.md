---
name: molecular-diagram-clash-resolution
description: Use when rendering spaghetti diagrams of modular PKS/NRPS clusters whose
  bonds or labels overlap, or when reproducing RAIChU's drawing engine layout rules
  for vertically aligned backbones in SVG output.
when_to_use_negative:
- The task is structure or product prediction rather than visualisation — use antiSMASH/PRISM
  prediction or PIKAChU chemical-correctness checking instead
- The target is non-modular chemistry (e.g. RiPP or terpene tailoring pathways) where
  the modular vertical-backbone spaghetti layout rules do not directly apply
- The diagram is for a summarized pathway view (initial and final intermediates only),
  which uses a different simplified representation
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RAIChU
  role: Provides the drawing engine whose bond-angle placement rules and clash-resolution
    finetuning produce the spaghetti diagram layout
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Runtime environment for installing RAIChU via pip and scripting the RaichuDrawer
    rendering
  repo: https://pypi.org/project/raichu
- name: antiSMASH
  role: Source of BGC detections/annotations that RAIChU can upload and draw as spaghetti
    diagrams
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:534f1bb05c165f76f77d2657ea409550c68498a12aed6a1f4457d7ce55107342
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# molecular-diagram-clash-resolution

## Summary

Applies RAIChU's clash-resolution finetuning to spaghetti-diagram layouts of modular PKS/NRPS biosynthetic gene clusters, removing overlaps by constraining central-chain bond angles to alternating 60/-60 degrees, cyclic bonds to 90 degrees, and side-chain bonds to 0/180 degrees so diagrams meet the paper's readability criteria.

## When to use

Use when rendering spaghetti diagrams of modular PKS/NRPS clusters whose bonds or labels overlap, or when reproducing RAIChU's drawing engine layout rules for vertically aligned backbones in SVG output.

## When NOT to use

- The task is structure or product prediction rather than visualisation — use antiSMASH/PRISM prediction or PIKAChU chemical-correctness checking instead
- The target is non-modular chemistry (e.g. RiPP or terpene tailoring pathways) where the modular vertical-backbone spaghetti layout rules do not directly apply
- The diagram is for a summarized pathway view (initial and final intermediates only), which uses a different simplified representation

## Inputs

- Modular PKS/NRPS or hybrid PKS/NRPS BGC definition (e.g. a MIBiG database cluster entry)
- antiSMASH BGC detection/annotation results uploaded to RAIChU
- User-constructed biosynthetic PKS/NRPS system definitions

## Outputs

- SVG spaghetti diagram file (e.g. mibig_spaghetti_diagram.svg) with vertically aligned backbone, circular domains, and rectangular genes
- Clash-resolved bond layout with central-chain angles alternating 60/-60 degrees, cyclic bonds at 90 degrees, and side-chain bonds at 0/180 degrees

## How to apply

Install RAIChU in a Python 3.9.12 conda environment via pip and load a modular PKS/NRPS BGC (e.g. from the MIBiG database) as the drawing input. Implement the RaichuDrawer central-chain placement by alternating central-chain bond angles between 60 and -60 degrees so the backbone runs vertically across modules, with domains drawn as circles and genes as rectangles. Set cyclic-bond angles to 90 degrees and side-chain angles to 0 or 180 degrees with respect to the drawing plane; this 0/180 rule for backbone-attached bonds is the readability criterion RAIChU uses. Then apply clash-resolution finetuning to remove any remaining overlaps between bonds, substituents, or domain labels. Render the result as an SVG (e.g. mibig_spaghetti_diagram.svg) matching the separate .svg spaghetti-diagram files RAIChU emits. Validate by manually inspecting structure correctness, bond angles, and absence of overlap, the same three characteristics used to score RAIChU's 97.66% drawing readability.

## Related tools

- **RAIChU** (Provides the drawing engine whose bond-angle placement rules and clash-resolution finetuning produce the spaghetti diagram layout) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Runtime environment for installing RAIChU via pip and scripting the RaichuDrawer rendering) — https://pypi.org/project/raichu
- **antiSMASH** (Source of BGC detections/annotations that RAIChU can upload and draw as spaghetti diagrams)

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Manual inspection confirms no overlapping bonds, substituents, or labels after clash-resolution finetuning
- Central-chain bonds alternate between 60 and -60 degrees and the backbone is vertically aligned across modules
- Side-chain bonds attached to the backbone are rendered at 0° or 180° with respect to the drawing plane
- Cyclic bonds are set to 90 degrees and depicted structures are chemically correct
- Output SVG matches RAIChU's separate .svg spaghetti-diagram files and the layout would fall within the 97.66% readability band from validation

## Limitations

- Programmatic shortcuts (e.g. for cationic cascade reactions in terpenes) can lead to simplified representations of reaction intermediates
- Python 3.9.12 is required for the main branch; newer Python versions require the biopython_independent branch
- Cationic cascade reactions may require a summarized pathway view showing only initial and final intermediates

## Evidence

- [other] RAIChU's spaghetti diagram drawing logic renders bonds attached to the backbone at a 0° or 180° angle with respect to the drawing plane, a layout rule used as a readability criterion: [evidence span withheld — anchor status: not_found]
- [results] We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [readme] If you want to use a newer python version, use the instructions in the biopython_independent branch.: "If you want to use a newer python version, use the instructions in the biopython_independent branch."

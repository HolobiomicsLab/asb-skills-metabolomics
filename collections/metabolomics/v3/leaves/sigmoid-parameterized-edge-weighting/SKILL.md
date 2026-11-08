---
name: sigmoid-parameterized-edge-weighting
description: Use when building a weighted molecular network for random-walk annotation
  propagation and you must fuse MetFrag annotation scores with spectral-cosine and
  structural-similarity edges into non-negative edge weights.
when_to_use_negative:
- Your molecular network already ships with edge weights from another fusion scheme
  (e.g. plain NAP-style local propagation) — this skill specifically implements the
  MetFusion-style sigmoid weighting.
- You only need the downstream random walk on an existing weighted graphml and have
  no MetFrag candidates or spectral cosine similarities to fuse.
- The problem is purely spectral library matching without candidate structures; then
  there is no structural similarity term to feed the sigmoid.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: MetFrag
  role: Provides the combinatorial in silico fragmentation candidate annotation scores
    that form the alpha-weighted component of each edge weight
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Supplies the Fusion scoring concept used to blend spectral cosine, structural
    similarity and MetFrag scores via a sigmoid weight function
- name: ChemWalker
  role: Assembles the weighted graph in graphml format and consumes it for random-walk
    annotation propagation
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Runs the random walk with restart over the constructed weighted graph after
    edge weighting is complete
  repo: https://github.com/jinhongjung/pyrwr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:f38ccc6c6a8cfafcadd19c475a30ebe371c7288aaef1b646b97447e64b4bf84f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - pyrwr
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sigmoid-parameterized-edge-weighting

## Summary

Constructs the weighted graph fed to ChemWalker's random walk by combining MetFrag candidate scores, spectral cosine similarity and chemical (structural) similarity through a MetFusion-style sigmoid weight function, parameterized by alpha.

## When to use

Use when building a weighted molecular network for random-walk annotation propagation and you must fuse MetFrag annotation scores with spectral-cosine and structural-similarity edges into non-negative edge weights.

## When NOT to use

- Your molecular network already ships with edge weights from another fusion scheme (e.g. plain NAP-style local propagation) — this skill specifically implements the MetFusion-style sigmoid weighting.
- You only need the downstream random walk on an existing weighted graphml and have no MetFrag candidates or spectral cosine similarities to fuse.
- The problem is purely spectral library matching without candidate structures; then there is no structural similarity term to feed the sigmoid.

## Inputs

- MetFrag in silico fragmentation candidate scores (e.g. MetFrag2.3-CL.jar output)
- Molecular-network edges with spectral cosine similarity scores (e.g. from GNPS feature-based molecular networking)
- Chemical (structural) similarity values among candidate structures
- Alpha parameter controlling the balance between MetFrag score averaging and similarity terms

## Outputs

- Weighted graph (graphml format) with non-negative edge weights w_ij ready for random-walk propagation
- Set of candidate-structure edge weights derived from the MetFusion-style sigmoid function

## How to apply

Parse the MetFrag in silico fragmentation results to obtain each candidate's annotation score, then compute spectral cosine similarities between connected molecular-network nodes and chemical (structural) similarities among the candidate structures. Combine them with the MetFusion Fusion scoring concept: an alpha-weighted average of the two nodes' MetFrag scores is blended (weight 1-alpha) with the sigmoid of the spectral cosine score multiplied by the structural similarity, yielding edge weights w_ij. Assemble the nodes and these computed edge weights into a weighted graph in graphml format, which is subsequently consumed by the random walk starting from seed nodes (spectral library matches). The alpha parameter controls the relative influence of library matches versus candidate similarity, mirroring MetFusion's spectral-library ranking improvement logic. The paper evaluates this mechanism only as part of the end-to-end random-walk propagation, so validate it structurally rather than with a standalone score.

## Related tools

- **MetFrag** (Provides the combinatorial in silico fragmentation candidate annotation scores that form the alpha-weighted component of each edge weight) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Supplies the Fusion scoring concept used to blend spectral cosine, structural similarity and MetFrag scores via a sigmoid weight function)
- **ChemWalker** (Assembles the weighted graph in graphml format and consumes it for random-walk annotation propagation) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Runs the random walk with restart over the constructed weighted graph after edge weighting is complete) — https://github.com/jinhongjung/pyrwr

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- The assembled graphml loads without error and every edge carries a finite, non-negative weight w_ij.
- The alpha-weighted MetFrag component equals the average of the two connected nodes' candidate scores when alpha is set to the MetFusion-style weighting.
- The sigmoid term equals sigmoid(spectral_cosine * chemical_similarity), confirming the product term is computed before the sigmoid transform.
- Changing alpha shifts edge weights monotonically between the MetFrag-score-only and similarity-only extremes, as expected from the 1-alpha weighting.
- The random walk launched from seed nodes (spectral library matches) produces candidate probabilities that improve ranking compared with raw MetFrag scores alone, as reported in the paper.

## Limitations

- The paper evaluates this weighting mechanism only as part of the end-to-end random-walk propagation, not via a standalone quantitative criterion.
- The construction depends on MetFrag in silico fragmentation candidates; if candidate lists are missing or low quality for a node, the alpha-weighted term degenerates.
- MetFusion-style Fusion scoring was borrowed for ranking improvement from the spectral library, so its behavior under very low cosine scores (sigmoid saturation) is not independently characterized in the paper.

## Evidence

- [abstract] MetFusion-style sigmoid edge weighting: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
- [methods] fusion blending of candidates and similarities: "ChemWalker uses the Fusion concept, proposed on MetFusion, for ranking improvement from spectral library."
- [readme] metfrag dependency for the weighting scheme: "ChemWalker uses MetFrag for in silico"
- [methods] graphml output of the weighted graph: [evidence span withheld — anchor status: not_found]
- [abstract] random walk consumes the weighted graph: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates"

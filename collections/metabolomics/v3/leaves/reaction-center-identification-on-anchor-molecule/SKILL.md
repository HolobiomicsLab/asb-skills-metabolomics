---
name: reaction-center-identification-on-anchor-molecule
description: Use when you have a known anchor molecule (SMILES) with high spectral
  similarity to an unknown suspect spectrum, a set of generated derivative candidates,
  and need to rank them by likely reaction-center likelihood.
when_to_use_negative:
- When the suspect spectrum has no known anchor molecule with high spectral similarity
  to establish the biotransformation context
- When candidate structures were generated without applying biotransformation rules
  to a known anchor (e.g., purely formula- or database-based annotation)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: GNN-SOM
  role: Predicts site-of-metabolism likelihood for each atom in the anchor/candidate
    molecular graphs using a graph neural network; these likelihoods rank the candidate
    derivatives
  repo: https://github.com/HassounLab/GNN-SOM
- name: PROXIMAL2
  role: Applies biotransformation rules to the anchor molecule, identifying suitable
    reaction centers and generating candidate derivative structures that are subsequently
    ranked
  repo: https://github.com/HassounLab/PROXIMAL2
- name: RDKit
  role: Converts the anchor molecule and each candidate derivative to molecular graphs
    consumed by GNN-SOM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:1218867d2b1f46083e784f9629b7629941e6cccf53620dcf8acfaec2fe731fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/GNN-SOM
  tools:
  - BAM
  - BioServices
  - GNN-SOM
  - KEGG
  - PROXIMAL2
  - PubChemPy
  - Python 3.9
  - RDKit
  - RetroRules
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# reaction-center-identification-on-anchor-molecule

## Summary

Identify likely reaction centers on a known anchor molecule and use their site-of-metabolism likelihoods to rank biotransformation-derived candidate structures for the suspect compound, via GNN-SOM graph neural network predictions over the anchor's molecular graph.

## When to use

Use when you have a known anchor molecule (SMILES) with high spectral similarity to an unknown suspect spectrum, a set of generated derivative candidates, and need to rank them by likely reaction-center likelihood.

## When NOT to use

- When the suspect spectrum has no known anchor molecule with high spectral similarity to establish the biotransformation context
- When candidate structures were generated without applying biotransformation rules to a known anchor (e.g., purely formula- or database-based annotation)

## Inputs

- Anchor molecule structure as SMILES
- Candidate derivative structures produced by PROXIMAL2 from the anchor molecule
- Mapping of each derivative to the atom (reaction center) on the anchor to which its biotransformation rule was applied
- Observed precursor mass difference between suspect and anchor (to match against biotransformation rules)

## Outputs

- Ranked list of candidate derivative structures with site-of-metabolism likelihood scores
- Rank position of the true suspect structure (target: within top-5)
- Tie-resolved average ranks for equally scored candidates

## How to apply

Convert each candidate derivative and the anchor molecule to a molecular graph with RDKit, then apply the GNN-SOM PyTorch Geometric model to predict the site-of-metabolism likelihood for each atom. Assign each derivative candidate the likelihood of the specific atom to which its biotransformation rule was applied by PROXIMAL2, and rank candidates by descending likelihood. Resolve rank ties with the average-rank method: sum all ranks including duplicates and divide by the number of tied candidates. A run is successful when the true suspect structure appears within the top-5 ranked candidates (the paper reports 24.2% correct at rank 1).

## Related tools

- **GNN-SOM** (Predicts site-of-metabolism likelihood for each atom in the anchor/candidate molecular graphs using a graph neural network; these likelihoods rank the candidate derivatives) — https://github.com/HassounLab/GNN-SOM
- **PROXIMAL2** (Applies biotransformation rules to the anchor molecule, identifying suitable reaction centers and generating candidate derivative structures that are subsequently ranked) — https://github.com/HassounLab/PROXIMAL2
- **RDKit** (Converts the anchor molecule and each candidate derivative to molecular graphs consumed by GNN-SOM)

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- The true suspect structure appears among the top-5 ranked candidates (paper reports 24.2% of cases correctly at rank 1)
- Correct candidates are defined by matching the first 14 characters of the InChIKey against the true suspect structure
- Atom-level site-of-metabolism likelihoods are non-null for every atom in each candidate graph processed by GNN-SOM
- Ties in likelihood scores are consistently resolved via the average-rank method across the ranked candidate table

## Limitations

- Ranking depends entirely on the accuracy of GNN-SOM atom-level likelihoods; no experimental confirmation of reaction centers is performed
- BAM assigned the correct structure in only 24.2% of examined anchor–suspect cases, so many cases will not resolve to rank 1
- Candidate set is limited to biotransformation rules present in RetroRules/KEGG; transformations outside those rule sets cannot be recovered

## Evidence

- [intro] GNN-SOM predicts atom likelihoods: "the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives"
- [intro] PROXIMAL2 identifies reaction centers: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes."
- [methods] Tie resolution by average rank: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [methods] GNN-SOM used for ranking in BAM: "BAM uses GNN-SOM to rank the generated derivatives based on the likelihood of their corresponding site-of-metabolism"
- [readme] README: GNN-SOM requires PyTorch/Geometric/RDKit: [evidence span withheld — anchor status: not_found]

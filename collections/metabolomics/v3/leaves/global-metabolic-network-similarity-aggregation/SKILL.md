---
name: global-metabolic-network-similarity-aggregation
description: Use when two organisms' metabolic networks have been reconstructed with
  MetNet (set or multiset representation) and local pathway similarity values are
  available and must be aggregated into global Psim, PsimW and Csim similarity indexes.
when_to_use_negative:
- The metabolic networks have not yet been reconstructed — this skill presupposes
  MetNet-built networks from KEGG KGML files and the local SimPi/SimSi values
- Genomic or experimental data are the input rather than KEGG organisms — MetNet currently
  works only on KEGG data and does not accept genomic data as input
- Only a single organism's network is available — the indexes are defined for pairwise
  comparison of two organisms selected in KEGG
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: MetNet
  role: Reconstructs the two KEGG organisms' metabolic networks (set or multiset representation)
    and computes the local SimPi/SimSi values that are aggregated into Psim, PsimW
    and Csim
  repo: https://github.com/simeoni-biolab/MetNet
- name: Java
  role: Compiler/runtime used to build and execute the MetNet jar that performs the
    aggregation
- name: KEGG database
  role: Source of the KGML files and reference pathways from which the reaction sets
    and similarity indexes are derived
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:1268712bd3891706260da78666561ccefbfbcaa08fa29d63f37eb7b25cf675d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# global-metabolic-network-similarity-aggregation

## Summary

Aggregates the local structural similarity indexes SimPi (and SimSi) computed pathway-by-pathway between two KEGG organisms into the global indexes Psim (arithmetic mean), PsimW (weighted mean by |Ri∪Ri′|), and the combined index Csim = Σ SimSi·SimPi/n, yielding a quantitative pairwise metabolism comparison.

## When to use

Use when two organisms' metabolic networks have been reconstructed with MetNet (set or multiset representation) and local pathway similarity values are available and must be aggregated into global Psim, PsimW and Csim similarity indexes.

## When NOT to use

- The metabolic networks have not yet been reconstructed — this skill presupposes MetNet-built networks from KEGG KGML files and the local SimPi/SimSi values
- Genomic or experimental data are the input rather than KEGG organisms — MetNet currently works only on KEGG data and does not accept genomic data as input
- Only a single organism's network is available — the indexes are defined for pairwise comparison of two organisms selected in KEGG

## Inputs

- Local pathway similarity values SimPi computed by Jaccard index over reaction sets Ri and Ri′ of each reference pathway in the two organisms
- Local structural similarity values SimSi over the nodes V∪V′ of the two structural-level networks
- Cardinality |Ri∪Ri′| of the union of reaction sets for each pathway (PsimW weights)
- MetNet configuration file pathwayList.txt listing the KEGG reference pathways considered for the comparison

## Outputs

- Global pathway similarity index Psim (arithmetic mean of SimPi)
- Weighted pathway similarity index PsimW (weighted mean of SimPi with weights |Ri∪Ri′|)
- Structure similarity index Ssim (arithmetic mean of SimSi)
- Combined similarity index Csim = Σ SimSi·SimPi/n
- Summary table of the three global indexes Psim, PsimW and Csim

## How to apply

After MetNet reconstructs both networks from KGML files downloaded via KEGG's public APIs, compute the local pathway similarity index SimPi for each reference pathway as the Jaccard index over the pathways' reaction sets, handling the physical-pathway cases where SimPi is forced to 0 or 1. Aggregate the local indexes into Psim as the arithmetic mean over the n KEGG pathways present in either organism. For PsimW, compute the weighted mean of SimPi with weights |Ri∪Ri′|, considering only reaction pathways (not physical pathways), so that pathways with more reactions contribute proportionally more. Separately, aggregate the structural similarity indexes SimSi into Ssim as their arithmetic mean over the |V∪V′| nodes of the structural level. Finally, compute Csim as Σ SimSi·SimPi/n; because both SimSi and SimPi lie in [0,1], their product amplifies differences between organisms, making Csim more discriminating than either index alone. Write Psim, PsimW and Csim to a summary table for downstream clustering against known evolutionary relationships.

## Related tools

- **MetNet** (Reconstructs the two KEGG organisms' metabolic networks (set or multiset representation) and computes the local SimPi/SimSi values that are aggregated into Psim, PsimW and Csim) — https://github.com/simeoni-biolab/MetNet
- **Java** (Compiler/runtime used to build and execute the MetNet jar that performs the aggregation)
- **KEGG database** (Source of the KGML files and reference pathways from which the reaction sets and similarity indexes are derived)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Psim must equal the arithmetic mean of all SimPi values over the n KEGG pathways present in either organism — recompute the mean and compare
- PsimW must equal the weighted mean of SimPi with weights |Ri∪Ri′|, and must exclude physical pathways (only reaction pathways weighted)
- Csim must lie in [0,1] since both SimSi and SimPi lie in [0,1]; a Csim outside [0,1] indicates an implementation error
- For identical organisms (or identical reaction sets) all SimPi are 1, so Psim, PsimW and Csim should all be 1
- The three global indexes Psim, PsimW and Csim must appear in the summary table output for the compared organism pair

## Limitations

- The reconstruction and hence the indexes are based uniquely on the KEGG database, which can be a limit with respect to the represented information
- Physical pathways have degenerate SimPi values (0 or 1) which are handled specially and affect the unweighted Psim
- MetNet does not currently support input of genomic data; extending it would require a reconstruction approach compliant with the modular organization of KEGG pathways

## Evidence

- [abstract] Psim as mean of local SimPi: "They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [readme] MetNet reconstructs and compares two KEGG organisms quantitatively and visually: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [methods] KGML retrieval step precedes index computation: [evidence span withheld — anchor status: not_found]
- [results] Configuration file restricts the pathways considered for the comparison: "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
- [discussion] KEGG-only limitation: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
- [methods] Comparison method choice set or multiset affects the reaction sets used in the indexes: "3. comparison method: "set" or "multiset""

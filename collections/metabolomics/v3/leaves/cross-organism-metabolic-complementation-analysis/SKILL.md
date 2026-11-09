---
name: cross-organism-metabolic-complementation-analysis
description: Use when two KEGG-coded organisms with suspected metabolic interdependence
  (e.g. host–endosymbiont pairs) need their metabolisms reconstructed from KGML data
  and compared pathway-by-pathway to reveal complementation.
when_to_use_negative:
- You need to reconstruct a metabolism from annotated genomic/experimental data rather
  than KEGG — MetNet is based uniquely on the KEGG database and does not currently
  support input of genomic data
- The organism pair is not available under KEGG organism codes, or you need a graphical-visualisation-driven
  comparison rather than the command-line two-organism functional comparison
- The goal is single-organism network reconstruction without a pairwise comparison
  (tools like AutoKEGGRec or KBase address that)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: MetNet
  role: Automatically reconstructs the metabolic networks of the two chosen KEGG organisms
    and computes the functional-level comparison (SimPi, Psim, PsimW) in command-line
    mode
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of KGML pathway files and organism codes; the reconstruction and the
    reference pathways (pathwayList.txt) are entirely KEGG-based
- name: Java
  role: Runtime/compiler for MetNet; the tool is compiled with javac against the bundled
    jar libraries and launched with java -jar MetNet.jar
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:1b6076e008076a971c8303ab74fa2cfd8d5ab5379fa70d5b67567890796853c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-organism-metabolic-complementation-analysis

## Summary

Reconstructs the metabolic networks of two KEGG organisms with MetNet and compares them at the functional level using per-pathway Pi similarity indexes (SimPi) and the global Psim/PsimW indexes, then builds a complementarity inventory to test whether low global similarity reflects complementary metabolisms (e.g. aphid apis vs. symbiont bap).

## When to use

Use when two KEGG-coded organisms with suspected metabolic interdependence (e.g. host–endosymbiont pairs) need their metabolisms reconstructed from KGML data and compared pathway-by-pathway to reveal complementation.

## When NOT to use

- You need to reconstruct a metabolism from annotated genomic/experimental data rather than KEGG — MetNet is based uniquely on the KEGG database and does not currently support input of genomic data
- The organism pair is not available under KEGG organism codes, or you need a graphical-visualisation-driven comparison rather than the command-line two-organism functional comparison
- The goal is single-organism network reconstruction without a pairwise comparison (tools like AutoKEGGRec or KBase address that)

## Inputs

- KEGG organism codes for the two organisms to compare (e.g. apis, bap) via organismList.txt
- pathwayList.txt configuration file listing the KEGG reference pathways to be compared
- KEGG KGML files of each pathway of both organisms, retrieved through the public KEGG APIs
- Choice of comparison method: 'set' or 'multiset'

## Outputs

- Per-pathway SimPi values for every KEGG reference pathway in pathwayList.txt
- Global functional-level indexes Psim and PsimW
- Complementarity inventory: reference pathways whose reactions occur only in one of the two organisms
- complementarity_analysis.csv with per-pathway similarities, Psim/PsimW values and the interpretation

## How to apply

Run MetNet in command-line mode ('java -jar MetNet.jar <org1> <org2> set'), giving the KEGG codes of both organisms (e.g. apis for Acyrthosiphon pisum, bap for Buchnera aphidicola) and the 'set' comparison method so both metabolisms are reconstructed from KEGG KGML files and the two-level comparison is computed. From the functional-level output, compute the Pi pathway similarity index (SimPi) for every reference pathway listed in pathwayList.txt: SimPi = 0 for pathways present in one organism only, SimPi = |Ri ∩ Ri′| / |Ri ∪ Ri′| for reaction pathways present in both, and SimPi = 1 for pathways based on physical mechanisms present in both. Aggregate into the pathway similarity index Psim (arithmetic mean over all pathways present in either organism) and the weighted index PsimW (weighted mean by |Ri ∪ Ri′|), remembering that PsimW ignores pathways dealing with physical transformations. Build the complementarity inventory by listing reference pathways whose reactions occur only in the aphid or only in the symbiont, focusing on pathways absent in one partner but needed to complete the other's metabolic functions. Interpret the pattern of absent/present pathway categories together with the low global similarity values (e.g. 5.02%–14.73% for apis vs. bap) as metabolic complementation, and write per-pathway similarities, Psim/PsimW values, complementarity lists and the interpretation to complementarity_analysis.csv.

## Related tools

- **MetNet** (Automatically reconstructs the metabolic networks of the two chosen KEGG organisms and computes the functional-level comparison (SimPi, Psim, PsimW) in command-line mode) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of KGML pathway files and organism codes; the reconstruction and the reference pathways (pathwayList.txt) are entirely KEGG-based)
- **Java** (Runtime/compiler for MetNet; the tool is compiled with javac against the bundled jar libraries and launched with java -jar MetNet.jar)

## Examples

```
java -jar MetNet.jar apis bap set
```

## Evaluation signals

- Every KEGG reference pathway listed in pathwayList.txt is reported with a SimPi value (0, 1, or the reaction-overlap ratio |Ri ∩ Ri′| / |Ri ∪ Ri′|)
- Computed global indexes Psim and PsimW both fall in [0,1]
- PsimW excludes pathways dealing with physical transformations, while Psim averages over all pathways present in either organism
- The complementarity inventory correctly partitions reference pathways into aphid-only, symbiont-only, and shared categories
- The reported similarity values reproduce the qualitative finding, e.g. very low global similarity (5.02%–14.73%) for apis vs. bap with functional links via Methane metabolism and Glyoxylate and dicarboxylate metabolism

## Limitations

- Reconstruction is limited to the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data and must be significantly extended to reconstruct metabolisms from newly annotated genomes
- The configuration files organismList.txt and pathwayList.txt must be present in the tool's directory; missing them prevents execution

## Evidence

- [abstract] MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually.: "MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks"
- [methods] Our metabolism reconstruction method is based on the KEGG database.: "Our metabolism reconstruction method is based on the KEGG database."
- [methods] 3. comparison method: "set" or "multiset": "3. comparison method: "set" or "multiset""
- [results] whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file): "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
- [readme] To run MetNet as a command line tool, the following three parameters must be specified when starting the execution: "To run MetNet as a command line tool, the following three parameters must be specified when starting the execution"
- [readme] Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory.: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."

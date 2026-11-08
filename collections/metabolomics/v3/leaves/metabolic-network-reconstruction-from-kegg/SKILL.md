---
name: metabolic-network-reconstruction-from-kegg
description: Use when the goal is to reconstruct organism-specific metabolic networks
  from KEGG organisms for downstream comparison (pairwise similarity indexes or multi-organism
  clustering), starting only from KEGG organism codes.
when_to_use_negative:
- When the input is raw genomic or experimental genomic data — MetNet does not currently
  support input of genomic data and would need significant extension to reconstruct
  networks from it
- When metabolic information beyond KEGG is required — the reconstruction is based
  uniquely on the KEGG database, which limits the represented information
- When a single-organism genome-scale model with reaction gaps filled from annotated
  genomes is needed — tools like AutoKEGGRec or the KBase Predictive Biology platform
  address that use case instead
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0092
tools:
- name: MetNet
  role: 'Java tool that performs the automatic reconstruction: retrieves KEGG data,
    parses KGML, and builds the two-level network representation for comparison'
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source database supplying organism organismList, KGML pathway files, and the
    standardized reference-pathway decomposition used by the reconstruction
- name: Java
  role: Compiler/runtime used to build and execute MetNet from source with its lib
    jars (gs-algo-1.3, gs-core-1.3, gs-ui-1.3, guava-19.0, poi-3.9, sax2r2)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:0030b1363d101f5feb0cd7da1ff9fed6c20c95ee5b0cc6c832903cc1f47e67ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# metabolic-network-reconstruction-from-kegg

## Summary

Automatically reconstruct the two-level metabolic network of a KEGG organism (structural level: pathways as nodes connected via shared non-ubiquitous compounds; functional level: each pathway represented by its reactions) by downloading and parsing KGML files through KEGG's public APIs.

## When to use

Use when the goal is to reconstruct organism-specific metabolic networks from KEGG organisms for downstream comparison (pairwise similarity indexes or multi-organism clustering), starting only from KEGG organism codes.

## When NOT to use

- When the input is raw genomic or experimental genomic data — MetNet does not currently support input of genomic data and would need significant extension to reconstruct networks from it
- When metabolic information beyond KEGG is required — the reconstruction is based uniquely on the KEGG database, which limits the represented information
- When a single-organism genome-scale model with reaction gaps filled from annotated genomes is needed — tools like AutoKEGGRec or the KBase Predictive Biology platform address that use case instead

## Inputs

- KEGG organism codes (e.g. sce, zro, cal, afm, abp, fgr, tpf, tre)
- KGML pathway files downloaded via KEGG's public APIs
- MetNet configuration files organismList.txt and pathwayList.txt
- comparison method parameter: 'set' or 'multiset'

## Outputs

- Reconstructed two-level metabolic network per organism (graph of pathways connected via shared non-ubiquitous compounds, plus per-pathway reaction content)
- Per-pathway compound and reaction sets ready for similarity-index computation
- Quantitative and visual comparison of two organism networks

## How to apply

Begin with automatic data retrieval: for each chosen KEGG organism (e.g. sce, zro, cal, afm, abp, fgr, tpf, tre), download the KGML files of each pathway via KEGG's public APIs, then parse each KGML file to extract the compounds and reactions of each pathway — the two entities needed for the two-level representation. Use MetNet (Java), ensuring the configuration files organismList.txt and pathwayList.txt are in the tool's directory, the latter listing the KEGG reference pathways to consider. MetNet supports any pair of KEGG organisms and reconstructs the metabolism in the two-level representation aligned with KEGG's standardized decomposition into reference pathways. For pairwise use, invoke MetNet with two KEGG codes and a comparison method ('set' treats pathways as sets of reactions, 'multiset' keeps multiplicities); for multi-organism studies, reconstruct all organisms and compute pairwise similarity indexes (Ssim, Psim, PsimW, Csim) over the extracted reaction sets. Rationale: reconstruction is fully automatic and based uniquely on KEGG data, avoiding the human intervention required when repositories are incomplete, heterogeneous and incoherent.

## Related tools

- **MetNet** (Java tool that performs the automatic reconstruction: retrieves KEGG data, parses KGML, and builds the two-level network representation for comparison) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source database supplying organism organismList, KGML pathway files, and the standardized reference-pathway decomposition used by the reconstruction)
- **Java** (Compiler/runtime used to build and execute MetNet from source with its lib jars (gs-algo-1.3, gs-core-1.3, gs-ui-1.3, guava-19.0, poi-3.9, sax2r2))

## Examples

```
javac -cp ./lib/gs-algo-1.3.jar:./lib/gs-core-1.3.jar:./lib/gs-ui-1.3.jar:./lib/guava-19.0.jar:./lib/poi-3.9.jar:./lib/sax2r2.jar -d ./ -sourcepath ./src ./src/MetNet/MetNet.java && java -jar MetNet.jar sce cal set
```

## Evaluation signals

- Each organism's network contains only compounds and reactions extracted from that organism's KGML files for the pathways listed in pathwayList.txt
- The structural level connects pathways via shared non-ubiquitous compounds, consistent with KEGG's standardized reference-pathway decomposition
- Running MetNet with the same pair of organisms and comparison method ('set') reproduces identical similarity indexes, confirming deterministic reconstruction
- Downstream sanity check: on the eight Fungi (sce, zro, cal, afm, abp, fgr, tpf, tre), complete-linkage clustering of 1 - Csim distances over all 28 pairs must split Yeasts from Molds at the top level
- Executable jar builds cleanly via 'jar cfm MetNet.jar manifest.txt MetNet/' after compiling with the listed -cp jars

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- No method yet exists in MetNet for reconstructing metabolic networks from experimental genomic data; the tool must be significantly extended to accept genomic input
- MetNet reconstructs and compares a pair of organisms per run; set-level comparisons (e.g. 8 fungi, 28 pairs) require iterating pairwise runs externally
- organismList.txt must be refreshed: MetNet can download the latest list of KEGG organisms on start, otherwise the local list may be stale

## Evidence

- [methods] Automatic KGML-driven reconstruction: [evidence span withheld — anchor status: not_found]
- [methods] Extraction of compounds and reactions: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [abstract] Two-level representation tied to KEGG: "The two-level representation complies with the KEGG database, which decomposes the metabolism of all the different organisms into “reference” pathways in a standardised way."
- [readme] Pairwise invocation contract: "java -jar MetNet.jar hsa ptr set"
- [discussion] Scope limited to KEGG: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
- [discussion] No genomic input support: "the tool MetNet has to be significantly extended to allow the input of genomic data and implement the reconstruction of the corresponding metabolic network"
- [readme] Required configuration files: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."

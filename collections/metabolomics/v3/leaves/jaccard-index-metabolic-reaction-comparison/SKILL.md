---
name: jaccard-index-metabolic-reaction-comparison
description: 'Use when comparing the metabolisms of two organisms selected from the
  KEGG database quantitatively: KGML files are available, reaction sets per pathway
  can be reconstructed, and global similarity indexes (Psim, PsimW, Csim) are the
  desired output.'
when_to_use_negative:
- The input is raw genomic or experimental data rather than KEGG-registered organisms
  — MetNet does not currently support input of genomic data and the reconstruction
  is based uniquely on the KEGG database.
- The pathways of interest are physical/pathway-of-pathways maps rather than reaction
  pathways and no interpretation of degenerate SimPi = 0 or SimPi = 1 cases is planned.
- A single organism's network is needed without a pairwise comparison partner — this
  skill is defined for two-organism (pairwise) comparison.
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: MetNet
  role: Java tool used to automatically reconstruct the two organisms' metabolic networks
    from KEGG and compute the local Jaccard SimPi values and the global indexes Psim,
    PsimW and Csim, either via GUI or command line.
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML pathway files, organism codes and standardized reference
    pathways from which the reaction sets for the Jaccard comparison are extracted.
- name: Java
  role: Compiler/runtime needed to build and execute MetNet from source (javac with
    gs-algo-1.3.jar, gs-core-1.3.jar, gs-ui-1.3.jar, guava-19.0.jar, poi-3.9.jar,
    sax2r2.jar on the classpath).
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:4c86bdf2d7e4cba1ecef41aba80cd687b08f5593cacae40220f50317f727581a
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
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# jaccard-index-metabolic-reaction-comparison

## Summary

A two-level metabolic similarity procedure from the MetNet methodology (PLOS ONE, 2021) in which local pathway similarity is computed as a Jaccard index over the reaction sets of two KEGG organisms' pathways, then aggregated into the global indexes Psim (arithmetic mean), PsimW (weighted mean) and the combined index Csim.

## When to use

Use when comparing the metabolisms of two organisms selected from the KEGG database quantitatively: KGML files are available, reaction sets per pathway can be reconstructed, and global similarity indexes (Psim, PsimW, Csim) are the desired output.

## When NOT to use

- The input is raw genomic or experimental data rather than KEGG-registered organisms — MetNet does not currently support input of genomic data and the reconstruction is based uniquely on the KEGG database.
- The pathways of interest are physical/pathway-of-pathways maps rather than reaction pathways and no interpretation of degenerate SimPi = 0 or SimPi = 1 cases is planned.
- A single organism's network is needed without a pairwise comparison partner — this skill is defined for two-organism (pairwise) comparison.

## Inputs

- KGML files of each pathway of two KEGG organisms, downloaded via KEGG's public APIs
- KEGG organism codes for the two organisms to compare (e.g. 'hsa', 'ptr')
- pathwayList.txt listing the KEGG reference pathways considered for the comparison
- organismList.txt listing the KEGG organisms available
- Choice of comparison method: 'set' or 'multiset' representation of reaction content

## Outputs

- Local pathway similarity indexes SimPi (one Jaccard value per reference pathway)
- Global pathway similarity index Psim (arithmetic mean of SimPi over n pathways)
- Weighted pathway similarity index PsimW (weighted mean of SimPi, weights |Ri∪Ri′|, reaction pathways only)
- Structure similarity index Ssim (arithmetic mean of SimSi over |V∪V′| nodes)
- Combined similarity index Csim = Σ SimSi·SimPi/n
- Summary table of Psim, PsimW and Csim (e.g. a similarity matrix over all compared organism pairs)

## How to apply

First reconstruct both organisms' metabolic networks from their KGML files downloaded through KEGG's public APIs, using MetNet with either the 'set' or 'multiset' comparison method specified on the command line. For each reference pathway, compute the local structural similarity index SimPi as the Jaccard index over the pathways' reaction sets, handling the physical-pathway cases where SimPi degenerates to 0 or 1. Aggregate the local indexes into Psim as the arithmetic mean over the n KEGG pathways present in either organism. For the weighted variant PsimW, compute the weighted mean of SimPi with weights |Ri∪Ri′|, considering only reaction pathways (not physical pathways). The structural level aggregates SimSi into Ssim as the arithmetic mean over the |V∪V′| nodes, and the combined index Csim is computed as Σ SimSi·SimPi/n — this combination amplifies differences because both factors lie in [0,1], so a low value of either one drives Csim down. Write Psim, PsimW and Csim to a summary table for pairwise or set-of-organism comparisons.

## Related tools

- **MetNet** (Java tool used to automatically reconstruct the two organisms' metabolic networks from KEGG and compute the local Jaccard SimPi values and the global indexes Psim, PsimW and Csim, either via GUI or command line.) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML pathway files, organism codes and standardized reference pathways from which the reaction sets for the Jaccard comparison are extracted.)
- **Java** (Compiler/runtime needed to build and execute MetNet from source (javac with gs-algo-1.3.jar, gs-core-1.3.jar, gs-ui-1.3.jar, guava-19.0.jar, poi-3.9.jar, sax2r2.jar on the classpath).)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every local SimPi value and all three global indexes (Psim, PsimW, Csim) must lie in [0,1]; any value outside this range indicates a miscomputed Jaccard union/intersection.
- Psim must equal the arithmetic mean of SimPi over exactly the n KEGG pathways present in either organism; verify the denominator counts pathways of both organisms, not just shared ones.
- PsimW must be a weighted mean with weights |Ri∪Ri′| and must include only reaction pathways, excluding physical-pathway cases where SimPi is 0 or 1 by construction.
- Csim = Σ SimSi·SimPi/n should be ≤ Psim when structural and functional similarities disagree, since the product of two [0,1] factors amplifies differences.
- Running MetNet twice with identical organism codes and the same 'set' (or 'multiset') method must reproduce identical index values, confirming deterministic Jaccard aggregation.

## Limitations

- The reconstruction and comparison are based uniquely on the KEGG database, which can be a limit with respect to the represented information; organisms or reactions absent from KEGG cannot be handled.
- MetNet does not currently accept genomic data as input, so newly sequenced organisms cannot be compared without prior KEGG registration; extending the tool for this would require significant development.
- The multiset vs set choice changes the reaction-set representation and hence the Jaccard values; results are not comparable across different comparison methods.
- Physical pathways must be treated specially (SimPi forced to 0 or 1) and are excluded from PsimW, or the weighted index is distorted.

## Evidence

- [methods] Reconstruction from KGML: [evidence span withheld — anchor status: not_found]
- [methods] Extraction of reactions for the Jaccard sets: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [methods] Choice of set vs multiset representation: "3. comparison method: "set" or "multiset""
- [abstract] Two-level similarity measures with local and global aggregation: "we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [abstract] MetNet scope: two-organism quantitative and visual comparison: "MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [readme] Command-line invocation with organism codes and comparison method: "java -jar MetNet.jar hsa ptr set

will start the execution for the organisms "hsa" (Homo Sapiens) and "prt" (Pan troglodytes) with the comparison method "set"."
- [discussion] KEGG-only reconstruction limitation: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
- [discussion] No genomic-data input support: "the tool MetNet has to be significantly extended to allow the input of genomic data and implement the reconstruction of the corresponding metabolic network"

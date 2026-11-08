---
name: metnet-command-line-execution
description: Use when two KEGG organisms must be automatically reconstructed and compared
  without GUI interaction, e.g. in batch pipelines or reproducible scripts, using
  MetNet's two-level representation.
when_to_use_negative:
- When the input is newly annotated genomic/experimental data rather than a KEGG organism
  — MetNet does not support input of genomic data and would need significant extension
- When an interactive, GUI-guided exploration of organism selection and comparison
  is preferred over a scripted run
- When the organisms of interest are not listed in the organismList.txt KEGG organism
  list
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3407
tools:
- name: MetNet
  role: The tool being executed in command-line mode; performs automatic KEGG-based
    network reconstruction and pairwise comparison
  repo: github.com/simeoni-biolab/MetNet
- name: Java
  role: Runtime and compiler used to build MetNet.jar and execute it via 'java -jar
    MetNet.jar' with parameters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:f20aae53842f64b7b1cf47da899da00d0df52ab929d7e35aaeab41d21caac33b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetNet
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metnet-command-line-execution

## Summary

Executes MetNet in command-line mode by passing the KEGG codes of two organisms and the comparison method ('set' or 'multiset'), enabling scriptable, non-interactive reconstruction and comparison of two metabolic networks.

## When to use

Use when two KEGG organisms must be automatically reconstructed and compared without GUI interaction, e.g. in batch pipelines or reproducible scripts, using MetNet's two-level representation.

## When NOT to use

- When the input is newly annotated genomic/experimental data rather than a KEGG organism — MetNet does not support input of genomic data and would need significant extension
- When an interactive, GUI-guided exploration of organism selection and comparison is preferred over a scripted run
- When the organisms of interest are not listed in the organismList.txt KEGG organism list

## Inputs

- KEGG organism code of the first organism (e.g. 'hsa' for Homo sapiens)
- KEGG organism code of the second organism (e.g. 'ptr' for Pan troglodytes)
- Comparison method parameter: 'set' or 'multiset'
- Configuration files organismList.txt and pathwayList.txt present in the tool's directory

## Outputs

- Automatically reconstructed two-level metabolic networks of the two organisms (structural graph of pathways and functional reaction content per pathway)
- Quantitative and visual comparison results of the two networks, including similarity indexes at both levels

## How to apply

Ensure the configuration files organismList.txt (KEGG organism list) and pathwayList.txt (pathways to consider) are in the tool's directory. Launch MetNet with three command-line parameters in this order: (1) the KEGG code of the first organism, (2) the KEGG code of the second organism, and (3) the comparison method, which must be the literal string 'set' or 'multiset' — this choice governs how pathways are represented when computing similarity indexes. MetNet then automatically retrieves the KGML files of each pathway via KEGG's public APIs, parses them to extract compounds and reactions, reconstructs the two networks, and computes the comparison indexes. For example, running with 'hsa' (Homo sapiens) and 'ptr' (Pan troglodytes) with method 'set' performs the full two-level comparison. The graphical interface version is the alternative when interactive organism selection is preferred.

## Related tools

- **MetNet** (The tool being executed in command-line mode; performs automatic KEGG-based network reconstruction and pairwise comparison) — github.com/simeoni-biolab/MetNet
- **Java** (Runtime and compiler used to build MetNet.jar and execute it via 'java -jar MetNet.jar' with parameters)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- The run completes only if organismList.txt and pathwayList.txt are found in the tool's directory; a missing file aborts execution
- The third parameter must be exactly 'set' or 'multiset'; any other value is invalid for the comparison method
- The KEGG codes supplied as parameters 1 and 2 must exist in the KEGG organism list (e.g. 'hsa' = Homo sapiens, 'ptr' = Pan troglodytes)
- On success, MetNet produces comparison results for the two organisms covering both the structural and functional levels
- Compiled jar file MetNet.jar must be generated via 'jar cfm MetNet.jar manifest.txt MetNet/' before execution

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can limit the represented information
- MetNet does not support input of genomic data; extension is required for reconstruction from newly annotated genomes
- The pathway scope of the comparison is restricted to the KEGG reference pathways listed in the pathwayList.txt configuration file
- Classpath separator differs by OS: semi-colon (;) on Windows, colon (:) on Linux when compiling

## Evidence

- [readme] To run MetNet as a command line tool, the following three parameters must be specified when starting the execution: "To run MetNet as a command line tool, the following three parameters must be specified when starting the execution"
- [readme] java -jar MetNet.jar hsa ptr set: "java -jar MetNet.jar hsa ptr set"
- [readme] Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory.: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
- [methods] download the KGML files of each pathway of the organism through the public KEGG's APIs: [evidence span withheld — anchor status: not_found]
- [methods] 3. comparison method: "set" or "multiset": "3. comparison method: "set" or "multiset""

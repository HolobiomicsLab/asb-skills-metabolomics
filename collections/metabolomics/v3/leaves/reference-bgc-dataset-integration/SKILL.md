---
name: reference-bgc-dataset-integration
description: Use when constructing BGC sequence similarity networks from genome-mining
  output and you need experimentally characterized reference clusters (e.g. MIBiG)
  mixed into the same clustering run to assign functions to novel GCFs.
when_to_use_negative:
- When the goal is only to count or cluster the study's own BGCs without functional
  anchoring — adding references changes family counts and network topology
- When the study BGCs were predicted with an antiSMASH version or parameters incompatible
  with the MIBiG GenBank format expected by BiG-SCAPE
- When comparing against a newer MIBiG release than the one the original analysis
  used (release 1.3, August 2016), as counts will not be comparable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0080
tools:
- name: BiG-SCAPE
  role: Consumes antiSMASH GenBank files and MIBiG reference BGC files as direct inputs
    to build the combined sequence similarity network and assign GCFs/GCCs
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: 'Produces the study-side GenBank BGC outputs (parameters: --minimal) that
    are merged with MIBiG references'
- name: HMMER 3.1b2
  role: Domain detection against Pfam version 31 applied uniformly to both study and
    reference BGCs
- name: hmmalign
  role: Aligns sequences of each predicted domain type during pairwise distance calculation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:ba0294dd6534876eaf22dfbed7a6d08c236ec3d3f71ef96e5e0003766aff197b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# reference-bgc-dataset-integration

## Summary

Augments a study's antiSMASH-predicted BGC set with curated reference biosynthetic gene clusters from the MIBiG database (release 1.3, 1,393 BGCs in this study) before BiG-SCAPE network construction, so that newly identified GCFs can be anchored to biochemically characterized clusters.

## When to use

Use when constructing BGC sequence similarity networks from genome-mining output and you need experimentally characterized reference clusters (e.g. MIBiG) mixed into the same clustering run to assign functions to novel GCFs.

## When NOT to use

- When the goal is only to count or cluster the study's own BGCs without functional anchoring — adding references changes family counts and network topology
- When the study BGCs were predicted with an antiSMASH version or parameters incompatible with the MIBiG GenBank format expected by BiG-SCAPE
- When comparing against a newer MIBiG release than the one the original analysis used (release 1.3, August 2016), as counts will not be comparable

## Inputs

- MIBiG release 1.3 reference BGC GenBank files (1,393 clusters)
- antiSMASH v4 (--minimal) GenBank outputs for the study genomes (74,652 BGCs from 3,080 actinobacterial genomes; Zenodo DOI 10.5281/zenodo.1532752)
- Pfam database version 31 for HMMER 3.1b2 domain detection

## Outputs

- Combined BGC input set (study BGCs + MIBiG references) for BiG-SCAPE
- Sequence similarity network linking novel BGCs to characterized MIBiG references
- GCF/GCC assignments (17,718 GCFs, 801 GCCs) in which reference-bearing families can be functionally annotated

## How to apply

Retrieve the reference BGC GenBank files — here MIBiG release 1.3 (August 2016) supplying 1,393 reference clusters — and combine them with the study's antiSMASH v4 (--minimal) GenBank outputs so both sets enter the same BiG-SCAPE run. BiG-SCAPE accepts antiSMASH GenBank files directly as input, so references need no reformatting; process the combined set with the same Pfam database (version 31 with HMMER 3.1b2) used for the study BGCs so domain detection is comparable. Keep the default analytical parameters (network cutoff distance 0.3, Affinity Propagation GCF assignment, GCC layer at cutoff 0.75) so reference and novel clusters are treated identically and verified counts (17,718 GCFs, 801 GCCs over 74,652 BGCs) remain reproducible. The rationale is that MIBiG provides biochemical reference data against which identified BGCs can be compared, allowing GCFs containing reference clusters to inherit known functions.

## Related tools

- **BiG-SCAPE** (Consumes antiSMASH GenBank files and MIBiG reference BGC files as direct inputs to build the combined sequence similarity network and assign GCFs/GCCs) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Produces the study-side GenBank BGC outputs (parameters: --minimal) that are merged with MIBiG references)
- **HMMER 3.1b2** (Domain detection against Pfam version 31 applied uniformly to both study and reference BGCs)
- **hmmalign** (Aligns sequences of each predicted domain type during pairwise distance calculation)

## Examples

```
python bigscape.py -i bgcs/ -o output/ --pfam-dir Pfam31 --include-mibig  # antiSMASH GenBank outputs + MIBiG 1.3 references, default cutoff 0.3
```

## Evaluation signals

- Total BGC input count equals study BGCs (74,652) plus MIBiG references (1,393)
- Combined run with default parameters reproduces 17,718 GCFs and 801 GCCs, written to gcf_gcc_counts.tsv
- MIBiG reference clusters appear in the resulting network/GCF output (e.g. accessions BGC0001840, BGC0001878, BGC0001841 recoverable)
- Domain detection used the same Pfam version 31 / HMMER 3.1b2 configuration for both reference and study clusters
- GCF calling shows high agreement if alignment method is varied (hmmalign vs Muscle vs MAFFT under BiG-SCAPE --mix)

## Limitations

- Reference version matters: the study used MIBiG release 1.3 (August 2016); using a different release will change counts and network structure
- BiG-SCAPE's interactive report may group genomic scaffolds by ORGANISM identifier, reporting fewer genomes than input (96 of 103 Streptomyces genomes in the article)
- Duplicate ORGANISM identifiers across assembly projects can collapse entries in the output report
- BiG-SCAPE 2.0 changed input validation and reference usage; the wiki advises even experienced BiG-SCAPE 1 users to read the updated documentation

## Evidence

- [readme] BiG-SCAPE input description from README: [evidence span withheld — anchor status: not_found]

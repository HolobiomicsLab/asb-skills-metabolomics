---
name: gcf-genome-based-class-annotation
description: Use when candidate BGCs have been detected by antiSMASH and grouped into
  GCFs by BiG-SCAPE, and genome-based class labels per GCF are needed for downstream
  chemical class matching (e.g. NPClassScore in NPLinker).
when_to_use_negative:
- When no genome data with RefSeq or GenBank identifiers are available, since genomes
  were only downloaded from NCBI in those cases
- When the goal is structure-based classification of MS/MS spectra — that requires
  CANOPUS (via Sirius v4.9.3) or MolNetEnhancer instead
- When a validated class mapping is already available (e.g. from MIBiG 2.0 json entries);
  this skill generates per-GCF labels, not the scoring tables themselves
tools:
- name: antiSMASH
  role: Detects BGCs and provides the BGC (sub)classes that constitute the genome-based
    class labels
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (GCFs) so class annotations can be
    assigned at the GCF level
- name: NPLinker
  role: Platform in which the genome-mining outputs are loaded and GCFs annotated
    with genome-based classes for NPClassScore
  repo: https://github.com/NPLinker/nplinker
- name: MIBiG
  role: Repository of validated BGCs and products used to bridge genome-based and
    structure-based class ontologies
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:541bcfd80ff5cc0e2b50a0a885624a46261a43fd1582231f42bce660c94630dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - MIBiG
  - MolNetEnhancer
  - NPLinker
  - antiSMASH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gcf-genome-based-class-annotation

## Summary

Annotates each candidate gene cluster family (GCF) with genome-based natural product classes derived from antiSMASH BGC (sub)class predictions, aggregated over the BiG-SCAPE clustering, so that these classes can be paired with structure-based classes in the NPClassScore lookup.

## When to use

Use when candidate BGCs have been detected by antiSMASH and grouped into GCFs by BiG-SCAPE, and genome-based class labels per GCF are needed for downstream chemical class matching (e.g. NPClassScore in NPLinker).

## When NOT to use

- When no genome data with RefSeq or GenBank identifiers are available, since genomes were only downloaded from NCBI in those cases
- When the goal is structure-based classification of MS/MS spectra — that requires CANOPUS (via Sirius v4.9.3) or MolNetEnhancer instead
- When a validated class mapping is already available (e.g. from MIBiG 2.0 json entries); this skill generates per-GCF labels, not the scoring tables themselves

## Inputs

- Assembled genomes (RefSeq/GenBank records downloaded from NCBI when identifiers are available)
- antiSMASH output (gbk files; antiSMASH 6 merged with antiSMASH 3 data)
- BiG-SCAPE output grouping BGCs into GCFs (run with cut-off of 0.3)

## Outputs

- GCF objects annotated with genome-based BGC classes and (sub)classes
- Class-annotated GCFs ready for pairing with spectrum classes in NPClassScore scoring tables

## How to apply

Run antiSMASH (the study used antiSMASH 6 on retrieved genomes, merged with prior antiSMASH 3 data from the other 50 strains) to obtain BGC (sub)classes, then run BiG-SCAPE (Docker version, cut-off of 0.3) to group BGCs into gene cluster families (GCFs). Within NPLinker, load the data and attach the antiSMASH-derived (sub)classes to each GCF, so every GCF carries the genome-based class terms used by the MIBiG-derived scoring tables. These genome-based classes form one half of the (genome-based class, structure-based class) pairs looked up after standardised Metcalf scoring (cut-off 2.5, lowered to 2 for the Nocardia dataset). The rationale is that bridging genome-mining ontologies to compound-class ontologies via MIBiG 2.0 entries lets generic class matching complement co-occurrence-based linking without needing large amounts of labelled data.

## Related tools

- **antiSMASH** (Detects BGCs and provides the BGC (sub)classes that constitute the genome-based class labels)
- **BiG-SCAPE** (Groups BGCs into gene cluster families (GCFs) so class annotations can be assigned at the GCF level)
- **NPLinker** (Platform in which the genome-mining outputs are loaded and GCFs annotated with genome-based classes for NPClassScore) — https://github.com/NPLinker/nplinker
- **MIBiG** (Repository of validated BGCs and products used to bridge genome-based and structure-based class ontologies) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data()
```

## Evaluation signals

- Every GCF in the candidate set carries at least one genome-based class derived from BiG-SCAPE/antiSMASH output
- BiG-SCAPE clustering used the paper's cut-off of 0.3; GCF count and membership are consistent with that threshold
- antiSMASH output versions are documented — the study merged antiSMASH 6 output with antiSMASH 3 data from 50 strains, a version inconsistency to check
- Genome-based classes are drawn from the same ontology space used in the MIBiG 2.0-derived scoring tables so (genome class, structure class) pairs can be looked up
- Downstream check: after NPClassScore with cut-off 0.25, ~63% of candidate links should be filtered on average, and ~96% of experimentally validated links should be retained

## Limitations

- Merged antiSMASH 3 and antiSMASH 6 outputs were used, introducing version inconsistency across the 50 strains
- Genome data were only downloaded from NCBI when RefSeq or GenBank identifiers were available; strains lacking these cannot be processed
- The lack of verified genome-metabolome links remains a bottleneck for validating such integrative omics mining methods

## Evidence

- [intro] GCFs grouped by BiG-SCAPE carry genome-based classes: "with the output of BiG-SCAPE, which groups BGCs into gene cluster families (GCFs)"
- [discussion] antiSMASH versions merged for the strain set: "antiSMASH 6 was run and the output was merged with antiSMASH 3 data from the other 50 strains"
- [discussion] Genome downloads depend on NCBI identifiers: "the corresponding genomes were downloaded from NCBI if they had RefSeq or GenBank identifiers"
- [readme] MIBiG class matching repo purpose: [evidence span withheld — anchor status: external_doc]

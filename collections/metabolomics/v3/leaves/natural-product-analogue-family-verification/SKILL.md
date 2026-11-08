---
name: natural-product-analogue-family-verification
description: Use when a manually annotated molecular network family must be validated
  against known reference compounds (e.g. MIBiG accessions) to confirm that discovered
  analogues belong to a specific biosynthetic class.
when_to_use_negative:
- The user only has a single MS/MS spectrum and is not building or validating a molecular
  network family.
- The target family is not tied to a known BGC reference or MIBiG accession, so there
  is no published composition to reconcile against.
- MS/MS data for the strains of interest are not deposited (only available upon request),
  preventing reconstruction of the network.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: Cytoscape
  role: Used to import BiG-SCAPE/MS network edge and node tables, generate a layout
    where molecular families appear as connected components, and export the annotated
    network for verification.
- name: BiG-SCAPE
  role: Provides the network/cutoff framework whose parameters (cosine similarity
    threshold, low-intensity ion filtering) underpin the family edges being verified.
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: CORASON
  role: Companion tool in the metabologenomic workflow that identified the detoxin/rimosamide
    BGC families corresponding to the MS family being verified.
  repo: https://github.com/nselem/corason
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:b18a68d04f51f526c8641f197b39e04b9539bbf70a05e677d34a09fce5c07e3a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - CORASON
  - Cytoscape
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# natural-product-analogue-family-verification

## Summary

Verifies that a molecular network family identified by manual inspection of LC-MS/MS spectra represents a coherent natural-product analogue family (e.g. detoxins/rimosamides) by reconciling node counts against known reference BGC compounds from MIBiG and the article's reported composition.

## When to use

Use when a manually annotated molecular network family must be validated against known reference compounds (e.g. MIBiG accessions) to confirm that discovered analogues belong to a specific biosynthetic class.

## When NOT to use

- The user only has a single MS/MS spectrum and is not building or validating a molecular network family.
- The target family is not tied to a known BGC reference or MIBiG accession, so there is no published composition to reconcile against.
- MS/MS data for the strains of interest are not deposited (only available upon request), preventing reconstruction of the network.

## Inputs

- Deposited LC-MS/MS files for the 363-strain metabolomic study (MassIVE accession MSV000083738)
- MIBiG reference accessions BGC0001840, BGC0001878 and BGC0001841 for known detoxin/rimosamide BGCs
- Cytoscape network layout of molecular families as connected components

## Outputs

- Exported molecular network in GraphML format containing the detoxin/rimosamide family
- detoxin_rimosamide_family_report.csv recording the verified composition (8 known detoxins, 4 known rimosamides, 99 putatively novel analogues)

## How to apply

Build the molecular network from MS/MS scans after removing the 25% of ions with the lowest intensity, retaining edges with cosine similarity >= 0.6. After locating the target family by manually analyzing the network for ions related to known detoxins and rimosamides, cross-check the family's member count against the article's reported composition (110 members: 8 known detoxins, 4 known rimosamides, 99 putatively novel analogues) and against the MIBiG reference accessions BGC0001840, BGC0001878 and BGC0001841. Export the annotated network as a GraphML file and write the verified counts to a family report CSV. Judgment criterion: the exported network exists in graphml format and the family report reproduces the 110-member detoxin/rimosamide family with the 8/4/99 composition breakdown.

## Related tools

- **Cytoscape** (Used to import BiG-SCAPE/MS network edge and node tables, generate a layout where molecular families appear as connected components, and export the annotated network for verification.)
- **BiG-SCAPE** (Provides the network/cutoff framework whose parameters (cosine similarity threshold, low-intensity ion filtering) underpin the family edges being verified.) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **CORASON** (Companion tool in the metabologenomic workflow that identified the detoxin/rimosamide BGC families corresponding to the MS family being verified.) — https://github.com/nselem/corason

## Examples

```
# After building the network, export annotated graph and report:
# cytoscape network -> GraphML export + detoxin_rimosamide_family_report.csv (8 known detoxins, 4 known rimosamides, 99 novel analogues)
```

## Evaluation signals

- The exported network file exists and is valid GraphML.
- The family report CSV contains exactly 110 members for the detoxin/rimosamide family.
- Composition breakdown matches 8 known detoxins, 4 known rimosamides, and 99 putatively novel analogues.
- Identified known compounds map to MIBiG accessions BGC0001840, BGC0001878 and BGC0001841.
- Only ions detected in two or more strains (within 4 ppm and cosine similarity above 0.75) are treated as corroborated members.

## Limitations

- The network was manually analyzed to identify ions related to known detoxins and rimosamides; this is a human-in-the-loop step, not an automated classifier.
- MS/MS data for non-detoxin-producing strains were not deposited and are only available upon request, limiting full reproduction of the family.
- Ion-identity criteria (4 ppm, cosine > 0.75) yield 5,824 ions detected in two or more strains; families outside this strictness may be missed or over-split.
- Known compound assignments rely on BGC reference accessions from MIBiG release 1.3; later MIBiG updates could shift the composition counts.

## Evidence

- [discussion] The family report must reproduce the 110-member detoxin/rimosamide family with the 8/4/99 composition breakdown.: [evidence span withheld — anchor status: not_found]

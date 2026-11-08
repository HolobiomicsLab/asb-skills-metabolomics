---
name: rhamnolipid-fragment-mz-annotation
description: Use when a molecular-networking component of rhamnolipid or homologous
  lipid MS/MS spectra (positive mode, [M+Na]+ fragments) must be interpreted by annotating
  diagnostic fragment peaks and checking which shifted homolog peaks the MMSA multiple
  alignment failed to capture.
when_to_use_negative:
- The spectra are not rhamnolipid (or closely homologous lipid) MS/MS with sodium-adduct
  fragments — the diagnostic m/z values 135.017, 211.130, 357.189, 239.162, 385.219
  will not apply.
- The goal is a single pairwise spectrum comparison (e.g. Metabolomics Spectrum Resolver
  or ModiFinder for one pair) rather than a multiple-spectrum alignment across a whole
  molecular family.
- The data is already processed into a feature table (FBMN feature intensity matrix)
  without the underlying MS/MS peaks needed for fragment-level annotation.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: Web-based tool that performs the global multiple-spectrum alignment whose
    captured peak sets are inspected to identify diagnostic rhamnolipid fragments
    and detect missed C12 shifted peaks.
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkX 3.3
  role: Builds the minimum spanning tree of spectra in the component with edges weighted
    by 1 - pairwise modified cosine similarity, which fixes the node processing order
    for alignment.
- name: GNPS2
  role: Source of the rhamnolipid molecular networking task (task ID + component number)
    whose spectra are pulled for annotation.
- name: metabolomics resolver
  role: Resolves Universal Spectrum Identifiers from public data repositories into
    the specific MS/MS spectra to be annotated.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:a60d7654437d791bebd809ddfbeff6965218490448d88a20d7817971b1e9c8a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rhamnolipid-fragment-mz-annotation

## Summary

Annotates diagnostic rhamnolipid fragment m/z values (e.g. rhamnose m/z 135.017, C10 HAA m/z 211.130, C10+rhamnose m/z 357.189) in MS/MS spectra of a Pseudomonas aeruginosa rhamnolipid molecular family, and checks whether MMSA alignment captures the shifted C12 analogues (m/z 239.162, 385.219).

## When to use

Use when a molecular-networking component of rhamnolipid or homologous lipid MS/MS spectra (positive mode, [M+Na]+ fragments) must be interpreted by annotating diagnostic fragment peaks and checking which shifted homolog peaks the MMSA multiple alignment failed to capture.

## When NOT to use

- The spectra are not rhamnolipid (or closely homologous lipid) MS/MS with sodium-adduct fragments — the diagnostic m/z values 135.017, 211.130, 357.189, 239.162, 385.219 will not apply.
- The goal is a single pairwise spectrum comparison (e.g. Metabolomics Spectrum Resolver or ModiFinder for one pair) rather than a multiple-spectrum alignment across a whole molecular family.
- The data is already processed into a feature table (FBMN feature intensity matrix) without the underlying MS/MS peaks needed for fragment-level annotation.

## Inputs

- GNPS2 molecular networking task ID and component number for a rhamnolipid molecular family
- List of Universal Spectrum Identifiers (USI) referencing specs_ms.mgf scan numbers
- MS/MS spectra in positive mode with sodium-adduct fragments ([M+Na]+)
- MST of spectra in the component built by NetworkX 3.3 with pairwise modified-cosine similarity edge weights

## Outputs

- Annotated list of diagnostic fragment m/z values per spectrum (rhamnose 135.017, C10 HAA 211.130, C10+rhamnose 357.189, and C12 analogues 239.162/385.219 where present)
- MMSA alignment JSON from 'Process and Save JSON' showing captured peak sets
- Consensus-spectra figure (SVG) highlighting which shifted C12 peaks were or were not captured
- Report of missed peaks and phantom peaks in C12 and R(OAc) consensus spectra

## How to apply

Source the rhamnolipid molecular family from a GNPS2 molecular networking task (task ID + component number) or via USIs like mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>, then run MMSA's preprocess (top-6 peaks within ±25 m/z, excluding ±17 m/z of the precursor; square-root intensity transform; normalization), build the MST with NetworkX 3.3 (edges weighted by 1 - modified cosine similarity), topologically sort the spectra, and run the transitive-then-global multiple alignment. In the resulting consensus spectra, annotate the conserved rhamnose fragment m/z 135.017 ([M+H]+ of C5H10O4) present in all spectra, the C10 HAA fragment m/z 211.130 ([M+Na]+ of C10H20O3) and rhamnose-bound-C10 fragment m/z 357.189 ([M+Na]+ of C16H30O7) each in ten spectra, then verify whether the C12 analogues m/z 239.162 (C12H24O3) and m/z 385.219 (C18H34O7) were captured. If they are absent while the greedy alignment also drops smaller alignments (e.g. y2 peaks), report the missed-peak and phantom-peak behavior of MMSA.

## Related tools

- **MMSA** (Web-based tool that performs the global multiple-spectrum alignment whose captured peak sets are inspected to identify diagnostic rhamnolipid fragments and detect missed C12 shifted peaks.) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkX 3.3** (Builds the minimum spanning tree of spectra in the component with edges weighted by 1 - pairwise modified cosine similarity, which fixes the node processing order for alignment.)
- **GNPS2** (Source of the rhamnolipid molecular networking task (task ID + component number) whose spectra are pulled for annotation.)
- **metabolomics resolver** (Resolves Universal Spectrum Identifiers from public data repositories into the specific MS/MS spectra to be annotated.)

## Examples

```
mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>
```

## Evaluation signals

- All spectra in the family show the conserved rhamnose fragment m/z 135.017 ([M+H]+ of C5H10O4).
- Exactly ten spectra show the C10 HAA fragment m/z 211.130 ([M+Na]+ of C10H20O3) and ten show the C10+rhamnose fragment m/z 357.189 ([M+Na]+ of C16H30O7).
- The alignment JSON's captured peak sets contain neither m/z 239.162 nor 385.219, matching the paper's reported C12 miss.
- The exported consensus-spectra SVG shows phantom peaks in the C12 and R(OAc) consensus spectra consistent with the greedy alignment's behavior.
- Smaller alignments (e.g. y2 peaks in the albicidin family analogue) are also absent, confirming the greedy alignment does not fully capture all peak sets.

## Limitations

- MMSA's greedy alignment fails to capture shifted peaks for C12-containing rhamnolipids (m/z 239.162 and 385.219), so spectra with C12 HAA cores remain unaligned with their C10 analogues.
- Smaller peak alignments such as y2 peaks in the albicidin family are not fully captured in the multiple alignment.
- Phantom peaks can appear in C12 and R(OAc) consensus spectra as artifacts of the greedy alignment, so consensus peaks should not be treated as directly diagnostic.
- The MMSA website is licensed under an Academic Software License © 2024 UCR, restricting use to academic/nonprofit research.

## Evidence

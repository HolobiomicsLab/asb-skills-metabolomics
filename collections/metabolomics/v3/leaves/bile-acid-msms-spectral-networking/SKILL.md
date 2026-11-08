---
name: bile-acid-msms-spectral-networking
description: Use when analyzing untargeted metabolomics MS/MS spectra of bile acids
  (or related lipid families) where conventional cosine-threshold molecular networking
  fragments metabolite families and novel conjugates may be hidden as missed connections.
when_to_use_negative:
- When the metabolite family is already well ordered by a conventional cosine-similarity
  molecular network and no hidden connections are expected — MCN's value is rescuing
  relationships fractured by arbitrary thresholds.
- When the input is already a feature table or abundance matrix without paired MS/MS
  spectra — MCN operates on spectral similarity structure, not on pre-computed feature
  tables.
- When the analysis targets non-spectral data (e.g., genomics or imaging) outside
  the untargeted metabolomics scope of the paper.
edam_operation: http://edamontology.org/operation_3669
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molecular_communities
  role: Implements molecular community networking (MCN), which optimizes connectivity
    for each metabolite to generate molecular communities over bile acid MS/MS spectra;
    used to recover hidden dipeptide-conjugated bile acid connections.
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:e7745d5da89003954cf073dcce11252845279695cb815b34a72dd56a621ffe68
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  edam_operation: http://edamontology.org/operation_3669
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bile-acid-msms-spectral-networking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Builds molecular networks over bile acid MS/MS spectra using molecular community networking (MCN), which optimizes connectivity per metabolite instead of applying arbitrary cosine-similarity thresholds, to rescue hidden spectral relationships and surface novel dipeptide-conjugated bile acids and their microbial associations.

## When to use

Use when analyzing untargeted metabolomics MS/MS spectra of bile acids (or related lipid families) where conventional cosine-threshold molecular networking fragments metabolite families and novel conjugates may be hidden as missed connections.

## When NOT to use

- When the metabolite family is already well ordered by a conventional cosine-similarity molecular network and no hidden connections are expected — MCN's value is rescuing relationships fractured by arbitrary thresholds.
- When the input is already a feature table or abundance matrix without paired MS/MS spectra — MCN operates on spectral similarity structure, not on pre-computed feature tables.
- When the analysis targets non-spectral data (e.g., genomics or imaging) outside the untargeted metabolomics scope of the paper.

## Inputs

- Bile acid MS/MS spectra dataset from the paper (Figures 2, S15-16)
- MCN code from the Alexander0/molecular_communities repository
- Supplemental materials (for biological associations such as Bifidobacterium breve in infant stool)

## Outputs

- Molecular community assignments over the bile acid spectra
- Identifications of novel dipeptide-conjugated bile acids rescued by MCN
- mcn_bile_acid_communities.csv containing community assignments, dipeptide-conjugated bile acid identifications, and the B. breve association

## How to apply

Obtain the bile acid dataset (Figures 2, S15-16) and the MCN code from the Alexander0/molecular_communities repository. First build the conventional MS/MS cosine-similarity molecular network on the bile acid spectra to establish the baseline network representation. Then run molecular community networking, which optimizes connectivity for each metabolite rather than imposing an arbitrary similarity threshold, to generate molecular communities over the bile acid dataset. Inspect the recovered communities to identify novel dipeptide-conjugated bile acids that MCN rescues as 'hidden' connections absent from the thresholded baseline network. Cross-reference the Supplemental materials for biological associations — e.g., one newly discovered dipeptide-conjugated bile acid detected in infant stool is associated with Bifidobacterium breve and other unclassified Bifidobacterium species. Record community assignments, molecule identifications, and microbial associations together in an output table such as mcn_bile_acid_communities.csv.

## Related tools

- **molecular_communities** (Implements molecular community networking (MCN), which optimizes connectivity for each metabolite to generate molecular communities over bile acid MS/MS spectra; used to recover hidden dipeptide-conjugated bile acid connections.) — https://github.com/Alexander0/molecular_communities

## Evaluation signals

- A conventional cosine-similarity molecular network on the same bile acid spectra reproduces the paper's baseline network (Figure 2) before MCN is applied.
- MCN output assigns each bile acid metabolite to a molecular community via per-metabolite connectivity optimization rather than a fixed similarity threshold.
- Novel dipeptide-conjugated bile acids recovered by MCN correspond to 'hidden' connections absent from the thresholded baseline network.
- The association of the infant-stool dipeptide-conjugated bile acid with Bifidobacterium breve and other unclassified Bifidobacterium species matches the Supplemental materials.
- Final table mcn_bile_acid_communities.csv contains community assignments, dipeptide-conjugated bile acid identifications, and the B. breve association for the analyzed dataset.

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families, so the baseline network is a starting point rather than the final answer; MCN is needed to recover lost relationships.
- Biological associations such as the Bifidobacterium breve link are reported in the Supplemental materials rather than the main text, and the provided repository README is effectively empty, offering no additional usage documentation.
- Demonstrated specifically on bile acid MS/MS data (Figures 2, S15-16) and dipeptide-conjugated bile acid discovery; transfer to other molecule classes is implied but not evidenced in the provided context.

## Evidence

- [abstract] Conventional networking fractures bile acid families via thresholds; MCN rescues hidden connections: "using arbitrary similarity thresholds that can fracture natural metabolite families"
- [abstract] MCN definition: optimizes connectivity per metabolite, capturing hidden metabolite connections: "We present molecular community networking (MCN), that optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise "hidden" metabolite connections."
- [abstract] Demonstrated application: discovery of novel dipeptide-conjugated bile acids: "Using MCN, we demonstrate the discovery of novel dipeptide-conjugated bile acids."
- [supplementary] Task-card finding: infant-stool bile acid associated with Bifidobacterium breve per Supplemental materials: [evidence span withheld — anchor status: not_found]

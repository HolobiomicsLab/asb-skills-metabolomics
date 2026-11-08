---
name: kegg-compound-identifier-mapping
description: Use when extracting or anchoring layer-1 metabolic-reaction-network (KMRN)
  subnetworks in MetDNA2Vis and you must specify centric metabolites by KEGG compound
  IDs (C00082) or extended unknown IDs (KeggExd000923) with is_unknown_annotation
  = TRUE.
when_to_use_negative:
- Your metabolites have no KEGG mapping and no MetDNA2 extended annotation (no 'C*'
  or 'KeggExd*' ID) — the centric_met anchor cannot be constructed
- You are doing de novo molecular networking in GNPS from MS/MS similarity alone,
  without a knowledge-based reaction-network layer
- Input is a raw feature table that has not yet passed MetDNA2 known/unknown annotation
  (reformatTable1 output missing)
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MetDNA2Vis
  role: R package providing retrieveSubNetwork1/2/3 and mergeSubnetwork, which consume
    KEGG compound and extended unknown identifiers as centric_met anchors for subnetwork
    extraction and merging
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab package loaded alongside MetDNA2Vis in the visualization/reproduction
    workflow
  repo: github.com/ZhuMetLab/SpectraTools
- name: MetDNA2_Web
  role: Web server generating the KGMN results (known and unknown metabolite annotations
    indexed by KEGG-style IDs) that MetDNA2Vis visualizes; hosts the detailed tutorial
  repo: github.com/ZhuMetLab/MetDNA2_Web
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:b62b863fe189f713e7322c0262925f62fd1e8926da044d9d61916393f412cb20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - MetDNA2_Web
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# kegg-compound-identifier-mapping

## Summary

Maps metabolite features in the MetDNA2Vis/KGMN framework to KEGG compound identifiers (KEGG CIDs such as 'C00082') and extended KEGG unknown identifiers ('KeggExd*' entries such as 'KeggExd000923'), so that metabolic-reaction-network subnetworks can be anchored on centric metabolites spanning both known and unknown annotations.

## When to use

Use when extracting or anchoring layer-1 metabolic-reaction-network (KMRN) subnetworks in MetDNA2Vis and you must specify centric metabolites by KEGG compound IDs (C00082) or extended unknown IDs (KeggExd000923) with is_unknown_annotation = TRUE.

## When NOT to use

- Your metabolites have no KEGG mapping and no MetDNA2 extended annotation (no 'C*' or 'KeggExd*' ID) — the centric_met anchor cannot be constructed
- You are doing de novo molecular networking in GNPS from MS/MS similarity alone, without a knowledge-based reaction-network layer
- Input is a raw feature table that has not yet passed MetDNA2 known/unknown annotation (reformatTable1 output missing)

## Inputs

- Centric metabolite identifier vector mixing KEGG compound IDs (e.g., 'C00082') and extended unknown IDs (e.g., 'KeggExd000923')
- MetDNA2 Vis output folder (Demo_MetDNA2_NIST_urine_pos/06_visualization) containing reconstructed layer-1 KMRN network
- Annotation table produced by reformatTable1() if known/unknown status must be resolved
- Logical flag is_unknown_annotation indicating whether centric metabolites are unknowns

## Outputs

- Layer-1 metabolic-reaction-network subnetwork exported to folder_output (e.g., 'M182T541_M262T526')
- Identifier-resolved annotation table linking feature peaks (M153T555, M285T555, M182T541, M262T526) to KEGG compound/extended IDs
- Consolidated subnetwork after mergeSubnetwork with no conflicting base-peak duplicates

## How to apply

Identify the centric metabolite(s) for the subnetwork of interest and express them in the dual-identifier scheme used by MetDNA2Vis: known metabolites carry KEGG compound IDs (e.g., 'C00082' for xanthosine-family chemistry), while putative unknowns carry 'KeggExd*' extended identifiers. Pass the identifier vector as the `centric_met` argument to retrieveSubNetwork1 together with `is_unknown_annotation = TRUE` so empirical rules are checked per subnetwork, and keep `folder_output` identical across retrieveSubNetwork1/2/3 and mergeSubnetwork so the three layers consolidate into one subnetwork (e.g., 'M182T541_M262T526'). The rationale is that KGMN's knowledge-based metabolic reaction network is indexed by KEGG-style compound identifiers, so consistent identifier use across layers is what allows peak annotations to merge and conflicting subnetwork assignments (e.g., M153T555 vs. M285T555/xanthosine) to be resolved by maximal self-consistency.

## Related tools

- **MetDNA2Vis** (R package providing retrieveSubNetwork1/2/3 and mergeSubnetwork, which consume KEGG compound and extended unknown identifiers as centric_met anchors for subnetwork extraction and merging) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab package loaded alongside MetDNA2Vis in the visualization/reproduction workflow) — github.com/ZhuMetLab/SpectraTools
- **MetDNA2_Web** (Web server generating the KGMN results (known and unknown metabolite annotations indexed by KEGG-style IDs) that MetDNA2Vis visualizes; hosts the detailed tutorial) — github.com/ZhuMetLab/MetDNA2_Web

## Examples

```
retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923'), is_unknown_annotation = TRUE, folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- The centric_met vector resolves without error: each entry is either a valid KEGG compound ID ('C*' pattern) or a valid MetDNA2 extended unknown ID ('KeggExd*' pattern)
- The folder_output is identical across retrieveSubNetwork1, retrieveSubNetwork2, retrieveSubNetwork3, and mergeSubnetwork calls (e.g., 'M182T541_M262T526')
- After mergeSubnetwork, the merged subnetwork exists in the shared output folder and all three layers have resolved into one consolidated subnetwork with no remaining conflicting base-peak duplicates
- Conflicting subnetwork assignments (e.g., M153T555/5-ureido-4-imidazole vs. M285T555/xanthosine) collapse to the annotation maximizing subnetwork self-consistency, verifiable against chemical-standard confirmation (xanthosine as in-source fragment)
- is_unknown_annotation = TRUE yields per-subnetwork empirical-rule checks consistent with the KGMN pipeline for unknown peaks

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), so identifier mapping alone does not resolve isomeric ambiguity
- Unknown-metabolite identifiers ('KeggExd*') derive from in silico biotransformation reaction sets (mainly BioTransformer human metabolic reactions), limiting transfer to non-model organisms
- Validation of unknown annotations remains difficult because unknowns are absent from spectral databases and lack chemical standards

## Evidence

- [methods] retrieveSubNetwork1 centric_met dual-ID anchor: "retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923'), 
  is_unknown_annotation = TRUE"
- [readme] Keep folder_output consistent across subnetwork layers: "# Note: the folder_output should keep same among different layer subnetworks"
- [abstract] KGMN three-layer integration rationale: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [readme] Install command for identifier-mapping tooling: "devtools::install_github("ZhuMetLab/SpectraTools")
devtools::install_github("ZhuMetLab/MetDNA2Vis")"
- [results] Conflict resolution by subnetwork self-consistency: [evidence span withheld — anchor status: not_found]

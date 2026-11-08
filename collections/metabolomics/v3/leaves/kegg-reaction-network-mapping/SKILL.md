---
name: kegg-reaction-network-mapping
description: Use when an untargeted LC-MS/MS peak table (MS1 m/z, RT, MS2 spectra)
  has seed 'known' peaks matched against a standard spectral library and you must
  annotate reaction-paired unknown peaks via a KEGG-style metabolic reaction network.
when_to_use_negative:
- Applying to non-model organisms or environmental/exposomics profiles, since organism-specific
  reaction databases are not yet incorporated
- When structurally similar isomers of unknowns must be discriminated (e.g., isocitrate
  glucuronidation product isomers), which KGMN cannot resolve effectively
- When the input is only a list of already annotated known peaks with no unknown peaks
  requiring reaction-paired extension
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: Provides reconstructNetwork1(is_unknown_annotation = TRUE), reformatTable1(),
    reconstructNetwork2/3, retrieveSubNetwork1/2/3, and mergeSubnetwork to build the
    reaction-network layer and merge it into the final annotation network
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Installed alongside MetDNA2Vis to support MS2 spectral handling during seed
    matching and network construction
  repo: ZhuMetLab/SpectraTools
- name: igraph
  role: CRAN/Bioconductor dependency used for graph representation of the multilayer
    metabolic networks
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:3f06cad533204edfaa37f844322e4bdf6217b54d8242eeb83d880e6401a13ef5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - SpectraTools
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# kegg-reaction-network-mapping

## Summary

Builds the knowledge-based metabolic reaction network (KMRN) layer of the KGMN approach, extending library-matched 'known' metabolites across KEGG-style reaction pairs to annotate reaction-paired 'unknown' peaks in untargeted LC-MS/MS metabolomics data.

## When to use

Use when an untargeted LC-MS/MS peak table (MS1 m/z, RT, MS2 spectra) has seed 'known' peaks matched against a standard spectral library and you must annotate reaction-paired unknown peaks via a KEGG-style metabolic reaction network.

## When NOT to use

- Applying to non-model organisms or environmental/exposomics profiles, since organism-specific reaction databases are not yet incorporated
- When structurally similar isomers of unknowns must be discriminated (e.g., isocitrate glucuronidation product isomers), which KGMN cannot resolve effectively
- When the input is only a list of already annotated known peaks with no unknown peaks requiring reaction-paired extension

## Inputs

- Peak table with MS1 m/z, retention time, and MS2 spectra from LC-MS/MS untargeted metabolomics data (per ionization mode)
- Standard spectral library (MetDNA2Vis 'cpd_lib') for seeding known metabolites
- KEGG-based reaction/network data packaged in MetDNA2Vis (e.g., 'network_46std', 'cpd_emrn', 'cpd_46stdExd')
- Demo/project export folder (e.g., 'Demo_MetDNA2_NIST_urine_pos/06_visualization/') with MetDNA2 result files

## Outputs

- Layer-1 knowledge-based metabolic reaction network (KMRN) subnetwork linking known seeds to reaction-paired unknown peaks
- Merged three-layer annotation network (via retrieveSubNetwork1/2/3 and mergeSubnetwork)
- Peak annotation table with known/unknown assignments per ionization mode
- Per-mode count summary (e.g., kgmn_nist_urine_stats.csv: seeds, total peaks, knowns, unknowns)

## How to apply

Seed the annotation by matching each peak's MS1 m/z and MS2 spectrum against the packaged standard spectral library (cpd_lib from MetDNA2Vis) to identify known metabolites (e.g., 39 knowns in positive and 41 in negative mode for the 46std_mix human liver S9 incubation). Install MetDNA2Vis and SpectraTools from GitHub with their CRAN/Bioconductor dependencies (dplyr, tidyr, readr, CHNOSZ, igraph, ggraph, tidygraph), set the working directory to the exported demo folder (e.g., 'Demo_MetDNA2_NIST_urine_pos/06_visualization/'), and run reconstructNetwork1(is_unknown_annotation = TRUE) to extend knowns to reaction-paired unknown peaks over the knowledge-based metabolic reaction network. Prepare the annotation table with reformatTable1() so the reaction layer can feed the knowledge-guided MS2 similarity layer (reconstructNetwork2(annotation_table = annotation_table)) and the global peak correlation layer (reconstructNetwork3()); merge subnetworks via retrieveSubNetwork1/2/3 and mergeSubnetwork to assign final annotations. Rationale: reaction pairing propagates annotation from trusted seed metabolites to biologically plausible unknowns, and the merged three-layer network lifted known-peak identification accuracy to >95% while annotating ~100–300 putative unknowns per dataset.

## Related tools

- **MetDNA2Vis** (Provides reconstructNetwork1(is_unknown_annotation = TRUE), reformatTable1(), reconstructNetwork2/3, retrieveSubNetwork1/2/3, and mergeSubnetwork to build the reaction-network layer and merge it into the final annotation network) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Installed alongside MetDNA2Vis to support MS2 spectral handling during seed matching and network construction) — ZhuMetLab/SpectraTools
- **igraph** (CRAN/Bioconductor dependency used for graph representation of the multilayer metabolic networks)

## Examples

```
reconstructNetwork1(is_unknown_annotation = TRUE)
```

## Evaluation signals

- 46std_mix + human liver S9 incubation: positive mode must report exactly 39 known + 69 unknown peaks of 700 MS1 peaks; negative mode exactly 41 known + 106 unknown of 741 peaks
- Output CSV must contain annotated entries for the IMP-derived cascade products inosine and inosine 5'-sulfate (confirmed with authentic standards)
- NIST human urine positive mode: 173 seed metabolites expanding to 927 annotated peaks = 634 knowns + 293 unknowns; negative mode: 161 seeds, 1283 peaks = 652 knowns + 631 unknowns
- More than 80% of annotated unknown metabolites should be corroborated with in silico MS/MS tools
- reconstructNetwork1 must run with is_unknown_annotation = TRUE and its output table (via reformatTable1) must feed reconstructNetwork2 without re-matching knowns

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; the BioTransformer reaction sets mainly originate from human metabolic reactions
- Validation of unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards

## Evidence

- [abstract] The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network.: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [readme] reconstructNetwork1(is_unknown_annotation = TRUE): "reconstructNetwork1(is_unknown_annotation = TRUE)"
- [intro] iMet combined reactant pairs from KEGG and MS/MS spectral similarity to train a classifier model for metabolite annotation47: "iMet combined reactant pairs from KEGG and MS/MS spectral similarity to train a classifier model for metabolite annotation47"
- [discussion] the remaining ~40% of reaction pairs have low MS/MS similarity scores even they have high structural similarity: "the remaining ~40% of reaction pairs have low MS/MS similarity scores even they have high structural similarity"
- [methods] reconstructNetwork2(annotation_table = annotation_table): "reconstructNetwork2(annotation_table = annotation_table)"

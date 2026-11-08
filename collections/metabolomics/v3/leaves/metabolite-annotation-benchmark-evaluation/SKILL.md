---
name: metabolite-annotation-benchmark-evaluation
description: Use when evaluating or benchmarking LC-MS untargeted metabolomics peak-annotation
  methods against a curated ground-truth feature set, or when reproducing the KGMN
  vs MetDNA1 accuracy/coverage comparison on the 242-metabolite urine benchmark.
when_to_use_negative:
- Input is raw MS data without a curated ground-truth annotation table — this skill
  evaluates annotation accuracy, it does not generate annotations from spectra alone
- Samples from non-model organisms or environmental/exposomics profiles — the article
  cautions KGMN relies on predefined BioTransformer reaction sets mainly from human
  metabolic reactions
- When discriminating structurally similar isomers of unknown metabolites is required
  — KGMN cannot effectively distinguish such isomers (e.g., isocitrate glucuronidation
  product isomers)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA2Vis
  role: R package used to reconstruct the three KGMN network layers, export and merge
    subnetworks, and reproduce the benchmark visualization
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab package installed alongside MetDNA2Vis for spectral handling
    during benchmark reproduction
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: igraph
  role: Graph dependency required for building and manipulating the knowledge-based
    metabolic reaction network and similarity networks
- name: tidygraph
  role: Tidy graph data-structure dependency used with ggraph for network layer construction
    and visualization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:1326e79cd2a296b5cb72995a58a7b1b4b86b12509bd7c3a148f9ddb16c75e612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  - dplyr
  - ggraph
  - igraph
  - tidygraph
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-annotation-benchmark-evaluation

## Summary

Reproduces and quantifies peak-annotation accuracy of the knowledge-guided multilayer network (KGMN) against MetDNA1 on the manually curated benchmark of 242 metabolites and 3451 metabolic features, using the ZhuMetLab R packages SpectraTools and MetDNA2Vis.

## When to use

Use when evaluating or benchmarking LC-MS untargeted metabolomics peak-annotation methods against a curated ground-truth feature set, or when reproducing the KGMN vs MetDNA1 accuracy/coverage comparison on the 242-metabolite urine benchmark.

## When NOT to use

- Input is raw MS data without a curated ground-truth annotation table — this skill evaluates annotation accuracy, it does not generate annotations from spectra alone
- Samples from non-model organisms or environmental/exposomics profiles — the article cautions KGMN relies on predefined BioTransformer reaction sets mainly from human metabolic reactions
- When discriminating structurally similar isomers of unknown metabolites is required — KGMN cannot effectively distinguish such isomers (e.g., isocitrate glucuronidation product isomers)

## Inputs

- Curated benchmark peak table from the 242-metabolite / 3451-feature NIST urine positive-mode dataset (Demo_MetDNA2_NIST_urine_pos/06_visualization)
- MetDNA1 baseline annotation results (correct/error counts per peak)
- Installed R environment with MetDNA2Vis, SpectraTools, and CRAN/Bioconductor dependencies

## Outputs

- Per-layer and merged subnetwork files in folder_output 'M182T541_M262T526' for peaks M182T541 and M262T526
- Correct-annotation rate, error-annotation rate, and annotation coverage metrics (KGMN vs MetDNA1)

## How to apply

Install SpectraTools and MetDNA2Vis from GitHub (devtools) after satisfying CRAN/Bioconductor dependencies (dplyr, tidyr, readr, CHNOSZ, igraph, magrittr, ggplot2, ggraph, tidygraph via BiocManager). Set the working directory to Demo_MetDNA2_NIST_urine_pos/06_visualization, then rebuild the three KGMN layers with reconstructNetwork1(is_unknown_annotation = TRUE), reformatTable1() followed by reconstructNetwork2(annotation_table = ...), and reconstructNetwork3(). To visualize the propagation from knowns to unknowns, export per-layer subnetworks anchored on peaks M182T541 and M262T526 via retrieveSubNetwork1 (centric_met = c('C00082', 'KeggExd000923')), retrieveSubNetwork2, and retrieveSubNetwork3 (base_adducts = c('[M+H]+', '[M+H]+')), all writing to the shared folder_output 'M182T541_M262T526', then mergeSubnetwork() them. The benchmark comparison is judged by counting correct vs error annotations (KGMN: 3325 correct / 98.5% vs 49 errors / 1.5%; MetDNA1: 1767 correct / 72.2% vs 682 errors) and annotation coverage (71.0% to 97.8%).

## Related tools

- **MetDNA2Vis** (R package used to reconstruct the three KGMN network layers, export and merge subnetworks, and reproduce the benchmark visualization) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab package installed alongside MetDNA2Vis for spectral handling during benchmark reproduction) — https://github.com/ZhuMetLab/SpectraTools
- **igraph** (Graph dependency required for building and manipulating the knowledge-based metabolic reaction network and similarity networks)
- **tidygraph** (Tidy graph data-structure dependency used with ggraph for network layer construction and visualization)

## Examples

```
library(MetDNA2Vis); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- Merged subnetwork files exist inside folder_output 'M182T541_M262T526' and match the demo reference workflow output
- KGMN correct-annotation count reaches 3325 of 3451 features (98.5%) with 49 errors (1.5%), versus MetDNA1's 1767 correct (72.2%) and 682 errors
- Annotation coverage increases from 71.0% (MetDNA1) to 97.8% (KGMN) on the 242-metabolite benchmark
- All three per-layer subnetwork exports share the identical folder_output 'M182T541_M262T526' so mergeSubnetwork succeeds
- Installation check: all required CRAN/Bioconductor packages (dplyr, tidyr, readr, CHNOSZ, igraph, magrittr, ggplot2, ggraph, tidygraph) report installed before devtools::install_github calls

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites, e.g. isocitrate glucuronidation product isomers
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions
- Caution needed when applying KGMN to non-model organisms and environmental/exposomics samples; organism-specific databases are not yet incorporated
- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- Validation of unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards

## Evidence

- [results] Benchmark correct/error rates: [evidence span withheld — anchor status: not_found]
- [abstract] Three-layer network architecture: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [readme] Installation and dependency steps: "required_pkgs <- c("dplyr","tidyr","readr","CHNOSZ","igraph",
  "magrittr","ggplot2","ggraph","tidygraph")"
- [readme] Network reconstruction workflow: "reconstructNetwork1(is_unknown_annotation = TRUE)

# construct network 2
annotation_table <- reformatTable1()
reconstructNetwork2(annotation_table = annotation_table)"
- [readme] Shared output folder requirement for subnetworks: "Note: the folder_output should keep same among different layer subnetworks"

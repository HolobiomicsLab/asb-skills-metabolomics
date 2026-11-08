---
name: sample-wise-metabolomics-workflow-reproducibility
description: Use when a task requires regenerating or validating the ENPKG knowledge
  graph from deposited LC-MS/MS DDA data (e.g. MassIVE MSV000087728), or when reproducing
  sample-level feature detection, annotation, and RDF integration figures.
when_to_use_negative:
- When the input is already an aligned feature quantification table or processed feature
  list rather than raw LC-MS2 DDA files — the workflow expects raw vendor/mzML data
  organized sample-wise.
- When the goal is batch-level feature alignment across samples via retention time;
  ENPKG is explicitly designed to avoid RT-based alignment, processing each sample
  individually.
- When working with non-LC-MS/MS DDA metabolomics data or datasets whose folder structure
  differs from the expected sample-wise MassIVE organization.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ENPKG
  role: Full workflow orchestrating sample-wise processing and RDF knowledge graph
    construction
  repo: https://github.com/enpkg/enpkg_full
- name: MZmine
  role: Feature detection and export (centroid detection, ADAP chromatogram builder,
    wavelets deconvolution, feature filtering)
  repo: http://mzmine.github.io/
- name: matchms
  role: Molecular networking and ISDB-LOTUS spectral matching with modified cosine
    scoring
- name: SIRIUS
  role: CSI:FingerID and CANOPUS structural annotation of exported MS2 spectra
  repo: https://boecker-lab.github.io/docs.sirius.github.io/install/
- name: MassIVE
  role: Repository hosting the raw MS2 data files for each sample
- name: GraphDB
  role: Load and explore the generated .ttl knowledge graph files
  repo: https://graphdb.ontotext.com/download/
- name: RDF
  role: Output format for standardized per-sample results in the knowledge graph
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:4c6d58cc84bca8da116ac9a15c179576653ffbea4a8c3b325d02be26cbfd7554
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_full
  tools:
  - ENPKG
  - GraphDB
  - MZmine
  - MassIVE
  - RDF
  - SIRIUS
  - matchms
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Sample-wise metabolomics workflow reproducibility

## Summary

Reproduces the ENPKG sample-centric LC-HRMS2 workflow end-to-end, processing each sample individually (taxonomic resolution, MZmine feature detection, ISDB-LOTUS spectral matching, SIRIUS/CANOPUS annotation) and rebuilding the RDF knowledge graph from the 1,600 plant extracts dataset (MSV000087728) instead of relying on batch-wise feature alignment.

## When to use

Use when a task requires regenerating or validating the ENPKG knowledge graph from deposited LC-MS/MS DDA data (e.g. MassIVE MSV000087728), or when reproducing sample-level feature detection, annotation, and RDF integration figures.

## When NOT to use

- When the input is already an aligned feature quantification table or processed feature list rather than raw LC-MS2 DDA files — the workflow expects raw vendor/mzML data organized sample-wise.
- When the goal is batch-level feature alignment across samples via retention time; ENPKG is explicitly designed to avoid RT-based alignment, processing each sample individually.
- When working with non-LC-MS/MS DDA metabolomics data or datasets whose folder structure differs from the expected sample-wise MassIVE organization.

## Inputs

- Raw LC-HRMS2 DDA data from MassIVE record MSV000087728 (1,600 plant extracts, PI and/or NI modes)
- Minimal per-sample metadata file containing the sample's originating taxon
- params/user.yml configuration (record_id/record_name of the dataset)
- .env with PATH_TO_SIRIUS and SIRIUS_USERNAME/SIRIUS_PASSWORD

## Outputs

- Per-sample MZmine feature lists with associated MS2 scans (filtered: RT 0.5–8.0 min, MS2-linked features only)
- Taxonomic annotations (Open Tree of Life ottID, Wikidata IDs) per sample
- ISDB-LOTUS spectral annotations reranked by taxonomic distance, and SIRIUS/CSI:FingerID + CANOPUS candidate annotations
- RDF (.ttl) files per sample, loadable into GraphDB as a unified knowledge graph

## How to apply

Clone enpkg/enpkg_full and edit params/user.yml to point at the MassIVE record (MSV000087728, 1,600 plant extracts), then configure .env with PATH_TO_SIRIUS and SIRIUS credentials and run 'sh workflow/00_workflow_all.sh'. The fixed pipeline organizes raw LC-MS2 data sample-wise (01_enpkg_data_organization), resolves each organism binomial against Open Tree of Life v3.5 and Wikidata (02_enpkg_taxo_enhancer), and runs MZmine with a centroid mass detector (noise level 1.0E4), ADAP chromatogram builder (min group size 5, min highest intensity 5.0E5, 12 ppm m/z tolerance), wavelets (ADAP) deconvolution (S/N 10, peak duration 0.02–0.5 min), isotope peaks grouper (12 ppm), and feature filtering keeping only features with an associated MS2 scan and RT between 0.5 and 8.0 min. Next, 03_enpkg_mn_isdb_isdb_taxo runs molecular networking and ISDB-LOTUS matching in Python with matchms (modified cosine score cutoff 0.7 for edges, max_links 10, top_n 15; min cosine 0.2 with 6 minimal matching fragments; parent/fragment m/z tolerance 0.01) with candidates reranked by taxonomic distance and network annotation propagation, and 04_enpkg_sirius_canopus submits spectra to SIRIUS/CSI:FingerID and CANOPUS (ppm tolerance 10, max m/z 800, 10 candidates; adducts [M+H]+, [M+NH4]+, [M+K]+, [M+Na]+ in PI and [M−H]−, [M+K-2H]−, [M+Na-2H]− in NI). Finally 06_enpkg_graph_builder formats per-sample results as RDF (.ttl); each sample is processed individually precisely to avoid RT-based feature alignment across batches, enabling iterative addition of samples.

## Related tools

- **ENPKG** (Full workflow orchestrating sample-wise processing and RDF knowledge graph construction) — https://github.com/enpkg/enpkg_full
- **MZmine** (Feature detection and export (centroid detection, ADAP chromatogram builder, wavelets deconvolution, feature filtering)) — http://mzmine.github.io/
- **matchms** (Molecular networking and ISDB-LOTUS spectral matching with modified cosine scoring)
- **SIRIUS** (CSI:FingerID and CANOPUS structural annotation of exported MS2 spectra) — https://boecker-lab.github.io/docs.sirius.github.io/install/
- **MassIVE** (Repository hosting the raw MS2 data files for each sample)
- **GraphDB** (Load and explore the generated .ttl knowledge graph files) — https://graphdb.ontotext.com/download/
- **RDF** (Output format for standardized per-sample results in the knowledge graph)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && cd enpkg_full && uv sync && cp .env.example .env && set -a && source .env && set +a && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Regenerated graph reproduces the reported feature counts: 788,623 features in PI mode and 364,967 in NI mode.
- GraphDB contains the expected statement scale (over 161 million statements as reported at publication time) and is consistent with the reference graph at https://enpkg.commons-lab.org/graphdb/.
- Every kept feature has an associated MS2 scan and a retention time between 0.5 and 8.0 min (feature filtering module output).
- Each sample's organism resolves to an Open Tree of Life ottID (taxonomy v3.5) and a Wikidata ID via the taxo_enhancer step.
- On the bundled test dataset (Zenodo 10018590), the full workflow completes in roughly 10 minutes without errors in 00_workflow_all.sh.

## Limitations

- Feature-alignment steps based on retention time in classical workflows hinder comparison of samples from different batches or instrumental setups; this skill's sample-wise approach sidesteps but does not quantify alignment errors.
- No statistical metrics (e.g., FDR, p-values, effect sizes) are reported for the compound annotations produced, and no benchmark against existing workflows exists.
- The emikg successor package warns it 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset'.
- Requires a SIRIUS account and local SIRIUS installation (PATH_TO_SIRIUS, SIRIUS_USERNAME, SIRIUS_PASSWORD in .env).

## Evidence

- [results] Sample-centric processing rationale: "788,623 and 364,967 features in positive ionization (PI) and negative
ionization (NI) modes, respectively"
- [methods] Taxonomic resolution step: "The scripts used for this taxonomic resolution are available in https://github.com/enpkg/enpkg_full/tree/main/02_enpkg_taxo_enhancer."
- [readme] Workflow launch command: "From the root of the repository, run:

```bash
sh workflow/00_workflow_all.sh
```

On the previous test dataset, this should take about 10 minutes to run."
- [readme] Graph exploration: "You can use GraphDB to explore the generated Knowledge Graph. To do so, you will need to install GraphDB (https://graphdb.ontotext.com/download/) and import the generated .ttl files."
- [abstract] Sample-level integration: "the experimental data is processed at the sample level, matched with external identifiers where possible, semantically enriched, and integrated into a unified knowledge graph."
- [readme] Sample-wise design: [evidence span withheld — anchor status: external_doc]

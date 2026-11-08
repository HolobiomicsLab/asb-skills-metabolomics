---
name: natural-extract-novelty-prioritization
description: Decision support for selecting which extracts from an untargeted metabolomics
  set to send for phytochemical isolation when the goal is maximizing novelty of discovered
  natural products.
when_to_use_negative:
- The input is a targeted assay or an already-annotated feature table without untargeted
  MS2 data — Inventa's components rely on unannotated feature ratios and spectral
  fingerprints.
- The sample set lacks GNPS-format metadata (species/organism part headers cleaned
  via the Open Tree of Life), so taxon-level literature and class components cannot
  be computed.
- No MZmine 2/MZmine 3 quantification table is available, in which case even the minimum
  Feature Component cannot be run.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: Inventa
  role: Computes the four novelty components and the combined prioritization rank
    of extracts.
  repo: https://github.com/luigiquiros/inventa
- name: MZmine2
  role: Produces the feature quantitative table (Peak area/height, row m/z, row retention
    time) consumed as the quantification input.
- name: MZmine3
  role: Alternative source of the feature quantitative table accepted directly by
    Inventa.
- name: GNPS
  role: Provides the standard feature-based molecular networking job and metadata
    format used as minimum input.
- name: SIRIUS
  role: Run over the sample set to generate the project space whose CANOPUS classes
    feed the Class Component.
- name: CANOPUS
  role: Predicts chemical classes; unreported classes trigger nonzero Class Component
    (CC) values.
- name: timaR
  role: Supplies taxonomically weighted in silico annotation results consumed as tima_results_filename.
  repo: https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- name: MEMO
  role: Vectorizes MS2 peaks and neutral losses into sample fingerprints used for
    the Similarity Component outlier detection.
  repo: https://github.com/mandelbrot-project/memo
- name: canopus_treemap
  role: Recomputes the Sirius project space into the CANOPUS NPC summary file required
    for the Class Component.
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: scikit-bio
  role: Dependency to install from conda-forge if environment creation fails.
- name: Open Tree of Life
  role: Used to clean ATTRIBUTE_Species names to currently recognized taxa before
    scoring.
- name: LOTUS
  role: Natural-products database queried for compounds reported per taxon to compute
    the Literature Component.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:b1de21134c69a4702ffcd2a312cda045e6c3c87b0a33c3b0c4083580f80a3169
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - CANOPUS
  - GNPS
  - Inventa
  - LOTUS
  - MEMO
  - MZmine2
  - MZmine3
  - Open Tree of Life
  - SIRIUS
  - canopus_treemap
  - scikit-bio
  - timaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 30
---

# natural-extract-novelty-prioritization

## Summary

Prioritize natural extracts for structural novelty using Inventa's combined rank of four components (Feature, Literature, Class, Similarity) computed from MZmine quantification tables, GNPS annotations, SIRIUS/CANOPUS chemical taxonomy, and MEMO MS2 spectral fingerprints.

## When to use

Decision support for selecting which extracts from an untargeted metabolomics set to send for phytochemical isolation when the goal is maximizing novelty of discovered natural products.

## When NOT to use

- The input is a targeted assay or an already-annotated feature table without untargeted MS2 data — Inventa's components rely on unannotated feature ratios and spectral fingerprints.
- The sample set lacks GNPS-format metadata (species/organism part headers cleaned via the Open Tree of Life), so taxon-level literature and class components cannot be computed.
- No MZmine 2/MZmine 3 quantification table is available, in which case even the minimum Feature Component cannot be run.

## Inputs

- GNPS feature-based molecular networking job output
- GNPS-format metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- MZmine 2 or MZmine 3 feature quantitative table ('Peak area' or 'Peak height', 'row m/z', 'row retention time'; Ion Identity state must be specified)
- Optional: timaR-weighted in silico annotation file (tima_results_filename)
- Optional: Sirius CANOPUS recomputed NPC summary file (canopus_npc_summary_filename)
- Optional: MEMO vectorized dissimilarity matrix (vectorized_data_filename)

## Outputs

- TAB-separated (.tsv) results file with per-extract FC, LC, CC, SC component scores and the final prioritization rank of the samples

## How to apply

Prepare the minimum inputs — a GNPS feature-based molecular networking job, a GNPS-format metadata table (with ATTRIBUTE_Species cleaned to currently recognized names via the Open Tree of Life, and ATTRIBUTE_Organe), and a MZmine 2 or MZmine 3 quantification table exporting only 'Peak area' (or 'Peak height'), 'row m/z', and 'row retention time' columns — to compute the Feature Component, the ratio of specific non-annotated features over total features per extract. Optionally add timaR-weighted in silico annotations (ISDB/SIRIUS), the CANOPUS NPC summary recomputed with canopus_treemap (from canopus import Canopus; C = Canopus(sirius='sirius_projectspace')), and a MEMO vectorized dissimilarity matrix to enable the Literature, Class, and Similarity components. Compute the Literature Component by subtracting from 1 the ratios of compounds reported per taxon (species/genus/family, e.g. from LOTUS) over user-defined maxima (max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500, weights ws = wg = wf = 1); the Class Component rewards chemical classes unreported for the species and genus (CC = 1 when new to both, CCs and CCg contributing 0.5 each), and the Similarity Component flags spectrally dissimilar outliers via MEMO-based outlier detection. Sum the four components, modulate them with user-defined weights (wn), and export the per-sample scores plus the final rank to a .tsv results file; the highest-ranked extracts are candidates for follow-up isolation, as was done for Pristimera indica roots (LC = 0.87), which yielded thirteen new dihydro-β-agarofuran sesquiterpenes.

## Related tools

- **Inventa** (Computes the four novelty components and the combined prioritization rank of extracts.) — https://github.com/luigiquiros/inventa
- **MZmine2** (Produces the feature quantitative table (Peak area/height, row m/z, row retention time) consumed as the quantification input.)
- **MZmine3** (Alternative source of the feature quantitative table accepted directly by Inventa.)
- **GNPS** (Provides the standard feature-based molecular networking job and metadata format used as minimum input.)
- **SIRIUS** (Run over the sample set to generate the project space whose CANOPUS classes feed the Class Component.)
- **CANOPUS** (Predicts chemical classes; unreported classes trigger nonzero Class Component (CC) values.)
- **timaR** (Supplies taxonomically weighted in silico annotation results consumed as tima_results_filename.) — https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- **MEMO** (Vectorizes MS2 peaks and neutral losses into sample fingerprints used for the Similarity Component outlier detection.) — https://github.com/mandelbrot-project/memo
- **canopus_treemap** (Recomputes the Sirius project space into the CANOPUS NPC summary file required for the Class Component.) — https://github.com/kaibioinfo/canopus_treemap
- **scikit-bio** (Dependency to install from conda-forge if environment creation fails.)
- **Open Tree of Life** (Used to clean ATTRIBUTE_Species names to currently recognized taxa before scoring.)
- **LOTUS** (Natural-products database queried for compounds reported per taxon to compute the Literature Component.)

## Examples

```
conda env create -n inventa -f inventa/environment.yml; conda activate inventa; python src/inventa.py
```

## Evaluation signals

- Each of the four components (FC, LC, CC, SC) falls within its defined 0–1 range; e.g. LC = 1 means no reported compounds for the taxon, FC = 0.6 means 60% of features are specific and non-annotated.
- The output .tsv contains one row per extract with FC, LC, CC, SC and the final summed rank, and samples are ordered by the combined (user-weighted) score.
- The Literature Component for a known case reproduces the article value — Pristimera indica roots with 2 species-, 8 genus-, and 6,064 family-level reports yields LC = 0.87.
- Metadata table headers exactly match the mandatory GNPS fields ATTRIBUTE_Species and ATTRIBUTE_Organe, and the quant table contains only 'Peak area' (or 'Peak height'), 'row m/z', 'row retention time' columns.
- Conda environment builds with the pinned inventa/environment.yml versions; 'Unknown: number' columns at the end of the quant table are handled automatically by the script.

## Limitations

- Binder instances shut down after 15 min of inactivity — parameters and results must be saved locally.
- If 'Peak Height' is preferred over 'Peak Area', the function quant_table() in src/inventa.py must be edited; only one of the two columns is considered at a time.
- Environment creation can fail; installing scikit-bio from conda-forge beforehand is recommended, and submodules must be pulled with git pull --recurse-submodules.
- The LC component depends on external literature coverage (LOTUS) and user-defined maxima (max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500); poorly studied taxa inflate apparent novelty.
- The Class Component requires an extra recomputation step of the Sirius project space via canopus_treemap, without which the CANOPUS/NPC summary file cannot be generated.

## Evidence

- [abstract] Inventa calculates multiple scores that estimate the structural novelty potential of the natural extracts.: "Inventa calculates multiple scores that estimate the structural novelty potential of the natural extracts."
- [readme] The Literature Component (LC) is a score based on the number of compounds reported in the literature for the taxon of a given extract.: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract."
- [readme] The Feature Component (FC) is a ratio of the number of specific non-annotated features over the total number of features of each extract.: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [readme] This will run at least the Feature component (FC).: "This will run at least the Feature component (FC)."
- [readme] The rank comes from the addition of four individual components:: "The rank comes from the addition of four individual components:"
- [readme] The output consists of .tsv file with all the information generated along with the final rank of the samples.: [evidence span withheld — anchor status: external_doc]
- [readme] Create a new conda environment to avoid clashes: conda env create -n inventa -f inventa/environment.yml: "Create a new conda environment to avoid clashes:

conda env create -n inventa -f inventa/environment.yml"

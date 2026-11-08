---
name: hpc-parallel-bioinformatics-workflow-execution
description: Use when a BGC-clustering pipeline must scale to ~10^6 clusters from
  hundreds of thousands of genomes/MAGs within days on a multi-core server, requiring
  a non-pairwise (near-linear) approach instead of O(n^2) pairwise comparison.
when_to_use_negative:
- When the dataset is small enough that pairwise network comparison with BiG-SCAPE
  is feasible — BiG-SLiCE is a non-pairwise Euclidean-space approach and does not
  produce pairwise similarity networks
- When BGCs come from non-antiSMASH tools (ClusterFinder, DeepBGC) or have manually
  refined borders without first converting them to antiSMASH-format GenBank via the
  provided converter script
- When the compute budget cannot provide a multi-core server with ~128 GB RAM for
  a multi-day run
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: 'Core clustering engine: feature extraction, --complete-only GCF model building
    at T thresholds, and --query mode membership assignment of fragmented BGCs'
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH v5.1.1
  role: Upstream BGC prediction producing the GenBank input for the 209,206 genomes,
    run with taxon-specific options and --minlength 5000
  repo: https://antismash.secondarymetabolites.org/
- name: BiG-SCAPE
  role: Pairwise-comparison baseline whose estimated 37,000-hour runtime on a 36-core
    CPU motivates the non-pairwise HPC strategy
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: pyHMMER
  role: Python-native HMM scanning backend in BiG-SLiCE 2.0 that speeds up domain
    detection and enables pip-only installation
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:548b033ed6e68c731bee5409d93d8ede322c8ee4b4750e012c289a403ca80503
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SCAPE
  - BiG-SLiCE
  - antiSMASH v5.1.1
  - pyHMMER
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# hpc-parallel-bioinformatics-workflow-execution

## Summary

Executes a full-scale genome-mining workflow — antiSMASH BGC prediction feeding BiG-SLiCE GCF clustering — on a 36-core CPU server with 128 GB RAM, processing 1,225,071 BGCs from 209,206 genomes/MAGs in ~240 hours (~10 days), over 150x faster than the estimated BiG-SCAPE runtime.

## When to use

Use when a BGC-clustering pipeline must scale to ~10^6 clusters from hundreds of thousands of genomes/MAGs within days on a multi-core server, requiring a non-pairwise (near-linear) approach instead of O(n^2) pairwise comparison.

## When NOT to use

- When the dataset is small enough that pairwise network comparison with BiG-SCAPE is feasible — BiG-SLiCE is a non-pairwise Euclidean-space approach and does not produce pairwise similarity networks
- When BGCs come from non-antiSMASH tools (ClusterFinder, DeepBGC) or have manually refined borders without first converting them to antiSMASH-format GenBank via the provided converter script
- When the compute budget cannot provide a multi-core server with ~128 GB RAM for a multi-day run

## Inputs

- antiSMASH-predicted BGC GenBank files for 209,206 genomes and MAGs (19,169 complete/chromosome-level and 162,352 draft bacterial RefSeq genomes, 5,939 fungal and 1,162 archaeal NCBI GenBank genomes)
- BiG-SLiCE input folder structured per the project wiki (Input folder specification)
- BiG-SLiCE HMM database (download_bigslice_hmmdb; bigslice-models release)
- MIBiG v2.0 reference set (1,910 entries) for downstream GCF-model distance benchmarking

## Outputs

- SQLite3 database in the BiG-SLiCE output folder containing all processed BGCs (1,225,071), GCF centroid models, and cluster/membership assignments
- GCF models built from 802,287 non-fragmented BGCs at each threshold T in {300, 600, 900, 1,200, 1,500}
- Query-mode ranked lists of matching GCFs/BGCs (n_ranks=1) for fragmented BGCs
- Flask-based visualization web-app served from the output folder (pip install -r requirements.txt; bash start_server.sh)

## How to apply

Install BiG-SLiCE via conda/mamba, fetch the HMM database with download_bigslice_hmmdb, and verify the version banner with 'bigslice --version'. Prepare the BiG-SLiCE input folder from the antiSMASH-predicted BGC GenBank files (run antiSMASH v5.1.1 with taxon-specific options and --minlength 5000 to filter contigs <5,000 bp). Run the clustering phase 'bigslice -i <input_folder> <output_folder>' with --complete-only so only the 802,287 (65%) non-fragmented BGCs build GCF centroid models, sweeping threshold values T = {300, 600, 900, 1,200, 1,500}; the first full run (T = 300) carries the whole workflow load and finished in ~240 hours. Then query the full input back against the GCF models in --query mode with --n_ranks 1 so the 35% fragmented BGCs are mapped onto GCFs via membership values d. All processed input data, GCF models, and cluster assignments are stored in the SQLite3 database inside the output folder.

## Related tools

- **BiG-SLiCE** (Core clustering engine: feature extraction, --complete-only GCF model building at T thresholds, and --query mode membership assignment of fragmented BGCs) — https://github.com/medema-group/bigslice
- **antiSMASH v5.1.1** (Upstream BGC prediction producing the GenBank input for the 209,206 genomes, run with taxon-specific options and --minlength 5000) — https://antismash.secondarymetabolites.org/
- **BiG-SCAPE** (Pairwise-comparison baseline whose estimated 37,000-hour runtime on a 36-core CPU motivates the non-pairwise HPC strategy) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **pyHMMER** (Python-native HMM scanning backend in BiG-SLiCE 2.0 that speeds up domain detection and enables pip-only installation) — https://github.com/althonos/pyhmmer

## Examples

```
bigslice -i <input_folder> <output_folder> --complete-only
```

## Evaluation signals

- The populated SQLite3 datastore contains exactly 1,225,071 BGCs from 209,206 genomes and MAGs, matching Table 1
- The full workflow (feature extraction through membership assignment) completes in ~240 hours (within 10 days) on a 36-core CPU server with 128 GB RAM
- The clustering phase with --complete-only consumed exactly 802,287 non-fragmented BGCs (65% of the 1,225,071 input)
- Sweeping T = {300, 600, 900, 1,200, 1,500} yields a strictly decreasing number of GCF models (e.g., 29,955 GCFs at T = 900)
- 'bigslice --version' prints a version banner with a matching HMM database build (e.g., bigslice-models-2022-11-30) before the run

## Limitations

- Pairwise BGC comparison tools have quadratic O(n^2) complexity and cannot scale to this corpus (BiG-SCAPE estimated at 37,000 hours on a 36-core CPU), so this skill requires the non-pairwise BiG-SLiCE design
- Fragmented BGCs (35% of input) cannot contribute to centroid models and rely on the --query membership assignment step
- As of BiG-SLiCE 2.0, the pre-processed result on ~1.2M microbial BGCs is only available for v1; per the README, 'there is currently no pre-processed result for BiG-SLiCE v2'
- Resource envelope is fixed by the published run: 36-core CPU server with 128 GB RAM over ~10 days

## Evidence

- [abstract] full-scale run duration and hardware: "We used BiG-SLiCE to analyze 1,225,071 BGCs collected from 209,206 publicly available microbial genomes and metagenome-assembled genomes within 10 days on a typical 36-core CPU server."
- [results] complete-only filter scope: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [results] threshold sweep: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [intro] motivating runtime comparison: "even the currently fastest tool (a tool that we previously developed, BiG-SCAPE [22]) will require an estimated 37,000 hours of runtime on a 36-core CPU"
- [readme] HPC installation guidance: "Install **BiG-SLiCE** on your local machine or HPC. The recommended method is via conda/mamba:"

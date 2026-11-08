---
name: genome-pipeline-execution
description: Use when raw bacterial genome sequences (e.g. FASTA) must be converted
  into predicted BGC regions with product-class and activity annotations, and no Pfam
  domain TSV is already available.
when_to_use_negative:
- Input is already a Pfam domain TSV with scores — use downstream thresholding/region
  detection directly instead of re-running Prodigal and hmmscan
- The goal is rule-based product-type annotation of known cluster architectures rather
  than deep-learning BGC detection — antiSMASH default settings are the appropriate
  baseline
- Target is a small manually curated set where only validation against known annotations
  is needed and no model download or genome-wide detection is required
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0196
tools:
- name: DeepBGC
  role: 'Primary pipeline: BiLSTM detector scores Pfam domains, post-processing merges
    calls into BGC regions, Random Forest classifiers assign product class and activity'
  repo: https://github.com/Merck/deepbgc
- name: Prodigal
  role: Protein-coding gene prediction on input genomes when no annotation is provided,
    feeding genes into Pfam domain detection
  repo: https://github.com/hyattpd/Prodigal
- name: HMMER
  role: hmmscan annotates predicted genes with Pfam domain compositions used as detector
    input
- name: antiSMASH
  role: Rule-based baseline BGC predictor run with default settings for genome-mining
    count and overlap comparisons
- name: ClusterFinder
  role: HMM-based baseline BGC predictor run with default lenient settings for comparison
- name: miniforge
  role: Conda installer used to provision the Python environment for DeepBGC and its
    dependencies
  repo: https://github.com/conda-forge/miniforge
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:0006ff6971ba0b184d26f2d07592cf6b0718400fddd8d041c0faf0e3534cf233
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Merck/deepbgc
  repo_url: https://github.com/Merck/deepbgc
  tools:
  - ClusterFinder
  - DeepBGC
  - HMMER
  - Prodigal
  - antiSMASH
  - miniforge
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genome-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute an end-to-end bacterial genome-mining pipeline in which DeepBGC (BiLSTM + pfam2vec) detects and classifies biosynthetic gene clusters, with Prodigal gene calling and HMMER hmmscan Pfam annotation run automatically when genomes lack precomputed domain calls.

## When to use

Use when raw bacterial genome sequences (e.g. FASTA) must be converted into predicted BGC regions with product-class and activity annotations, and no Pfam domain TSV is already available.

## When NOT to use

- Input is already a Pfam domain TSV with scores — use downstream thresholding/region detection directly instead of re-running Prodigal and hmmscan
- The goal is rule-based product-type annotation of known cluster architectures rather than deep-learning BGC detection — antiSMASH default settings are the appropriate baseline
- Target is a small manually curated set where only validation against known annotations is needed and no model download or genome-wide detection is required

## Inputs

- Bacterial (or fungal) genome nucleotide sequences in FASTA format
- Trained DeepBGC detector and classification models plus Pfam database (fetched via `deepbgc download`)
- Prodigal binary on PATH for gene prediction
- Optional precomputed Pfam TSV annotations (bypassing Prodigal/hmmscan)

## Outputs

- Per-genome Pfam domain annotation tables with DeepBGC detector scores
- Predicted BGC regions with detector scores and coordinates
- Random Forest product-class and molecular-activity classifications per BGC
- antiSMASH-compatible JSON for uploading DeepBGC regions to antiSMASH visualization
- Aggregated BGC count and cross-tool overlap tables (e.g. figure5A_bgc_overlap.csv, bgc_precision_comparison.csv)

## How to apply

First provision dependencies: install DeepBGC via conda/pip, then run `deepbgc download` to fetch the trained BiLSTM detector model, the Random Forest classifier models, and the Pfam database, and verify with `deepbgc info` that all dependencies and models are present. Ensure the Prodigal `prodigal` binary is on PATH, since DeepBGC delegates protein-coding gene prediction to it for unannotated sequences. Then run `deepbgc pipeline <genome.fa>` per genome: the pipeline runs Prodigal gene calling, HMMER hmmscan against Pfam, per-domain detector scoring (calibrated at cutoffs such as the stringent 2-10% FPR and lenient 80% TPR operating points), region post-processing/merging into discrete BGCs, and Random Forest prediction of product class and chemical activity. For large comparative runs (e.g. 3376 RefSeq genomes), run the same pipeline plus baselines (antiSMASH default rule-based, ClusterFinder default lenient HMM) per genome, then aggregate per-genome BGC counts and pairwise sequence-overlap fractions. Validate by checking that thresholded precision matches reported operating points (e.g. DeepBGC 34% vs ClusterFinder 26% BGC-level precision at 10% FPR) and that counts/overlaps reproduce the reference figures.

## Related tools

- **DeepBGC** (Primary pipeline: BiLSTM detector scores Pfam domains, post-processing merges calls into BGC regions, Random Forest classifiers assign product class and activity) — https://github.com/Merck/deepbgc
- **Prodigal** (Protein-coding gene prediction on input genomes when no annotation is provided, feeding genes into Pfam domain detection) — https://github.com/hyattpd/Prodigal
- **HMMER** (hmmscan annotates predicted genes with Pfam domain compositions used as detector input)
- **antiSMASH** (Rule-based baseline BGC predictor run with default settings for genome-mining count and overlap comparisons)
- **ClusterFinder** (HMM-based baseline BGC predictor run with default lenient settings for comparison)
- **miniforge** (Conda installer used to provision the Python environment for DeepBGC and its dependencies) — https://github.com/conda-forge/miniforge

## Examples

```
deepbgc download && deepbgc info && deepbgc pipeline mySequence.fa
```

## Evaluation signals

- BGC-level precision at the 10% FPR threshold reproduces DeepBGC 34% vs ClusterFinder 26%, and at the 80% TPR threshold reproduces DeepBGC 44% vs ClusterFinder 9% on the nine manually annotated genomes
- `deepbgc info` lists all downloaded models, the Pfam database, and dependencies before any pipeline run
- Across 3376 RefSeq genomes, per-tool BGC counts should fall near DeepBGC 10926, antiSMASH 13865, and ClusterFinder 62491, with ~90% DeepBGC-ClusterFinder and ~39% DeepBGC-antiSMASH sequence overlap
- Each predicted BGC region carries non-empty Random Forest product-class and activity predictions and a detector score
- Prodigal gene calls (GFF3/protein FASTA) are produced for every genome lacking pre-existing annotation before domain scoring

## Limitations

- Detector models were trained on BGC databases heavily biased towards natural-product 'workhorses' such as Streptomyces, limiting detection in poorly characterized bacteria
- BGC class recovery is threshold-dependent: some well-studied classes are recovered regardless of threshold while others vary, and some classes were mis-identified in the S. coelicolor reference genome example
- The majority of BGCs of novel product classes detected by DeepBGC could not be confidently assigned to a class

## Evidence

- [readme] Prodigal output formats: [evidence span withheld — anchor status: external_doc]

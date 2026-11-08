---
name: miclip-m6a-site-integration
description: Use when aggregating m6A modification sites from miCLIP-seq data into
  per-omics-level counts alongside SNP, CNV, SV, mutated-gene, RNA-seq, and ncRNA
  panels for an intestinal-disease multi-omics database or statistics report.
when_to_use_negative:
- The task is to derive RNA expression profiles or differential expression from RNA-seq
  rather than to aggregate m6A modification-site counts — GutUDB's epitranscriptomics
  level is only the miCLIP-seq site tally, not an expression workflow.
- The pipeline already folds m6A sites into a generic 'transcriptomics' profile count
  — GutUDB's convention requires a dedicated epitranscriptomics entry separate from
  the 7.9 million RNA-seq profiles.
edam_operation: http://edamontology.org/operation_3804
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Used to implement the aggregation functions that read GutUDB EnrichedIndex
    counts, encode the miCLIP-seq-derived m6A harmonization rule as a constant, and
    emit the combined per-level statistics JSON.
- name: GutUDB
  role: Source of the EnrichedIndex per-level counts and the 1908 miCLIP-seq m6A sites
    that are aggregated into the statistics JSON.
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:68a8146c7a1ac79751d7d0bc23e0dc980818c735018019bc5409446f7d1d857d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - Python
  edam_operation: http://edamontology.org/operation_3804
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# miclip-m6a-site-integration

## Summary

Integrates RNA N6-methyladenosine (m6A) modification sites derived from miCLIP-seq experiments into a per-level multi-omics statistics structure, following GutUDB's bulk-epitranscriptomics harmonization convention.

## When to use

Use when aggregating m6A modification sites from miCLIP-seq data into per-omics-level counts alongside SNP, CNV, SV, mutated-gene, RNA-seq, and ncRNA panels for an intestinal-disease multi-omics database or statistics report.

## When NOT to use

- The task is to derive RNA expression profiles or differential expression from RNA-seq rather than to aggregate m6A modification-site counts — GutUDB's epitranscriptomics level is only the miCLIP-seq site tally, not an expression workflow.
- The pipeline already folds m6A sites into a generic 'transcriptomics' profile count — GutUDB's convention requires a dedicated epitranscriptomics entry separate from the 7.9 million RNA-seq profiles.

## Inputs

- GutUDB EnrichedIndex counts (per-omics-level statistics, including the 1908 m6A sites reported from miCLIP-seq)
- miCLIP-seq m6A site annotations for intestinal-disease samples
- Python module-level constants file encoding bulk-omics harmonization rules

## Outputs

- gutudb_statistics.json — a single JSON object containing an entry for each of the eight bulk omics levels plus the four single-cell/spatial modalities, with the epitranscriptomics level reporting the m6A site count (1908)
- Per-level statistic records combining SNPs (72248), CNVs (54131), SVs (1097), mutated genes (92888), RNA-seq profiles (7.9M), ncRNAs (62 circRNAs, 182 lncRNAs, 58 microRNAs), m6A sites (1908), and microbe–metabolite relationships (2764)

## How to apply

Read the GutUDB EnrichedIndex counts and locate the epitranscriptomics level: GutUDB reports 1908 RNA m6A modification sites derived from miCLIP-seq (as a distinct level from the 137 RNA-seq transcriptomics data sets). Encode miCLIP-seq as the sole source of m6A sites as a module-level constant in the harmonization script, treating epitranscriptomics as a separate bulk omics level with its own entry in the per-level statistics output. Implement an aggregation function that emits the m6A site count (1908) into the combined JSON alongside the eight other bulk omics types and the four single-cell/spatial modalities. Rationale: GutUDB explicitly separates m6A sites from RNA-seq profiles, so the statistics object must carry a dedicated epitranscriptomics record rather than folding sites into the transcriptomics profile count.

## Related tools

- **Python** (Used to implement the aggregation functions that read GutUDB EnrichedIndex counts, encode the miCLIP-seq-derived m6A harmonization rule as a constant, and emit the combined per-level statistics JSON.)
- **GutUDB** (Source of the EnrichedIndex per-level counts and the 1908 miCLIP-seq m6A sites that are aggregated into the statistics JSON.) — https://github.com/Ansanqi/GutUDB

## Examples

```
python scripts/aggregate_levels.py --enriched-index EnrichedIndex.json --m6a-source miCLIP-seq --output gutudb_statistics.json
```

## Evaluation signals

- The output JSON parses without errors and contains a distinct epitranscriptomics entry separate from the transcriptomics RNA-seq record.
- The epitranscriptomics entry reports exactly 1908 m6A sites, matching the EnrichedIndex / GutUDB-reported value.
- The combined statistics object contains an entry for each of the eight bulk omics levels plus the four single-cell/spatial modalities.
- The m6A site count is sourced from miCLIP-seq and not conflated with the RNA-seq profile count (7.9 million profiles).
- Cross-check that other bulk-level counts (SNPs 72248, CNVs 54131, SVs 1097, mutated genes 92888, circRNAs 62, lncRNAs 182, microRNAs 58, microbe-metabolite relationships 2764) are unchanged when the epitranscriptomics entry is added.

## Limitations

- The m6A site count (1908) is a fixed value from the GutUDB snapshot described in the article; future releases with additional miCLIP-seq data will change this number and require the constant to be updated.
- The skill only aggregates site counts; it does not perform m6A peak calling or differential methylation analysis on the underlying miCLIP-seq reads.

## Evidence

- [abstract] m6A site count source: "Based on miCLIP‐seq, 1908 RNA m6A modification sites were included in GutUDB"
- [abstract] bulk omics levels description: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] transcriptomics level counts: [evidence span withheld — anchor status: not_found]

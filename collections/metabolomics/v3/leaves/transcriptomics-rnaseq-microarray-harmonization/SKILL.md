---
name: transcriptomics-rnaseq-microarray-harmonization
description: Reference for building multiomics databases that ingest RNA-seq and microarray
  transcriptomics datasets into per-level statistics.
when_to_use_negative:
- Input is single-cell or spatial transcriptomics — those modalities follow a separate
  GutUDB ingestion path, not the bulk RNA-seq/microarray harmonization rules.
- The goal is querying or browsing the already-built GutUDB statistics page rather
  than reconstructing or re-deriving the ingestion pipeline.
- Input omics type is genomic variation (SNPs, CNVs, SVs), methylation, proteomics,
  or metabolomics — those levels require their own per-level constants and aggregation
  rules.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3172
tools:
- name: GutUDB
  role: Target multiomics database whose per-level statistics (bulk transcriptomics
    counts, ncRNA panels) this skill reconstructs
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:6d9562033f9d73244e37b590d1afae0cf59f07a9005c3b1ca43b3cd5f7c3308d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - Python
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Transcriptomics RNA-seq/microarray harmonization

## Summary

Normalizes and merges bulk transcriptomics data types (RNA-seq and microarray) into per-level statistics within GutUDB's multiomics database framework, enabling consistent counts and browsing across intestinal disease datasets.

## When to use

Reference for building multiomics databases that ingest RNA-seq and microarray transcriptomics datasets into per-level statistics.

## When NOT to use

- Input is single-cell or spatial transcriptomics — those modalities follow a separate GutUDB ingestion path, not the bulk RNA-seq/microarray harmonization rules.
- The goal is querying or browsing the already-built GutUDB statistics page rather than reconstructing or re-deriving the ingestion pipeline.
- Input omics type is genomic variation (SNPs, CNVs, SVs), methylation, proteomics, or metabolomics — those levels require their own per-level constants and aggregation rules.

## Inputs

- Bulk transcriptomics datasets (RNA-seq expression matrices, microarray intensity tables)
- GutUDB EnrichedIndex per-level counts (e.g. 137 RNA-seq datasets, 7.9M transcriptional profiles, 62 circRNAs, 182 lncRNAs, 58 miRNAs)

## Outputs

- Per-level transcriptomics statistics record (counts of RNA-seq profiles, ncRNA types) merged into a combined JSON
- Canonical gutudb_statistics.json containing entries for each of the eight bulk omics levels and four single-cell/spatial modalities

## How to apply

Encode GutUDB's harmonization rules for bulk transcriptomics (RNA-seq and microarray) as module-level constants in the ingestion script. Parse the GutUDB EnrichedIndex counts for the transcriptomics level — 137 RNA-seq data sets totaling 7.9 million transcriptional profiles, plus ncRNA panels (62 circRNAs, 182 lncRNAs, 58 miRNAs) — and merge them into a single per-level statistics record. Combine the transcriptomics record with records derived from the other seven bulk omics types (genomics, m6A modification, microbe-metabolite relationships) and the four single-cell/spatial modalities. Emit the aggregated per-level statistics as a single JSON object and validate that each level's counts match the EnrichedIndex-reported values before saving to the canonical output file.

## Related tools

- **GutUDB** (Target multiomics database whose per-level statistics (bulk transcriptomics counts, ncRNA panels) this skill reconstructs) — https://github.com/Ansanqi/GutUDB

## Examples

```
python -m gutudb.harmonize --level transcriptomics --rna-seq raw_counts/ --microarray norm_tables/ --out gutudb_statistics.json
```

## Evaluation signals

- Output JSON parses without error.
- JSON contains an entry for each of the eight bulk omics levels plus the four single-cell/spatial modalities.
- Transcriptomics entry reports 137 RNA-seq data sets / 7.9 million transcriptional profiles, matching the EnrichedIndex value.
- ncRNA panel counts (62 circRNAs, 182 lncRNAs, 58 miRNAs) match GutUDB's published statistics.
- Counts do not drift after re-running the aggregation pipeline (deterministic output).

## Limitations

- The skill's counts are frozen at GutUDB's current snapshot (~9M profiles total, 56 intestinal diseases, six species); ingesting new datasets requires re-deriving EnrichedIndex values rather than assuming these constants.
- No per-step harmonization parameters (e.g. normalization method, batch-correction model, mapping thresholds) are documented in the extracted article text — only the final counts are reported, so the skill must treat the harmonization rules as opaque constants unless upstream code is consulted.
- The README states 'No changelog found,' so versioning of the underlying statistics cannot be tracked from the repository alone.

## Evidence

- [abstract] 137 RNA-seq data sets related to 7.9 million transcriptional profiles: [evidence span withheld — anchor status: not_found]
- [abstract] including 62 circular RNAs, 182 long noncoding RNAs, and 58 microRNAs: "including 62 circular RNAs, 182 long noncoding RNAs, and 58 microRNAs"
- [abstract] Gut Universe Database (GutUDB) provides a comprehensive, systematic, and practical platform for researchers: "Gut Universe Database (GutUDB) provides a comprehensive, systematic, and practical platform for researchers"

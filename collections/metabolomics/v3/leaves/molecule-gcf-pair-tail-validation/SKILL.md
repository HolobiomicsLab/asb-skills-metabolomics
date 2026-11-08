---
name: molecule-gcf-pair-tail-validation
description: Use when binary presence/absence GCF and ion tables from 363 strains
  have been correlated and you must validate that known molecule–GCF pairs (e.g. 9
  detoxin/rimosamide-type pairs) rank in the rightmost tail, or choose between BiG-SCAPE
  mode/cutoff settings.
when_to_use_negative:
- The ion identity table has not been built yet — ions must already satisfy mass within
  4 ppm and fragmentation cosine similarity above 0.75 before correlating
- A GCF of interest has representative gene clusters in only a single strain — such
  GCFs were not considered correlatable and cannot be tail-validated
- The goal is de novo molecule-GCF linking without any known pairs to anchor tail
  validation — molecular network family identification in this study was performed
  manually against known detoxins and rimosamides, not automatically inferred
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Generates BGC sequence similarity networks under global and glocal modes with
    distance cutoffs 0.30/0.50; Affinity Propagation on the filtered network assigns
    GCFs whose presence/absence matrix feeds the correlations
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH
  role: Supplies the BGC GenBank predictions used directly as BiG-SCAPE inputs for
    the 363 strains
- name: HMMER 3.1b2
  role: Domain detection against Pfam version 31 during BiG-SCAPE distance calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:d8c5dd8f9113068016a7982dd12b723ef2c155c452ad9b12e1690e0d919e25e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# molecule-gcf-pair-tail-validation

## Summary

Validates metabologenomic correlation output by checking whether known molecule–GCF pairs fall in the rightmost tail of the binary correlation score distribution, and selects the BiG-SCAPE alignment mode/cutoff combination (glocal, 0.30) that best recovers these known connections.

## When to use

Use when binary presence/absence GCF and ion tables from 363 strains have been correlated and you must validate that known molecule–GCF pairs (e.g. 9 detoxin/rimosamide-type pairs) rank in the rightmost tail, or choose between BiG-SCAPE mode/cutoff settings.

## When NOT to use

- The ion identity table has not been built yet — ions must already satisfy mass within 4 ppm and fragmentation cosine similarity above 0.75 before correlating
- A GCF of interest has representative gene clusters in only a single strain — such GCFs were not considered correlatable and cannot be tail-validated
- The goal is de novo molecule-GCF linking without any known pairs to anchor tail validation — molecular network family identification in this study was performed manually against known detoxins and rimosamides, not automatically inferred

## Inputs

- antiSMASH-predicted BGC GenBank files for 363 strains
- Binary GCF presence/absence matrix (363 strains x GCFs) per BiG-SCAPE mode/cutoff combination
- Binary ion presence/absence table (5,824 ions detected in two or more strains; 4 ppm mass tolerance, cosine similarity > 0.75)
- Curated list of 9 known molecule-GCF combinations
- BiG-SCAPE filtered networks at cutoffs 0.30 and 0.50 for global and glocal modes

## Outputs

- metabologenomic_correlations.csv with per-pair correlation scores and mode/cutoff selection summary
- Correlation score distributions per mode/cutoff combination with tail positions of the 9 known pairs
- Selected default setting: glocal alignment mode at 0.30 distance cutoff

## How to apply

Run BiG-SCAPE (with HMMER 3.1b2 against Pfam version 31) on the antiSMASH-predicted BGC GenBank files under both global and glocal alignment modes, generating networks filtered at distance cutoffs 0.30 and 0.50 for each mode. Assign GCFs via Affinity Propagation clustering on each filtered network and build a binary 363-strain x GCF presence/absence matrix per combination. Restrict to correlatable GCFs (representative gene clusters in two or more strains) and correlate binary GCF presence/absence against the binary ion table of the 5,824 ions detected in two or more strains (ions required mass within 4 ppm and fragmentation cosine similarity above 0.75). Compare the four mode/cutoff combinations and select glocal/0.30 as default; correctness is judged by at least 6 of the 9 known molecule-GCF pairs falling in the rightmost tail of the correlation score distribution, with per-pair scores written to metabologenomic_correlations.csv.

## Related tools

- **BiG-SCAPE** (Generates BGC sequence similarity networks under global and glocal modes with distance cutoffs 0.30/0.50; Affinity Propagation on the filtered network assigns GCFs whose presence/absence matrix feeds the correlations) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH** (Supplies the BGC GenBank predictions used directly as BiG-SCAPE inputs for the 363 strains)
- **HMMER 3.1b2** (Domain detection against Pfam version 31 during BiG-SCAPE distance calculation)

## Examples

```
python bigscape.py -i antiSMASH_bgc_genbanks/ --mode glocal --cutoffs 0.30 0.50 --mix
```

## Evaluation signals

- At least 6 of the 9 known molecule-GCF pairs fall in the rightmost tail of the metabologenomic correlation score distribution under the selected glocal/0.30 setting
- All four mode/cutoff combinations (global/glocal x 0.30/0.50) produce complete presence/absence matrices restricted to GCFs with representatives in >=2 strains
- metabologenomic_correlations.csv contains a per-pair correlation score for each of the 9 known pairs plus the selection summary
- Ion table row count matches the expected 5,824 ions detected in two or more strains after 4 ppm / cosine > 0.75 filtering
- GCF assignment reproduces Affinity Propagation clustering on each cutoff-filtered network (default BiG-SCAPE cutoff of 0.3 applies unless overridden to 0.50)

## Limitations

- The glocal/0.30 choice is validated only against the small set of 9 known molecule-GCF pairs from the detoxin/rimosamide-associated data; it is not a genome-wide benchmark
- Molecular network family identification was performed manually to identify ions related to known detoxins and rimosamides, so the known-pair list depends on human curation
- BiG-SCAPE interactive reports can drop genomes with duplicate ORGANISM identifiers (only 96 of 103 Streptomyces genomes reported in one case), which could silently alter the GCF matrix

## Evidence

- [intro] Glocal alignment mode as the default BiG-SCAPE mode: [evidence span withheld — anchor status: not_found]
- [readme] BiG-SCAPE input format (antiSMASH GenBank BGC predictions): [evidence span withheld — anchor status: external_doc]

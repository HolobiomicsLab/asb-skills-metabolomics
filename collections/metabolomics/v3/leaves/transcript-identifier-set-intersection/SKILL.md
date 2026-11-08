---
name: transcript-identifier-set-intersection
description: Decision support for quantifying transcript-coverage overlap between
  variant annotation tools (ANNOVAR, SnpEff, VEP) before interpreting HGVS or coding-impact
  discrepancies as real biological or clinical differences.
when_to_use_negative:
- When the goal is to compare annotation content (HGVS c./p. strings, coding impact,
  LoF/PVS1 calls) rather than transcript identifier coverage — a different concordance
  analysis is needed
- When annotation outputs come from only one tool or one reference database, since
  pairwise/three-way cross-tool intersection is undefined and no concordance percentage
  can be computed
- When transcript identifiers are missing, mixed between RefSeq and Ensembl prefixes,
  or unnormalized (e.g., version suffixes inconsistently present), since set intersection
  on unclean identifiers would produce misleading overlap
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3320
tools:
- name: Variant Effect Predictor (VEP)
  role: Source of the largest transcript identifier set (25,381 RefSeq NM/NR; 67,695
    Ensembl ENST) whose reported transcript accessions are intersected with the other
    two tools' sets
- name: SnpEff
  role: Source of the smallest transcript identifier set (5,848 RefSeq; 6,183 Ensembl)
    contributing to the cross-tool concordance computation
- name: ANNOVAR
  role: Third annotation tool whose transcript identifiers are intersected against
    SnpEff and VEP sets to compute RefSeq and Ensembl concordance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:5a0aca16e9b8ec129725bb2c954c0e61b15066f86e7d5372d23181c960712c7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# transcript-identifier-set-intersection

## Summary

Computes cross-tool concordance of RefSeq (NM/NR) and Ensembl (ENST) transcript identifier sets used by ANNOVAR, SnpEff, and VEP, quantifying the fraction of shared transcript accessions per reference database. This exposes how much of the annotation disagreement stems simply from tools using different transcript pools rather than differing variant models.

## When to use

Decision support for quantifying transcript-coverage overlap between variant annotation tools (ANNOVAR, SnpEff, VEP) before interpreting HGVS or coding-impact discrepancies as real biological or clinical differences.

## When NOT to use

- When the goal is to compare annotation content (HGVS c./p. strings, coding impact, LoF/PVS1 calls) rather than transcript identifier coverage — a different concordance analysis is needed
- When annotation outputs come from only one tool or one reference database, since pairwise/three-way cross-tool intersection is undefined and no concordance percentage can be computed
- When transcript identifiers are missing, mixed between RefSeq and Ensembl prefixes, or unnormalized (e.g., version suffixes inconsistently present), since set intersection on unclean identifiers would produce misleading overlap

## Inputs

- Annotation outputs (ANNOVAR, SnpEff, VEP) over the same input VCF, containing transcript accession fields (NM/NR for RefSeq, ENST for Ensembl)
- Reference database label per annotation run (RefSeq vs Ensembl) to group transcript sets

## Outputs

- Per-tool unique transcript counts split by database (RefSeq NM/NR count, Ensembl ENST count)
- Cross-tool transcript concordance percentages (pairwise and/or three-way) per reference database, e.g. 22.68% RefSeq (n=5,776) and 8.53% Ensembl (n=5,845)
- Tab-delimited summary table such as transcript_concordance.tsv

## How to apply

Extract the unique transcript identifiers reported in each tool's annotation output, separating RefSeq NM/NR accessions from Ensembl ENST accessions, from ANNOVAR, SnpEff, and VEP runs over the same input VCF (e.g., the 164,549 two-star ClinVar variants). Count unique transcripts per tool per database, then intersect the transcript identifier sets pairwise and across all three tools, normalizing the intersection as a percentage of the shared transcript pool per database. In the paper this yields 22.68% (n=5,776) cross-tool concordance for RefSeq and 8.53% (n=5,845) for Ensembl, with VEP covering the most transcripts (25,381 RefSeq NM/NR; 67,695 Ensembl ENST) and SnpEff the fewest (5,848 RefSeq; 6,183 Ensembl). Record per-tool counts and concordance percentages in a table (e.g., transcript_concordance.tsv) so low overlap can be traced as a root cause of downstream HGVSc/HGVSp/coding-impact discordance.

## Related tools

- **Variant Effect Predictor (VEP)** (Source of the largest transcript identifier set (25,381 RefSeq NM/NR; 67,695 Ensembl ENST) whose reported transcript accessions are intersected with the other two tools' sets)
- **SnpEff** (Source of the smallest transcript identifier set (5,848 RefSeq; 6,183 Ensembl) contributing to the cross-tool concordance computation)
- **ANNOVAR** (Third annotation tool whose transcript identifiers are intersected against SnpEff and VEP sets to compute RefSeq and Ensembl concordance)

## Examples

```
python scripts/transcript_concordance.py --annovar annovar.tsv --snpeff snpeff.txt --vep vep.tsv --ref-db refseq,ensembl --output transcript_concordance.tsv
```

## Evaluation signals

- VEP should report the largest unique transcript count (25,381 RefSeq NM/NR; 67,695 Ensembl ENST) and SnpEff the smallest (5,848 RefSeq; 6,183 Ensembl); deviations beyond rounding indicate an error in transcript-set extraction
- Cross-tool RefSeq concordance must equal 22.68% (n=5,776 shared transcripts) and Ensembl concordance 8.53% (n=5,845); any other value signals a parsing or intersection error
- RefSeq set counts must be separated from Ensembl set counts before intersection — a mixed-pool count that does not reproduce the two per-database concordance values indicates identifier normalization failure
- Concordance is a percentage of the shared transcript pool per database, not of the union; verify the denominator matches the per-database intersection size
- Output rows should round-trip: recomputing the intersection from a saved transcript_concordance.tsv must yield the same counts and percentages

## Limitations

- Cross-tool transcript concordance is low (22.68% RefSeq; 8.53% Ensembl), so restricting analysis to transcripts shared across all three tools discards most of each tool's coverage and can understate tool-specific strengths
- Different MANE selections still impact final variant annotation, so even identical transcript identifiers do not guarantee identical HGVSc/HGVSp strings
- Transcript availability alone does not explain all discrepancy — the study found residual HGVSc/HGVSp/coding-impact discordance and LoF/PVS1 divergence (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3% incorrect PVS1 downgrades) even after controlling for transcript choice
- No software versions are reported for the compared tools, so the exact transcript pools may shift with tool/reference releases

## Evidence

- [results] Cross-tool concordance 22.68% / 8.53%: [evidence span withheld — anchor status: not_found]
- [results] Per-tool transcript coverage extremes: [evidence span withheld — anchor status: not_found]
- [results] VEP and SnpEff per-database transcript counts: [evidence span withheld — anchor status: not_found]
- [abstract] Transcript concordance as root-cause context for annotation discordance: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [abstract] Concordance measures downstream of transcript availability: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [intro] Dataset basis for transcript identifier extraction: "comparing the concordance of variant nomenclature and predicted coding impacts generated by ANNOVAR, SnpEff, and VEP for 164,549 high review status ClinVar variants19,20"
- [intro] Same-transcript caveat (MANE selection): "However, different MANE selections still impact the final annotation."
- [discussion] Standardization recommendation motivating cross-tool transcript overlap analysis: "standardizing transcript sets and systematically cross-validating results across multiple annotation tools is"

---
name: target-decoy-random-matching-confidence-estimation
description: Use when de novo peptide predictions must be given an FDR-style confidence
  score for metaproteomics or dark-proteome spectra, especially when comparing de
  novo peptide detection power against a database search engine at a fixed 1% threshold.
when_to_use_negative:
- When a rigorous, purpose-built confidence-estimation method for de novo sequencing
  is available — this approach is an ad hoc target-decoy proxy, not a calibrated score
- When the goal is to assign peptides to foreign spectra or exploit de novo power
  beyond database matching — target-decoy matching misses that power
- When inputs are already FDR-controlled PSMs from a database search that need no
  de novo confidence assignment
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Casanovo
  role: Produces the de novo peptide predictions whose confidence is assigned by target-decoy
    random matching
  repo: https://github.com/Noble-Lab/casanovo
- name: Tide
  role: Database search engine whose 1% FDR identifications serve as the comparison
    baseline under the same filter
- name: Percolator
  role: Re-scores Tide PSMs to obtain confident database-search identifications at
    1% FDR for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:5384614509aaa9e13b0281614e78cb90acec82b86f054743354656632a92be45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - Percolator
  - Tide
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# target-decoy-random-matching-confidence-estimation

## Summary

Assigns confidence to de novo peptide sequencings (e.g. from Casanovo) by matching predicted peptides against a target and corresponding decoy peptide database, then filtering at a target-decoy-style random-matching FDR (1%) so de novo identifications can be compared head-to-head with database search (Tide+Percolator) peptide counts.

## When to use

Use when de novo peptide predictions must be given an FDR-style confidence score for metaproteomics or dark-proteome spectra, especially when comparing de novo peptide detection power against a database search engine at a fixed 1% threshold.

## When NOT to use

- When a rigorous, purpose-built confidence-estimation method for de novo sequencing is available — this approach is an ad hoc target-decoy proxy, not a calibrated score
- When the goal is to assign peptides to foreign spectra or exploit de novo power beyond database matching — target-decoy matching misses that power
- When inputs are already FDR-controlled PSMs from a database search that need no de novo confidence assignment

## Inputs

- Tandem MS/MS spectra files (DDA, bottom-up) from metaproteomics samples, e.g. MassIVE accession MSV000094709 (Bering Strait / Chukchi Sea)
- Casanovo de novo peptide predictions (exportable as mzTab)
- Target metapeptide database and corresponding decoy peptide database
- Non-redundant environmental protein database (and its decoy)
- Tide+Percolator PSM identifications re-scored at 1% FDR

## Outputs

- Per-tool, per-database unique peptide counts at the 1% random-matching filter (e.g. peptide_counts.csv: Tide 5623 meta / 2460 env; Casanovo 8277 meta / 3532 env)
- Set of de novo peptide identifications retained at 1% random-matching FDR

## How to apply

Run Casanovo de novo sequencing on the tandem MS/MS spectra (e.g. the Bering Strait / Chukchi Sea metaproteomics files from MassIVE MSV000094709) and match each predicted peptide against a target metapeptide/environmental protein database and a corresponding decoy database; a prediction whose match is to the decoy set is treated as a random match. Apply the same target-decoy-style 1% random-matching filter to both the Casanovo de novo identifications and the Tide+Percolator database-search identifications so both tools are controlled at an equivalent confidence level. Count unique peptides passing the filter for each tool and each database (metapeptide vs. non-redundant environmental) and write per-tool, per-database counts (e.g. peptide_counts.csv) for comparison. The rationale is that the field lacks a rigorous confidence-estimation method for de novo sequencing, so the standard database-search target-decoy logic is borrowed as an ad hoc but comparable filter.

## Related tools

- **Casanovo** (Produces the de novo peptide predictions whose confidence is assigned by target-decoy random matching) — https://github.com/Noble-Lab/casanovo
- **Tide** (Database search engine whose 1% FDR identifications serve as the comparison baseline under the same filter)
- **Percolator** (Re-scores Tide PSMs to obtain confident database-search identifications at 1% FDR for comparison)

## Examples

```
casanovo sequence --mode de_novo --output predictions.mztab bering_strait.mgf  # then match peptides vs target+decoy metapeptide FASTA and filter at 1% random-matching FDR
```

## Evaluation signals

- At the 1% random-matching filter, Casanovo peptide counts must match the Fig. 4a-b reference values: 8277 peptides (Bering Strait metapeptide DB) and 3532 (Chukchi Sea), vs Tide+Percolator 5623 and 2460
- Full-database variants must also reproduce: Casanovo 3425/1612 and Tide 1364/682 for meta/env databases
- Casanovo should show ~44–47% more detected peptides than Tide+Percolator at the same 1% filter, as a sanity check on relative power
- Discrepancies between reproduced counts and reference values should be flagged for expert review
- Decoy matches after filtering should be consistent with the 1% random-matching criterion (roughly 1% of target matches)

## Limitations

- Matching Casanovo predictions to a target and corresponding decoy peptide database 'misses out on the power of de novo sequencing to assign peptides to foreign spectra'
- The field lacks a rigorous method for confidence estimation for de novo sequencing — this target-decoy random-matching filter is an ad hoc workaround, not a calibrated probability
- It remains an open question whether, for a given DDA dataset, Casanovo outperforms a standard database search in statistical power to detect peptides, so count gains should be interpreted carefully

## Evidence

- [discussion] we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra.: "we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra."
- [discussion] at this stage one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing.: "at this stage one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing."
- [results] Using Casanovo predictions filtered at a 1% random matching criterion against the metapeptide database, 8277 peptides are detected in the Bering Strait data and 3532 in the Chukchi Sea data, increases of 47% and 44%: [evidence span withheld — anchor status: not_found]
- [discussion] an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides.: "an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides."

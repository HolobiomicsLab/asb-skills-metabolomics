---
name: tide-percolator-database-search-peptide-identification
description: Use when DDA tandem MS spectra (e.g., from MassIVE MSV000094709) must
  be searched against a sample-specific metapeptide or environmental protein database
  with 1% FDR control, to yield confident PSM/peptide counts as a database-search
  baseline.
when_to_use_negative:
- The sample's protein set is unknown or contains unexpected peptides (contaminants,
  variants, unexpected PTMs, foreign peptides) — sequence database searching requires
  the set of proteins to be known in advance; use de novo sequencing (Casanovo) instead.
- The goal is to assign peptides to foreign spectra beyond target-decoy matching against
  a database — this approach 'misses out on the power of de novo sequencing' per the
  article.
- Input is already a list of confident PSMs or peptide identifications requiring only
  counting or FDR re-application rather than a new search.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Tide
  role: Database search engine run on the tandem MS spectra against the metapeptide
    and environmental protein databases
  repo: http://cruxtoolkit.sourceforge.net
- name: Percolator
  role: Re-scores Tide PSMs to obtain confident identifications for 1% FDR filtering
- name: Casanovo
  role: De novo sequencing comparator whose peptide-detection counts (8277 meta /
    3532 env at 1% random matching) are benchmarked against the Tide+Percolator baseline
  repo: https://github.com/Noble-Lab/casanovo
- name: SearchGUI
  role: Common interface that can configure and run Tide database searches within
    proteomics pipelines
  repo: https://github.com/CompOmics/searchgui
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:87252363931e9c9f1efd82787a1d6e8648b5a4f6b027ea13291b7be569686741
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - Percolator
  - SearchGUI
  - Tide
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tide-percolator-database-search-peptide-identification

## Summary

Identify peptides from tandem MS/MS spectra by searching with the Tide database search engine against a sample-specific FASTA (e.g., a metapeptide or non-redundant environmental protein database) and re-scoring PSMs with Percolator, filtering at 1% FDR. Serves as the standard database-search baseline against which de novo sequencing (Casanovo) statistical power is compared in metaproteomics.

## When to use

Use when DDA tandem MS spectra (e.g., from MassIVE MSV000094709) must be searched against a sample-specific metapeptide or environmental protein database with 1% FDR control, to yield confident PSM/peptide counts as a database-search baseline.

## When NOT to use

- The sample's protein set is unknown or contains unexpected peptides (contaminants, variants, unexpected PTMs, foreign peptides) — sequence database searching requires the set of proteins to be known in advance; use de novo sequencing (Casanovo) instead.
- The goal is to assign peptides to foreign spectra beyond target-decoy matching against a database — this approach 'misses out on the power of de novo sequencing' per the article.
- Input is already a list of confident PSMs or peptide identifications requiring only counting or FDR re-application rather than a new search.

## Inputs

- DDA tandem MS spectra files from MassIVE accession MSV000094709 (Bering Strait / Chukchi Sea metaproteomics samples)
- Sample-specific metapeptide database (FASTA)
- Non-redundant environmental protein database (FASTA)
- Corresponding decoy peptide database for target-decoy FDR estimation

## Outputs

- Percolator-rescored PSMs filtered at 1% FDR per database
- peptide_counts.csv with per-tool, per-database unique peptide counts (Tide 5623 meta / 2460 env; 1364/682 full-length for Fig. 4a-b)

## How to apply

Download the tandem MS spectra (MassIVE MSV000094709) together with the metapeptide database and the non-redundant environmental protein database. Run Tide on the spectra against each of the two databases, then re-score the resulting PSMs with Percolator to obtain confident database-search identifications. Apply the target-decoy-style 1% FDR (or 1% random matching) filter to control reported peptide confidence. Count the unique peptides passing the filter per tool and per database (metapeptide vs. environmental, full-length vs. truncated) and write the per-tool, per-database counts to peptide_counts.csv, flagging discrepancies from the reference values (Tide 5623 meta / 2460 env, 1364/682 full) for expert review.

## Related tools

- **Tide** (Database search engine run on the tandem MS spectra against the metapeptide and environmental protein databases) — http://cruxtoolkit.sourceforge.net
- **Percolator** (Re-scores Tide PSMs to obtain confident identifications for 1% FDR filtering)
- **Casanovo** (De novo sequencing comparator whose peptide-detection counts (8277 meta / 3532 env at 1% random matching) are benchmarked against the Tide+Percolator baseline) — https://github.com/Noble-Lab/casanovo
- **SearchGUI** (Common interface that can configure and run Tide database searches within proteomics pipelines) — https://github.com/CompOmics/searchgui

## Examples

```
crux tide-search spectra.mzML metapeptide.fasta && crux percolator tide-search.psms.txt
```

## Evaluation signals

- Reproduced counts in peptide_counts.csv match the Fig. 4a-b reference values for Tide: 5623 (metapeptide) and 2460 (environmental), with 1364/682 full-length matches.
- The reported identifications are controlled at 1% FDR (target-decoy-style) for both the database search and the Casanovo comparison arm.
- Counts are split per tool and per database (metapeptide vs. environmental; full-length vs. truncated), matching the 6-step validation workflow.
- Discrepancies from reference values are explicitly flagged for expert review rather than silently accepted.
- The Casanovo arm shows the expected gains over the baseline (8277/3532, i.e., +47% and +44%); a reversed ordering would indicate a pipeline error.

## Limitations

- Sequence database searching requires the set of proteins in the sample to be known in advance, preventing detection of unexpected peptides (contaminants, sequence variants, unexpected PTMs, foreign peptides).
- The target-decoy matching approach for metaproteomics misses the power of de novo sequencing to assign peptides to foreign spectra.
- No rigorous method for confidence estimation for de novo sequencing exists, complicating a fully like-for-like power comparison with database-search FDR control.

## Evidence

- [results] Using Casanovo predictions filtered at a 1% random matching criterion against the metapeptide database, 8277 peptides are detected in the Bering Strait data: [evidence span withheld — anchor status: not_found]
- [discussion] an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides.: "an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides."
- [discussion] we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra.: "we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra."
- [intro] Pioneered by the SEQUEST algorithm2, dozens of database search engines have been subsequently developed and are very widely deployed3.: "Pioneered by the SEQUEST algorithm2, dozens of database search engines have been subsequently developed and are very widely deployed3."
- [readme] SearchGUI is a a highly adaptable open-source common interface for configuring and running proteomics search and de novo engines, currently supporting: [evidence span withheld — anchor status: external_doc]

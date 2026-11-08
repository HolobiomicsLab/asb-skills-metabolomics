---
name: metaproteome-peptide-count-benchmarking
description: Use when asked to reproduce or compare peptide-detection counts for de
  novo vs database-search proteomics tools on metaproteomics spectra (e.g., Fig. 4a-b
  reference values) against metapeptide or environmental protein FASTA databases.
when_to_use_negative:
- Input is quantitative proteomics (feature tables, LFQ intensities) rather than raw
  DDA tandem MS spectra needing peptide identification counts
- The protein complement of the sample is fully known and the goal is standard identification
  — database searching alone suffices and de novo matching adds little
- Data are DIA or non-bottom-up acquisition; this benchmarking workflow assumes DDA
  bottom-up tandem MS spectra
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: De novo sequencing of the tandem MS spectra; predicted peptides are matched
    against target/decoy databases to assign confidence under the 1% random-matching
    filter
  repo: https://github.com/Noble-Lab/casanovo
- name: Tide
  role: Database search engine run on the same spectra against both the metapeptide
    and environmental protein databases to establish the baseline identifications
  repo: http://cruxtoolkit.sourceforge.net
- name: Percolator
  role: Re-scores Tide PSMs to obtain confident database-search identifications for
    the baseline comparison arm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:29575a2198823e019217b6bfcad5def136794fed1690ec55aaa1028948b9c5b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - Percolator
  - Tide
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metaproteome-peptide-count-benchmarking

## Summary

Benchmarks de novo peptide sequencing (Casanovo) against standard database search (Tide+Percolator) on metaproteomics tandem MS data by counting unique peptides detected against sample-specific metapeptide and environmental protein databases at a 1% random-matching confidence filter, outputting per-tool, per-database counts for comparison.

## When to use

Use when asked to reproduce or compare peptide-detection counts for de novo vs database-search proteomics tools on metaproteomics spectra (e.g., Fig. 4a-b reference values) against metapeptide or environmental protein FASTA databases.

## When NOT to use

- Input is quantitative proteomics (feature tables, LFQ intensities) rather than raw DDA tandem MS spectra needing peptide identification counts
- The protein complement of the sample is fully known and the goal is standard identification — database searching alone suffices and de novo matching adds little
- Data are DIA or non-bottom-up acquisition; this benchmarking workflow assumes DDA bottom-up tandem MS spectra

## Inputs

- Metaproteomics DDA tandem mass spectra from MassIVE accession MSV000094709 (Bering Strait / Chukchi Sea samples)
- Sample-specific metapeptide FASTA database
- Non-redundant environmental protein FASTA database
- Corresponding decoy peptide databases for target-decoy confidence assignment

## Outputs

- peptide_counts.csv with per-tool, per-database unique peptide counts (Tide: 5623 meta / 2460 env, 1364/682 full; Casanovo: 8277 meta / 3532 env, 3425/1612 full)
- FDR-filtered PSM/peptide identification lists per tool (Tide+Percolator and Casanovo) for the metapeptide and environmental databases

## How to apply

Download the metaproteomics tandem MS spectra from the MassIVE accession (MSV000094709) together with the metapeptide and non-redundant environmental protein databases. Run Tide on the spectra against each database and re-score PSMs with Percolator to obtain confident database-search identifications; in parallel run Casanovo de novo sequencing on the same spectra. Apply a target-decoy-style 1% random-matching filter to both tool's identifications (matching Casanovo predictions against a target and corresponding decoy peptide database) so peptide confidence is controlled identically across tools. Count unique peptides passing the filter per tool and per database (metapeptide vs environmental, full-length variants counted separately), write the results to peptide_counts.csv, and validate against reference values: Tide 5623 meta / 2460 env (1364/682 full) and Casanovo 8277 meta / 3532 env (3425/1612 full), flagging discrepancies for expert review. The rationale is that de novo sequencing can detect peptides from unexpected/foreign sequences that a database search cannot, since database searching requires the sample's proteins to be known in advance.

## Related tools

- **Casanovo** (De novo sequencing of the tandem MS spectra; predicted peptides are matched against target/decoy databases to assign confidence under the 1% random-matching filter) — https://github.com/Noble-Lab/casanovo
- **Tide** (Database search engine run on the same spectra against both the metapeptide and environmental protein databases to establish the baseline identifications) — http://cruxtoolkit.sourceforge.net
- **Percolator** (Re-scores Tide PSMs to obtain confident database-search identifications for the baseline comparison arm)

## Examples

```
casanovo db_search spectra.mgf
```

## Evaluation signals

- peptide_counts.csv Tide row matches 5623 (metapeptide) / 2460 (environmental), with full-length counts 1364/682
- peptide_counts.csv Casanovo row matches 8277 (metapeptide) / 3532 (environmental), with full-length counts 3425/1612
- Relative increases of Casanovo over Tide are ~47% (Bering Strait) and ~44% (Chukchi Sea) on the metapeptide database
- Both tools' identifications are filtered at the same 1% target-decoy-style random-matching criterion before counting
- Unique peptide counts are computed per tool AND per database (metapeptide vs environmental), not pooled

## Limitations

- The target-decoy matching approach for de novo predictions misses out on the power of de novo sequencing to assign peptides to foreign spectra
- There is no rigorous method for confidence estimation for de novo sequencing; the random-matching filter is an interim control, a key bottleneck in the field
- The reference values are dataset-specific (MSV000094709); whether Casanovo outperforms database search in statistical power is an open question for a general DDA dataset

## Evidence

- [discussion] we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra.: "we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra."
- [discussion] an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides.: "an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides."
- [intro] a fundamental requirement for sequence database searching is that the set of proteins that may be present in the sample is known in advance.: "a fundamental requirement for sequence database searching is that the set of proteins that may be present in the sample is known in advance."
- [methods] Casanovo-DB mode (`casanovo db_search`) to use Casanovo as a learned score function for sequence database searching: "Casanovo-DB mode (`casanovo db_search`) to use Casanovo as a learned score function for sequence database searching"
- [readme] particularly useful for immunopeptidomics, metaproteomics, paleoproteomics, venomics, or any setting in which you are interested in identifying peptides that may not be in your protein database: "particularly useful for immunopeptidomics, metaproteomics, paleoproteomics, venomics, or any setting in which you are interested in identifying peptides that may not be in your protein database"

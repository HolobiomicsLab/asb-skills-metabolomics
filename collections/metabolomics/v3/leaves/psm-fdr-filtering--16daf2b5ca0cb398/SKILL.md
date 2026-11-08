---
name: psm-fdr-filtering
description: Use when MS/MS spectra (e.g. MGF files) must be converted into trusted
  peptide identifications via database search to serve as ground truth for evaluating
  de novo sequencing output, or whenever PSMs must be FDR-controlled.
when_to_use_negative:
- The goal is direct de novo sequencing from spectra without any reference database
  — this skill requires known reference sequences to search against.
- Input is raw vendor instrument files that have not yet been converted to MGF/mascot
  generic format for the search engines.
- The evaluation target is peptide assembly quality (e.g. ALPS sequence coverage)
  rather than per-PSM identification confidence.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-GF+
  role: Database search engine executed via SearchGUI to produce candidate PSMs from
    the MGF spectra against the antibody reference database
- name: X!Tandem
  role: Second database search engine run in parallel so results can be combined with
    MS-GF+ for more complete PSM coverage
- name: SearchGUI
  role: Wrapper (v.4.1.7) that executes MS-GF+ and X!Tandem with the specified precursor/fragment
    tolerances and fixed/variable modifications
- name: PeptideShaker
  role: Post-processor (v.2.2.2) that consolidates PSMs from both engines and applies
    the 1% FDR filter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:8b28f54ee51c67a5ebe6ff0a05a5f5c724853aad1b53637189a50c222c63eeec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - Novor
  - PeptideShaker
  - SearchGUI
  - X!Tandem
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# psm-fdr-filtering

## Summary

Generates a high-confidence reference set of peptide-spectrum matches (PSMs) by combining database search results from multiple engines (MS-GF+ and X!Tandem) and filtering at 1% FDR, producing pseudo-ground-truth peptide identifications for benchmarking de novo sequencing predictions.

## When to use

Use when MS/MS spectra (e.g. MGF files) must be converted into trusted peptide identifications via database search to serve as ground truth for evaluating de novo sequencing output, or whenever PSMs must be FDR-controlled.

## When NOT to use

- The goal is direct de novo sequencing from spectra without any reference database — this skill requires known reference sequences to search against.
- Input is raw vendor instrument files that have not yet been converted to MGF/mascot generic format for the search engines.
- The evaluation target is peptide assembly quality (e.g. ALPS sequence coverage) rather than per-PSM identification confidence.

## Inputs

- MGF-converted MS/MS experiments (spectra files)
- Protein/antibody reference sequences used as the search database (FASTA)
- Enzyme specification per input file (to set enzyme-specific cleavage parameters)

## Outputs

- FDR-filtered PSM list at 1% FDR (e.g. 23,844 peptides across three antibody data sets)
- Pseudo-ground-truth peptide identifications for de novo sequencing benchmarking

## How to apply

Load the MGF-converted MS/MS experiments together with a protein sequence database (here, known antibody reference sequences) and run MS-GF+ and X!Tandem through SearchGUI (v.4.1.7), selecting enzyme-specific cleavage parameters per input file, a precursor tolerance of 10 ppm, a fragment mass tolerance of 0.02 Da, carbamidomethylation of cysteine (C + 57.02 Da) as fixed modification, and oxidation of methionine (M + 15.99 Da) plus deamidation of N and Q (+0.98 Da) as variable modifications. Post-process the combined results with PeptideShaker (v.2.2.2) to consolidate PSMs from both engines. Filter all resulting PSMs at 1% false discovery rate to retain only high-confidence identifications — in the source article this yielded 23,844 database-search-identified peptides across three antibody data sets, which served as the pseudo-ground truth against which de novo predictions from tools such as Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo were scored.

## Related tools

- **MS-GF+** (Database search engine executed via SearchGUI to produce candidate PSMs from the MGF spectra against the antibody reference database)
- **X!Tandem** (Second database search engine run in parallel so results can be combined with MS-GF+ for more complete PSM coverage)
- **SearchGUI** (Wrapper (v.4.1.7) that executes MS-GF+ and X!Tandem with the specified precursor/fragment tolerances and fixed/variable modifications)
- **PeptideShaker** (Post-processor (v.2.2.2) that consolidates PSMs from both engines and applies the 1% FDR filter)

## Evaluation signals

- The filtered output must contain PSMs at 1% FDR, i.e. the estimated false discovery rate of retained identifications does not exceed 1%.
- The resulting peptide count matches the reported reference set size (23,844 database-search-identified peptides across the three antibody data sets).
- Both engines (MS-GF+ and X!Tandem) contributed PSMs that were consolidated via PeptideShaker rather than only a single engine's output.
- Enzyme-specific cleavage parameters were applied per input file, and the specified tolerances (10 ppm precursor, 0.02 Da fragment) and modifications (C +57.02 fixed; M +15.99, N/Q +0.98 variable) are reflected in the search configuration.

## Limitations

- Pseudo-ground truth depends on the completeness of the reference database; peptides absent from the reference sequences cannot be recovered by the search.
- FDR-filtered database search identifications represent a proxy, not an absolute truth, so de novo predictions may outperform the 'ground truth' on some spectra.
- The 1% FDR threshold and parameter set (10 ppm / 0.02 Da, specific modifications) are tuned for this antibody data setup and may need adjustment for other instruments or digestion chemistries.

## Evidence

- [other] The pseudo-ground truth was produced by combining the database search results of MS-GF+ and X!Tandem, both executed via SearchGUI (v.4.1.7) and post-processed via PeptideShaker (v.2.2.2), with enzyme-specific cleavage parameters per input file.: [evidence span withheld — anchor status: not_found]
- [other] Filter all resulting PSMs at a 1% false discovery rate to retain high-confidence identifications, yielding the reference set of 23,844 peptides across the three antibody data sets.: [evidence span withheld — anchor status: not_found]
- [abstract] we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo): "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"

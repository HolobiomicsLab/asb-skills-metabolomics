---
name: peptide-filtering-for-assembly-input
description: Use when raw de novo peptide predictions (e.g. from Novor, pNovo 3, DeepNovo,
  SMSNet, PointNovo, Casanovo) must be quality-filtered before de Bruijn assembly
  of antibody sequences with ALPS.
when_to_use_negative:
- The predictions will be used with a template-mapping tool such as Stitch or a database
  search pipeline, which have their own scoring/filtering semantics — this threshold
  is calibrated specifically for ALPS de Bruijn assembly input.
- You need high-precision filtering well beyond 50% AA precision (e.g. for direct
  sequence reporting rather than assembly input); a simple precursor mass filter may
  be more appropriate.
- No database-search-derived reference peptides are available to compute AA precision,
  so tool-specific thresholds cannot be calibrated.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Novor
  role: De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision
    point
- name: pNovo 3
  role: De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision
    point
- name: DeepNovo
  role: De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision
    point
- name: SMSNet
  role: De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision
    point
- name: PointNovo
  role: De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision
    point
- name: Casanovo
  role: De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision
    point; its predictions were the assembly input in the paper
- name: PeptideShaker
  role: Combines MS-GF+ and X!Tandem database-search results at 1% FDR to build the
    pseudo-ground truth used for precision calibration
- name: ALPS
  role: Downstream de Bruijn assembler that consumes the filtered peptide predictions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:e8984b2dee7bdc04a7771b01af1994be22a2f2f98a14581401da502a13518fc0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - Casanovo
  - DeepNovo
  - Novor
  - PeptideShaker
  - PointNovo
  - SMSNet
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# peptide-filtering-for-assembly-input

## Summary

Calibrates per-tool confidence-score thresholds at 50% amino-acid-level precision against a database-search pseudo-ground truth, then removes low-confidence de novo peptide predictions before feeding the surviving sequences into the ALPS de Bruijn assembly.

## When to use

Use when raw de novo peptide predictions (e.g. from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) must be quality-filtered before de Bruijn assembly of antibody sequences with ALPS.

## When NOT to use

- The predictions will be used with a template-mapping tool such as Stitch or a database search pipeline, which have their own scoring/filtering semantics — this threshold is calibrated specifically for ALPS de Bruijn assembly input.
- You need high-precision filtering well beyond 50% AA precision (e.g. for direct sequence reporting rather than assembly input); a simple precursor mass filter may be more appropriate.
- No database-search-derived reference peptides are available to compute AA precision, so tool-specific thresholds cannot be calibrated.

## Inputs

- Per-tool de novo prediction sets (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) with peptide sequences and tool-specific confidence scores
- Pseudo-ground-truth peptides from database search (MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR)

## Outputs

- filtered_predictions_per_tool.tsv with one filtered peptide set per tool (peptide sequence, confidence score, source tool), ready as input for ALPS de Bruijn assembly

## How to apply

Load the per-tool de novo prediction sets (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo), each record carrying a peptide sequence and a tool-specific confidence score. Build a pseudo-ground truth by combining MS-GF+ and X!Tandem results via PeptideShaker at 1% FDR. For each tool, sweep candidate confidence thresholds and compute amino-acid-level precision at each, counting an amino acid as matched when the mass difference is <0.1 Da and the prefix-mass difference is <0.5 Da; select the lowest threshold at which AA precision reaches 50%. Because each algorithm calculates its confidence score differently, thresholds must be derived per tool, not globally. Remove peptides below their tool's threshold and write the surviving peptides per tool to filtered_predictions_per_tool.tsv (peptide sequence, confidence score, source tool) as the cleaned input for ALPS assembly.

## Related tools

- **Novor** (De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision point)
- **pNovo 3** (De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision point)
- **DeepNovo** (De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision point)
- **SMSNet** (De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision point)
- **PointNovo** (De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision point)
- **Casanovo** (De novo sequencing tool whose prediction set is thresholded at the 50% AA-precision point; its predictions were the assembly input in the paper)
- **PeptideShaker** (Combines MS-GF+ and X!Tandem database-search results at 1% FDR to build the pseudo-ground truth used for precision calibration)
- **ALPS** (Downstream de Bruijn assembler that consumes the filtered peptide predictions)

## Evaluation signals

- filtered_predictions_per_tool.tsv exists and contains exactly one filtered peptide set per evaluated tool (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo)
- For every tool, re-computing amino-acid-level precision of the retained peptide set against the PeptideShaker 1% FDR pseudo-ground truth yields ≥ 50% under the matched-amino-acid criterion (mass difference <0.1 Da, prefix-mass difference <0.5 Da)
- Each tool's threshold is the lowest score cutoff at which AA precision first reaches 50%, not a shared numeric cutoff, since confidence scores are tool-specific
- No peptide in the output has a confidence score below its tool's selected threshold
- Downstream ALPS assembly on the filtered set produces the reported sequence coverage range (97.69–99.53% on light chains with Casanovo predictions)

## Limitations

- Each de novo algorithm calculates its confidence score differently, so thresholds are tool-specific and cannot be transferred between tools
- The 50% AA-precision calibration depends on the database-search pseudo-ground truth (1% FDR); errors in the reference propagate into the threshold
- Missing fragmentation sites, noisy spectra and long peptide sequences limit all de novo tools, so filtering alone cannot recover poor predictions
- Isoleucine/leucine and other isobaric residues cannot be resolved via MS/MS, capping achievable matching precision

## Evidence

- [other] per-tool threshold calibration: [evidence span withheld — anchor status: not_found]
- [other] matched-amino-acid criterion: [evidence span withheld — anchor status: not_found]
- [other] tool-specific scores: [evidence span withheld — anchor status: not_found]
- [other] pseudo-ground truth at 1% FDR: [evidence span withheld — anchor status: not_found]
- [abstract] downstream assembly coverage: "We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo."

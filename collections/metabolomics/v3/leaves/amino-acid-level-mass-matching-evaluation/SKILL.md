---
name: amino-acid-level-mass-matching-evaluation
description: Use when raw per-tool de novo prediction sets (peptide sequence + tool-specific
  confidence score) must be filtered to a precision-calibrated subset, using a pseudo-ground-truth
  peptide list from database search at 1% FDR.
when_to_use_negative:
- When the predictions will be evaluated at the whole-peptide level rather than the
  amino-acid level — the matched-AA criterion (mass difference <0.1 Da, prefix-mass
  difference <0.5 Da) will not reflect peptide-level recall or precision.
- When no database-search reference (PeptideShaker/MS-GF+/X!Tandem at 1% FDR) is available
  to serve as pseudo-ground truth for precision computation.
- When a database-search identification at 1% FDR is already the analysis product
  — thresholding de novo predictions is only needed to prepare inputs for database-independent
  assembly (e.g. ALPS).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Novor
  role: De novo prediction set whose confidence scores are swept to find the 50% AA-precision
    cutoff
- name: pNovo 3
  role: De novo prediction set whose confidence scores are swept to find the 50% AA-precision
    cutoff
- name: DeepNovo
  role: De novo prediction set whose confidence scores are swept to find the 50% AA-precision
    cutoff
- name: SMSNet
  role: De novo prediction set whose confidence scores are swept to find the 50% AA-precision
    cutoff
- name: PointNovo
  role: De novo prediction set whose confidence scores are swept to find the 50% AA-precision
    cutoff
- name: Casanovo
  role: De novo prediction set whose confidence scores are swept to find the 50% AA-precision
    cutoff; its predictions are the primary input to ALPS assembly
- name: PeptideShaker
  role: Combines MS-GF+ and X!Tandem results at 1% FDR to build the pseudo-ground-truth
    peptide reference used for precision computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:c1b496032fb5d7ac14c4a1b4e99454145afee00b816052acb6a855f1f8d2f15a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# amino-acid-level-mass-matching-evaluation

## Summary

Evaluates de novo peptide predictions (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) at the amino-acid level against a database-search pseudo-ground truth, then selects the per-tool confidence-score cutoff where AA-level precision reaches 50% before ALPS de Bruijn assembly.

## When to use

Use when raw per-tool de novo prediction sets (peptide sequence + tool-specific confidence score) must be filtered to a precision-calibrated subset, using a pseudo-ground-truth peptide list from database search at 1% FDR.

## When NOT to use

- When the predictions will be evaluated at the whole-peptide level rather than the amino-acid level — the matched-AA criterion (mass difference <0.1 Da, prefix-mass difference <0.5 Da) will not reflect peptide-level recall or precision.
- When no database-search reference (PeptideShaker/MS-GF+/X!Tandem at 1% FDR) is available to serve as pseudo-ground truth for precision computation.
- When a database-search identification at 1% FDR is already the analysis product — thresholding de novo predictions is only needed to prepare inputs for database-independent assembly (e.g. ALPS).

## Inputs

- Per-tool de novo prediction sets (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) with peptide sequence and tool-specific confidence score
- Pseudo-ground-truth peptides from database search (MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR)

## Outputs

- filtered_predictions_per_tool.tsv — one confidence-calibrated peptide set per tool, preserving peptide sequence, confidence score, and source tool, ready for ALPS de Bruijn assembly

## How to apply

Load the six de novo prediction sets and build a pseudo-ground-truth reference by combining MS-GF+ and X!Tandem results through PeptideShaker at 1% FDR. For each tool independently, sweep candidate confidence-score thresholds and compute amino-acid-level precision against the pseudo-ground truth using the matched-amino-acid criterion: a matched AA requires a mass difference <0.1 Da and a prefix-mass difference <0.5 Da. Because each algorithm computes its confidence score differently, select the lowest per-tool threshold at which AA-level precision is at least 50% — do not reuse a single cutoff across tools. Remove peptides scoring below their tool's threshold and write the surviving records (peptide sequence, confidence score, source tool) to filtered_predictions_per_tool.tsv as the cleaned input for ALPS de Bruijn assembly.

## Related tools

- **Novor** (De novo prediction set whose confidence scores are swept to find the 50% AA-precision cutoff)
- **pNovo 3** (De novo prediction set whose confidence scores are swept to find the 50% AA-precision cutoff)
- **DeepNovo** (De novo prediction set whose confidence scores are swept to find the 50% AA-precision cutoff)
- **SMSNet** (De novo prediction set whose confidence scores are swept to find the 50% AA-precision cutoff)
- **PointNovo** (De novo prediction set whose confidence scores are swept to find the 50% AA-precision cutoff)
- **Casanovo** (De novo prediction set whose confidence scores are swept to find the 50% AA-precision cutoff; its predictions are the primary input to ALPS assembly)
- **PeptideShaker** (Combines MS-GF+ and X!Tandem results at 1% FDR to build the pseudo-ground-truth peptide reference used for precision computation)

## Evaluation signals

- Re-evaluate each retained peptide set against the pseudo-ground truth and confirm amino-acid-level precision is at least 50% for every tool, mirroring the paper's filtering criterion.
- Confirm filtered_predictions_per_tool.tsv exists and contains exactly one filtered peptide set per tool (six tools), each record carrying peptide sequence, confidence score, and source tool.
- Verify the matched-amino-acid criterion was applied consistently: mass difference <0.1 Da and prefix-mass difference <0.5 Da for every counted match.
- Check that the pseudo-ground truth was built at 1% FDR via PeptideShaker combining MS-GF+ and X!Tandem before any threshold sweep.
- Confirm thresholds differ across tools — a single shared confidence-score cutoff across Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo indicates the per-tool derivation was skipped.

## Limitations

- Confidence scores are not comparable across algorithms — each tool calculates its score differently, so thresholds must be derived per tool and cannot be transferred.
- Mass-matching evaluation cannot resolve isoleucine vs leucine, Q vs AG, or deamidated N vs D, since these pairs are indistinguishable by MS/MS and require MS3 or additional methods.
- No tool version numbers, code, or data repository accessions were reported, limiting reproducibility of the threshold sweep.
- Advanced post-processing steps such as this filtering are acknowledged as necessary because raw de novo tool accuracy is otherwise insufficient.

## Evidence

- [methods] matched-AA precision computation: [evidence span withheld — anchor status: not_found]
- [methods] 50% AA precision threshold rationale: [evidence span withheld — anchor status: not_found]
- [abstract] evaluated de novo tools: "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
- [discussion] precision benefits of post-prediction filtering: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"

---
name: retrospective-challenge-benchmark-analysis
description: Use when a benchmark-contest solution set (e.g. CASMI challenges with
  known ground-truth molecules) must be re-analyzed after the fact to isolate the
  effect of a ranking or filtering parameter on correct-candidate rank.
when_to_use_negative:
- The input is a fresh prospective analysis with no known ground-truth structures
  — rank-improvement trends of the correct molecule cannot be evaluated.
- The candidate lists were not scored by MAGMa's substructure penalty score with refscore
  tie-breaking; refscore thresholds have no defined effect on other scoring schemes.
- The goal is de novo structure elucidation rather than benchmarking of an existing
  annotation workflow against challenge answers.
tools:
- name: MAGMa
  role: Produces the retrospective candidate rankings (substructure penalty scores,
    refscore ordering) that the threshold sweep re-analyzes; run in command line mode
    or via the web interface
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules and the refscore values used as the filtering
    threshold in the stepwise reduction
- name: emetabolomics web interface
  role: Alternative route to run MAGMa jobs and inspect per-challenge results for
    the retrospective analysis
  repo: http://www.emetabolomics.org/magma
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:08d0e750b465f98c890aae43845adc75aae92fe15b58adf046d5cfb2544de8de
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# retrospective-challenge-benchmark-analysis

## Summary

A retrospective benchmarking procedure in which MAGMa's submitted CASMI 2013 results (candidate rankings based on substructure penalty scores and PubChem refscore) are re-analyzed offline to quantify how processing parameters, especially refscore candidate-list filtering, affect the rank of the correct chemical structure.

## When to use

Use when a benchmark-contest solution set (e.g. CASMI challenges with known ground-truth molecules) must be re-analyzed after the fact to isolate the effect of a ranking or filtering parameter on correct-candidate rank.

## When NOT to use

- The input is a fresh prospective analysis with no known ground-truth structures — rank-improvement trends of the correct molecule cannot be evaluated.
- The candidate lists were not scored by MAGMa's substructure penalty score with refscore tie-breaking; refscore thresholds have no defined effect on other scoring schemes.
- The goal is de novo structure elucidation rather than benchmarking of an existing annotation workflow against challenge answers.

## Inputs

- MAGMa retrospective candidate rankings (penalty-score-ordered, refscore tie-broken) for CASMI 2013 challenges
- PubChem refscore values for each candidate molecule (local PubChem database of 2 December 2013)
- Ground-truth correct chemical structures for each challenge
- Challenge metadata (challenge IDs 2, 14, 11, 15, 16, 3; submitted vs. unsolved entries)

## Outputs

- Table of absolute correct-candidate ranks at refscore thresholds <1000, <500, <200 per challenge (e.g. refscore_filtering_table.csv)
- Counter-example annotation for challenges lost under filtering (challenge 3, N2-acetyl glutaminyl leucinamide, refscore 1)
- Parameter-influence notes for unsolved challenges (mass tolerance, in silico fragmentation depth)

## How to apply

Start from the retrospective candidate rankings and refscore values produced by MAGMa (candidates retrieved from a local PubChem database snapshot of 2 December 2013, fragmented by up to 3 bond dissociations, ranked by reversed intensity-weighted penalty score with ties broken by PubChem refscore). Stepwise raise the refscore threshold on each challenge until candidate sets are reduced to <1000, <500, and <200 molecules, and record the absolute rank of the correct candidate at each level for challenges such as 2, 14, 11, 15, 16, and 3. Tabulate the ranks so the trend — improved absolute ranking as lists shrink — is visible, and explicitly include counter-examples where filtering loses the correct molecule, e.g. challenge 3's N2-acetyl glutaminyl leucinamide (refscore 1). Supplement the parameter sweep with diagnostic notes on other misses, such as challenge 11's overly restrictive mass tolerance (10 ppm / 0.002 Da) and challenge 16's unexplained high-energy peaks under 3-bond dissociation.

## Related tools

- **MAGMa** (Produces the retrospective candidate rankings (substructure penalty scores, refscore ordering) that the threshold sweep re-analyzes; run in command line mode or via the web interface) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules and the refscore values used as the filtering threshold in the stepwise reduction)
- **emetabolomics web interface** (Alternative route to run MAGMa jobs and inspect per-challenge results for the retrospective analysis) — http://www.emetabolomics.org/magma

## Evaluation signals

- The tabulated ranks reproduce the reported trend: correct-molecule ranks improve in absolute terms as candidate lists shrink from <1000 to <500 to <200 molecules.
- The challenge 3 counter-example (N2-acetyl glutaminyl leucinamide, refscore 1) is present and shows the correct compound is lost when the list is reduced below 200 molecules.
- Baseline (unfiltered) results are consistent with the published outcome: 6 of 12 submitted category 2 solutions ranked the correct structure first, 3 ranked outside the top 5.
- Results are traceable to the same candidate universe as the paper (PubChem compounds on 2 December 2013) and the same scoring (reversed penalty ranking with refscore tie-break).
- Unsolved challenges are explained with the correct diagnostics: challenge 11 attributed to too-restrictive mass tolerance (10 ppm / 0.002 Da), challenge 16 to peaks unexplained by up to 3 bond dissociations.

## Limitations

- Refscore filtering can discard the correct molecule entirely, as with challenge 3's N2-acetyl glutaminyl leucinamide (refscore 1), so improved ranks must be weighed against loss events.
- Ranking among equally scoring candidates depends on the PubChem refscore, so the benchmark outcome is sensitive to the database snapshot date (2 December 2013) used for candidate retrieval.
- Retrospective conclusions about misses (e.g. mass-tolerance restrictiveness, insufficient in silico fragmentation for challenge 16) are interpretive hints, not guarantees for future contests.

## Evidence

- [results] Stepwise refscore threshold sweep: [evidence span withheld — anchor status: not_found]

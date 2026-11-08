---
name: screening-outcome-data-extraction
description: Use when a task requires reporting the numbers behind a published genetic
  screen (mutagenised genomes analysed, suppressor/non-CID strains obtained, long-lived
  mutants identified) for tabulation, cross-checking, or meta-analysis.
when_to_use_negative:
- The input is raw screening data (e.g. your own mutagenised strain library) rather
  than counts reported in a published article
- The article under curation reports only qualitative candidate gene identities (e.g.
  rcd-1, smg-1, smg-2, mtk-1, nsy-1, gst-13, sur-2) without quantitative screen statistics
- The task is to experimentally reproduce the screen rather than to extract and tabulate
  its published outcome numbers
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50111-8
    title: Regulatory mechanism of cold-inducible diapause in Caenorhabditis elegans
  source_content_hash: sha256:2426999679bdbc5a7c0fd385aacab418443faf87d763a15ed90a015f49221667
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# screening-outcome-data-extraction

## Summary

Extracts the quantitative outcome statistics of a published genetic screen — screen scale, intermediate yields, and confirmed hits — from the article's abstract/results text into a machine-readable table. Used here to capture the EMS suppressor screen of the hsf-1(sy441) cold-inducible diapause (CID) mutant.

## When to use

Use when a task requires reporting the numbers behind a published genetic screen (mutagenised genomes analysed, suppressor/non-CID strains obtained, long-lived mutants identified) for tabulation, cross-checking, or meta-analysis.

## When NOT to use

- The input is raw screening data (e.g. your own mutagenised strain library) rather than counts reported in a published article
- The article under curation reports only qualitative candidate gene identities (e.g. rcd-1, smg-1, smg-2, mtk-1, nsy-1, gst-13, sur-2) without quantitative screen statistics
- The task is to experimentally reproduce the screen rather than to extract and tabulate its published outcome numbers

## Inputs

- Published article text (DOI 10.1038/s41467-024-50111-8, Nature Communications 2024), specifically the abstract/results sections describing the EMS suppressor screen
- Screen statistics reported in the paper: F2 genome count, non-CID strain count, long-lived mutant count, lifespan extension percentages, and log-rank P-values
- Paper location identifiers (section/figure/table) for each reported metric

## Outputs

- Machine-readable CSV table with one row per reported screening metric
- Columns: value, unit/parameter, and exact paper location (section/figure/table) from which the value was taken

## How to apply

Locate the published article by DOI (10.1038/s41467-024-50111-8) and navigate to the Abstract and Results sections describing the EMS mutagenesis suppressor screen of hsf-1(sy441). Extract the three tiers of screening statistics: screening scale (~1,000,000 mutagenised F2 genomes analysed at 9 °C), intermediate yield (49 non-CID strains obtained), and confirmed hits (16 mutants that lived longer than hsf-1(sy441) background strains at 15 and 20 °C). Also extract per-mutant characterisation statistics, e.g. the sur-2(ku9) 15% lifespan extension and log-rank P-values (0.000036 against wild-type; 0.19 for rcd-1(hru150)). Assemble each metric as one row in a CSV with columns for value, unit/parameter, and the exact paper location (section/figure/table). Validation is by cross-check: the table is accepted only if every cell matches the reported number verbatim at its cited location.

## Evaluation signals

- The extracted counts match the article verbatim: ~1,000,000 F2 genomes analysed at 9 °C, 49 non-CID strains, and 16 long-lived mutants
- The sur-2(ku9) lifespan extension is recorded as exactly 15%
- Both log-rank P-values match verbatim: 0.000036 (against wild-type) and 0.19 (rcd-1(hru150))
- Every CSV row carries a concrete paper location (section/figure/table), not a generic citation
- A second-pass cross-check against the cited locations finds no discrepancy in any cell before the table is accepted

## Limitations

- Extraction is limited to what the article reports; underlying raw screen data (e.g. per-strain CID scores or lifespans) are not provided in the article text or the surfaced repositories
- The surfaced GitHub repository (fermo-metabolomics/FERMO) is a metabolomics dashboard unrelated to this C. elegans screen and supplies no screen data for this skill
- Numbers reported at different granularity (e.g. '~1,000,000' as an approximation) must be preserved as reported, not rounded or recomputed

## Evidence

- [abstract] 16 long-lived mutants among 49 non-CID mutants: "successfully isolate 16 long-lived mutants among 49 non-CID mutants via genetic screening"
- [discussion] Suppressor-screen strategy used to obtain long-lived mutants: "By selecting for suppressors of the CID phenotype of hsf-1(sy441) mutant, we obtained several long-lived mutants."
- [discussion] Candidate CID regulators isolated by the screen: [evidence span withheld — anchor status: not_found]
- [abstract] sur-2 nonsense mutation both suppresses CID and extends lifespan: "the nonsense mutation of MED23/sur-2 prevents CID entry of the hsf-1(sy441) mutant and extends lifespan"

---
name: recombination-foci-quantification
description: Use when reproducing or verifying reported per-nucleus recombination
  foci counts (GFP::RMH-1, GFP::MSH-5) across germline zones (2–4) for BTR-complex
  genotypes such as gfp::rmh-1, top-3-ZnF, and rmh-2; top-3-ZnF::ha from a Results
  text.
when_to_use_negative:
- The task requires re-counting foci from raw immunofluorescence images — this skill
  only transcribes and structures values already reported in the article text.
- The input text reports only qualitative descriptions (e.g. 'delayed and reduced
  in number') without explicit numeric means and SDs per zone.
- Statistics beyond transcription are needed (e.g. p-values, ANOVA) — these were performed
  separately in Prism 6/GraphPad, not by this extraction step.
edam_operation: null
tools:
- name: pandas
  role: Assemble the 16 genotype-zone-marker rows and write foci_means.csv
- name: Microsoft Excel
  role: Alternative spreadsheet tool for assembling and writing the foci means table
- name: GraphPad Prism
  role: Downstream statistics on the extracted foci counts (used in the study)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:481c478bb84ce776deaf3a4fbc6ba5e0056ded031692af4c8b31c894ad6df10f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphPad Prism
  - Microsoft Excel
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# recombination-foci-quantification

## Summary

Extracts and tabulates the reported mean numbers and SDs of recombination foci per nucleus (e.g. GFP::RMH-1 and GFP::MSH-5) across C. elegans germline zones, where zone 2 = early, zone 3 = mid, and zone 4 = late pachynema. It turns prose numbers from the Results text into a structured 16-row CSV for verification and downstream statistics.

## When to use

Use when reproducing or verifying reported per-nucleus recombination foci counts (GFP::RMH-1, GFP::MSH-5) across germline zones (2–4) for BTR-complex genotypes such as gfp::rmh-1, top-3-ZnF, and rmh-2; top-3-ZnF::ha from a Results text.

## When NOT to use

- The task requires re-counting foci from raw immunofluorescence images — this skill only transcribes and structures values already reported in the article text.
- The input text reports only qualitative descriptions (e.g. 'delayed and reduced in number') without explicit numeric means and SDs per zone.
- Statistics beyond transcription are needed (e.g. p-values, ANOVA) — these were performed separately in Prism 6/GraphPad, not by this extraction step.

## Inputs

- Article full text with a Results section containing focus-quantification passages (Figure 4D for GFP::RMH-1, Figure 5A for GFP::MSH-5)
- Zone scheme definition (zone 2 = early pachynema, zone 3 = mid pachynema, zone 4 = late pachynema)

## Outputs

- foci_means.csv with 16 data rows and columns marker, genotype, zone, mean_foci_per_nucleus, sd

## How to apply

Parse the Results section text to locate the focus-quantification passages for GFP::RMH-1 (Figure 4D) and GFP::MSH-5 (Figure 5A). Extract each explicit numeric mean and SD for the genotype-zone-marker combinations, using the zone scheme where zone 2 is early pachynema, zone 3 mid pachynema and zone 4 late pachynema. Assemble the 16 genotype-zone-marker rows into a table with columns marker, genotype, zone, mean_foci_per_nucleus and sd. Write the table to foci_means.csv using pandas or Microsoft Excel; GraphPad Prism 6 was used in the study for downstream statistics. Values to anchor on include GFP::RMH-1 zone 2 means of 15.2, 3.6, 1.6 and GFP::MSH-5 zone 2 means of 8.6, 2.3, 1.8, 0.5.

## Related tools

- **pandas** (Assemble the 16 genotype-zone-marker rows and write foci_means.csv)
- **Microsoft Excel** (Alternative spreadsheet tool for assembling and writing the foci means table)
- **GraphPad Prism** (Downstream statistics on the extracted foci counts (used in the study))

## Examples

```
import pandas as pd; df = pd.DataFrame(rows, columns=['marker','genotype','zone','mean_foci_per_nucleus','sd']); df.to_csv('foci_means.csv', index=False)
```

## Evaluation signals

- foci_means.csv exists, parses as CSV, and contains exactly 16 data rows.
- Extracted mean values match the numbers printed in the Results text, e.g. 15.2, 3.6, 1.6 for GFP::RMH-1 zone 2 and 8.6, 2.3, 1.8, 0.5 for GFP::MSH-5 zone 2.
- SD columns match the reported ±SD values, e.g. ±6.9 SD for gfp::rmh-1 and ±3.8 SD for gfp::rmh-1; top-3-ZnF in zone 2 for GFP::RMH-1.
- Each row resolves to exactly one (marker, genotype, zone) triple covering GFP::RMH-1 for gfp::rmh-1 (zones 2–3), gfp::rmh-1; top-3-ZnF (zones 2–3), rmh-2(jf92); top-3-ZnF::ha (zones 2–4), and the analogous GFP::MSH-5 combinations.

## Limitations

- Values are transcribed from prose; the article reports large standard deviations for some measurements (e.g. highly variable RMH-2::FLAG expression levels with large SDs in the top-3-ZnF background), so transcription errors in SDs are easy to make.
- No mention of biological replicates or sample sizes for the described experiments in the provided text, so the 16 means cannot be quality-checked against n.
- The study used Prism 6 for downstream statistics; the CSV itself carries no statistical test results.

## Evidence

- [results] zone-2 mean GFP::RMH-1 foci: [evidence span withheld — anchor status: not_found]
- [discussion] zone scheme and early foci reduction: "Early TOP-3-containing meiotic recombination foci (seen in zones 2 and 3 in the wt) were delayed and reduced in number, but the number and timing of late foci (in late pachynema) were not different"
- [discussion] RMH-2::FLAG variability caveat: "Western blot analysis of RMH-2::FLAG whole worm extracts of the top-3-ZnF; rmh-2::flag mutant revealed highly variable expression levels"

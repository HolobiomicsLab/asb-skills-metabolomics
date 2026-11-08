---
name: meiotic-recombination-focus-count-extraction
description: Use when reproducing reported TOP-3::HA (or related recombination-marker)
  focus counts per nucleus across C. elegans germline zones 2–4 from published Results
  text, e.g. for the rtel-1 sensitized background.
when_to_use_negative:
- You need raw per-nucleus focus counts or new image-based counting from microscopy
  — this skill only transcribes reported zone-level means and SDs.
- The analysis concerns replication defects, crossover counts, or GFP::MSH-5 localization
  rather than TOP-3::HA focus numbers per germline zone.
- Reported values are given as figures without accompanying zone-wise mean/SD text,
  since the workflow depends on verbatim numeric extraction.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: pandas
  role: Compiles the extracted genotype/zone/mean/SD values into a single validated
    CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:ccd9b4634ef7f3cfaaab2b83f29b7b28b5ea7b09a68a22b8c15f378a567a8449
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# meiotic-recombination-focus-count-extraction

## Summary

Extracts published TOP-3::HA recombination-focus counts (mean ± SD foci per nucleus) for germline zones 2–4 across top-3::ha, top-3-ZnF::ha, rtel-1 and rmh-2 backgrounds from the article's Results text, and compiles them into a single pandas CSV.

## When to use

Use when reproducing reported TOP-3::HA (or related recombination-marker) focus counts per nucleus across C. elegans germline zones 2–4 from published Results text, e.g. for the rtel-1 sensitized background.

## When NOT to use

- You need raw per-nucleus focus counts or new image-based counting from microscopy — this skill only transcribes reported zone-level means and SDs.
- The analysis concerns replication defects, crossover counts, or GFP::MSH-5 localization rather than TOP-3::HA focus numbers per germline zone.
- Reported values are given as figures without accompanying zone-wise mean/SD text, since the workflow depends on verbatim numeric extraction.

## Inputs

- Article Results text quoting TOP-3::HA foci per nucleus for Figures 3A/3B and 6A
- Genotype and germline-zone labels (zones 2–4: early, mid, late pachynema)

## Outputs

- CSV with 13 data rows and columns genotype, zone, mean_foci_per_nucleus, sd
- Empty cells for rmh-2(jf94) zones 3–4 (single zone-2 value 9.4 ± 6)

## How to apply

Read the Results text describing Figures 3A/3B and 6A and extract the quoted mean and SD TOP-3::HA foci per nucleus for each genotype and zone: top-3::ha (zone 2: 14.7 ± 7.8; zone 3: 22.1 ± 6.2; zone 4: 8), top-3-ZnF::ha (5.5 ± 5; 14.8 ± 4.1; 7.9 ± 1.4), rtel-1; top-3::ha (12.9 ± 7.9; 27.1 ± 7.3; 11.1 ± 6.6), rtel-1; top-3-ZnF::ha (8.8 ± 6.5; 17.1 ± 5.7; 7.2 ± 1.2), plus the single rmh-2(jf94) zone-2 value (9.4 ± 6) with zones 3–4 left empty. Use pandas to compile genotype, zone, mean_foci_per_nucleus and sd columns into one CSV. Validate that the CSV has exactly 13 data rows and that each mean/SD pair matches the Results text verbatim. The rationale is that the paper reports focus quantification per pachytene zone (early, mid, late pachynema), so a tidy genotype-by-zone table is the correct unit for cross-genotype comparison.

## Related tools

- **pandas** (Compiles the extracted genotype/zone/mean/SD values into a single validated CSV)

## Examples

```
import pandas as pd; df = pd.DataFrame(rows, columns=['genotype','zone','mean_foci_per_nucleus','sd']); df.to_csv('top3_ha_foci.csv', index=False)
```

## Evaluation signals

- Output CSV contains exactly 13 data rows.
- Every mean/SD pair matches the values quoted verbatim in the Results text (e.g. rtel-1; top-3-ZnF::ha: 8.8 ± 6.5, 17.1 ± 5.7, 7.2 ± 1.2 for zones 2–4).
- rmh-2(jf94) has only the zone-2 value (9.4 ± 6); zones 3–4 are empty, giving the expected sparse row pattern.
- Columns are exactly genotype, zone, mean_foci_per_nucleus, sd.
- Zone 2–4 ordering is preserved for every genotype (early, mid, late pachynema).

## Limitations

- rmh-2(jf94) contributes only a single zone-2 value (9.4 ± 6); zones 3–4 cannot be populated from the reported text.
- The counts are manual extractions from Results prose — the article provides no raw per-nucleus data or code/data availability statement for re-derivation.
- Reported SDs can be large and variability is noted for related constructs (e.g. highly variable RMH-2::FLAG expression levels with large standard deviations in the top-3-ZnF background), so differences between genotypes should be interpreted cautiously.

## Evidence

- [results] rtel-1; top-3-ZnF::ha foci: [evidence span withheld — anchor status: not_found]
- [discussion] zone-wise focus dynamics: "Early TOP-3-containing meiotic recombination foci (seen in zones 2 and 3 in the wt) were delayed and reduced in number"
- [intro] rtel-1 double mutant rationale: "To understand whether the ZnF domain contributes to D-loop discouragement, we constructed the top-3-ZnF; rtel-1 double mutant."
- [discussion] immunofluorescence basis of counts: "Our immunofluorescence data revealed that, although recombination is initiated on time, the processing of intermediates is delayed"

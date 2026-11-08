---
name: msi-metabolite-database-interpretation
description: Use when metabolite annotations from ambient MSI (e.g., AFADESI-MSI of
  mouse body sections) must be organized into per-organ membership/enrichment sets
  and compared across organs, ionization modes, or mouse strains.
when_to_use_negative:
- Input is raw, unannotated MSI peak data — run MSIannotator.m first to obtain metabolite/adduct
  annotations before set-based organ comparison.
- The metabolite list comes from a single organ or tissue with no multi-organ panel
  — per-organ set-difference/intersection analysis needs the nine organ/serum sheets.
- You need m/z-level peak matching rather than interpreting already-annotated metabolite
  names — that is MSIannotator's 10 ppm matching step, not this interpretation skill.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MATLAB
  role: Environment in which the database sheets are imported as string arrays and
    set operations (membership sets, set-difference, intersection) are performed per
    organ and ionization mode
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator
  role: Upstream MATLAB tool that annotates metabolites and adducts in processed MSI
    data against the organ database, producing the annotated lists this interpretation
    step consumes
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: Script executed in MATLAB to generate the 'Annotator' annotation file (metabolite
    names plus adduct forms) using a default 10 ppm m/z error range
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:b7589b994684e5e292665fe93e46b8c63af7665536e7ddaac3b88c416dcde71e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# msi-metabolite-database-interpretation

## Summary

Interprets organ-specific metabolite databases (1298 positive-mode and 515 negative-mode metabolites, Sheets 1–11 of 'Organ specific metabolite database.xlsx') using MATLAB set operations to determine whether metabolites are distributed organ-specifically across the whole mouse body and whether that pattern is conserved across mouse strains.

## When to use

Use when metabolite annotations from ambient MSI (e.g., AFADESI-MSI of mouse body sections) must be organized into per-organ membership/enrichment sets and compared across organs, ionization modes, or mouse strains.

## When NOT to use

- Input is raw, unannotated MSI peak data — run MSIannotator.m first to obtain metabolite/adduct annotations before set-based organ comparison.
- The metabolite list comes from a single organ or tissue with no multi-organ panel — per-organ set-difference/intersection analysis needs the nine organ/serum sheets.
- You need m/z-level peak matching rather than interpreting already-annotated metabolite names — that is MSIannotator's 10 ppm matching step, not this interpretation skill.

## Inputs

- 'Organ specific metabolite database.xlsx' — 11 sheets: Sheet1 (1298 positive-mode metabolites), Sheet2 (515 negative-mode metabolites), Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum)
- Metabolite annotation output 'Annotator' from MSIannotator.m (annotated metabolite names and adduct forms)
- Per-strain MSI annotation sets for the two mouse strains being compared

## Outputs

- Per-organ enriched/unique metabolite sets (set-difference and intersection results across the nine organ/serum sheets)
- Summary table exported as organ_specific_distribution.csv with organ, mode, metabolite, and adduct columns
- Cross-strain comparison confirming conservation of the organ-specific distribution pattern

## How to apply

Load the nine organ/serum sheets (Sheet3–Sheet11: brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum) from 'Organ specific metabolite database.xlsx' into MATLAB as string arrays, covering 1298 positive-mode (Sheet1) and 515 negative-mode (Sheet2) metabolites. Build metabolite membership sets per organ and ionization mode from the first-column metabolite names, tracking the adduct ion forms listed per metabolite. Compute per-organ enriched and unique metabolite sets via set-difference and intersection analysis across the nine sheets — the article reports 1620 unique metabolites across eight organs plus serum, drawn from a database of 27,407 adduct ions. Repeat the set construction for both mouse strains and verify the organ-specific distribution pattern is conserved (the reported finding: metabolites were distributed across the body in an organ-specific manner spanning different mouse strains). Export a summary table with organ, mode, metabolite, and adduct columns.

## Related tools

- **MATLAB** (Environment in which the database sheets are imported as string arrays and set operations (membership sets, set-difference, intersection) are performed per organ and ionization mode) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator** (Upstream MATLAB tool that annotates metabolites and adducts in processed MSI data against the organ database, producing the annotated lists this interpretation step consumes) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (Script executed in MATLAB to generate the 'Annotator' annotation file (metabolite names plus adduct forms) using a default 10 ppm m/z error range) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
msi = readtable('MSIdata.xlsx'); db = readtable('Database.xlsx'); % run MSIannotator.m (10 ppm default), then in MATLAB: enriched{liver} = setdiff(liverSet, intersect(brainSet, kidneySet)); writetable(summaryTbl, 'organ_specific_distribution.csv');
```

## Evaluation signals

- Per-organ metabolite counts computed from Sheet3–Sheet11 sets match the source sheets (consistent with 1620 unique metabolites from eight organs plus serum).
- Set operations cover both ionization modes: 1298 positive-mode (Sheet1) and 515 negative-mode (Sheet2) metabolites are represented.
- Cross-strain comparison reproduces the reported finding that organ-specific distribution spans different mouse strains.
- Exported organ_specific_distribution.csv contains organ, mode, metabolite, and adduct columns with adduct forms traced per metabolite.
- Each metabolite in a per-organ set carries a defined adduct ion form consistent with the database columns (theoretical m/z in specified adduct forms).

## Limitations

- Annotations are only as reliable as the organ database and mode choice — the README requires that the selected organ database and detection mode (positive or negative) match the MSIdata being annotated.
- Set-based organ specificity depends on the curated database's coverage (27,407 adduct ions) and does not itself validate spatial assignment at the pixel level.
- Adduct-form differences between LC-MS and AFADESI-MSI must be accounted for when merging annotations sourced from LC-MS/MS-derived databases.

## Evidence

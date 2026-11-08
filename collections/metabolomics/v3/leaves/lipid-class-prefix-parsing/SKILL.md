---
name: lipid-class-prefix-parsing
description: Use when a feature table of lipid identifications (e.g., Additional file
  3 Table S4 LipidMatch annotations) lacks an explicit class column and you need distinct
  lipid ion, feature, or lipid-type counts to verify reported numbers.
when_to_use_negative:
- The annotation table already contains a separate, explicit lipid class/type column
  — count unique values directly instead of parsing prefixes
- Annotation strings do not follow lipid-abbreviation syntax (e.g., free-text compound
  names or InChIKeys), where a leading-token regex would not yield the class
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidMatch
  role: Source of the rule-based lipid identifications whose annotation strings are
    parsed for lipid-class prefixes; appends identifications to each feature table
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: pandas
  role: Filtering rows to LipidMatch annotations and applying the class-prefix extraction/unique-counting
    over the worksheet
- name: R
  role: Language in which LipidMatch itself is implemented, generating the annotation
    feature tables that this skill consumes
  repo: http://secim.ufl.edu/secim-tools/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:1c7d77b660fb7c76daefd2ec1e60ebf15defffa028df5942f0d47bcb3e3cf4d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - R
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lipid-class-prefix-parsing

## Summary

Derives lipid-class (lipid type) counts from lipid annotation strings—e.g., LipidMatch identifications in a feature table—by parsing the leading class prefix (CE, Cer, LPC, PE, PG, PI, PS, SM, TAG, etc.) so annotation richness can be compared against published figures such as the 15 lipid types reported for Red Cross plasma negative-mode data.

## When to use

Use when a feature table of lipid identifications (e.g., Additional file 3 Table S4 LipidMatch annotations) lacks an explicit class column and you need distinct lipid ion, feature, or lipid-type counts to verify reported numbers.

## When NOT to use

- The annotation table already contains a separate, explicit lipid class/type column — count unique values directly instead of parsing prefixes
- Annotation strings do not follow lipid-abbreviation syntax (e.g., free-text compound names or InChIKeys), where a leading-token regex would not yield the class

## Inputs

- Additional file 3 Table S4 negative-polarity worksheet (xlsx/csv) with LipidMatch, MS-DIAL, and Greazy annotation columns for Red Cross plasma features
- LipidMatch-appended feature table rows keyed by m/z and retention time
- Lipid annotation strings in lipid-abbreviation form (e.g., 'LPC 16:0', 'PE 38:4')

## Outputs

- verification_summary.csv listing recounted vs. published values (210 lipid ions, 159 features, 15 lipid types) with pass/fail flags
- Distinct lipid-type list parsed from annotation prefixes (e.g., CE, Cer, LPC, LPE, PA, PE, PG, PI, PS, SM, TAG)

## How to apply

Open the polarity-specific worksheet (e.g., the negative-polarity sheet of Additional file 3 Table S4, which contains LipidMatch, MS-DIAL, and Greazy annotations for Red Cross plasma) and filter to the LipidMatch annotation columns using pandas. Count distinct lipid ions as unique lipid identification entries, and count distinct features as unique m/z + retention-time pairs carrying at least one annotation. Parse the lipid class by extracting the leading alphabetic prefix of each annotation string (e.g., regex '^([A-Za-z]+)' applied to the LipidMatch column), then take unique prefixes as lipid types. Because LipidMatch's in silico libraries span 56 lipid types (per the 2017 paper; the current README cites over 60), expect the Red Cross plasma negative-mode subset to cover only a portion of the library vocabulary. Write recounted vs. published figures (210 lipid ions, 159 features, 15 lipid types) with pass/fail flags to verification_summary.csv.

## Related tools

- **LipidMatch** (Source of the rule-based lipid identifications whose annotation strings are parsed for lipid-class prefixes; appends identifications to each feature table) — https://github.com/GarrettLab-UF/LipidMatch
- **pandas** (Filtering rows to LipidMatch annotations and applying the class-prefix extraction/unique-counting over the worksheet)
- **R** (Language in which LipidMatch itself is implemented, generating the annotation feature tables that this skill consumes) — http://secim.ufl.edu/secim-tools/

## Examples

```
import pandas as pd; df = pd.read_excel('TableS4.xlsx', sheet_name='negative'); lm = df.dropna(subset=['LipidMatch']); lm['lipid_type'] = lm['LipidMatch'].str.extract(r'^([A-Za-z]+)'); print(lm['LipidMatch'].nunique(), lm.groupby(['mz','RT']).ngroups, lm['lipid_type'].nunique())
```

## Evaluation signals

- Recounted distinct lipid ions in negative polarity equals the published 210
- Recounted distinct features (unique m/z + retention-time pairs) equals the published 159
- Recounted distinct lipid types equals the published 15
- Parsed type set includes expected Red Cross plasma classes (e.g., CE, Cer, LPC, LPE, PA, PE, PG, PI, PS, SM, TAG) and contains no empty/NaN prefixes after extraction
- Every annotation row maps to exactly one lipid type, so summed per-type counts equal the total annotated-row count

## Limitations

- Counts depend on selecting the correct polarity worksheet and excluding MS-DIAL/Greazy columns from Additional file 3 Table S4; mixing polarities or software inflates the tallies
- Lipid-type vocabulary shifts with library version — the 2017 paper describes 56 lipid types while the current README cites over 60 — so parsed type sets may not match the paper's 15-type subset exactly if library versions differ

## Evidence

- [methods] LipidMatch libraries cover 56 lipid types: "With 56 lipid types, LipidMatch in silico libraries cover the greatest number of lipid types of any open source software to date, with MS-DIAL containing 34 lipid types, and Greazy containing 24"
- [abstract] 98% negative-mode corroboration: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [methods] Identifications appended to feature tables: "LipidMatch will automatically append identifications to each feature table using MS/MS files contained in that feature table’s subfolder."
- [readme] README library scale: "using in-silico fragmentation libraries of over 500,000 lipid species across over 60 lipid types making it one of the most comprehensive open-source software"
- [readme] Multi-software annotation combination: "LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software."

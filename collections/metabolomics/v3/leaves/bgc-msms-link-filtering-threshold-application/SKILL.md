---
name: bgc-msms-link-filtering-threshold-application
description: Use when a Metcalf-scored list of candidate BGC–MS/MS spectrum links
  must be reduced and prioritised using genome-based (BiG-SCAPE/antiSMASH) versus
  structure-based (CANOPUS/MolNetEnhancer) chemical class agreement.
when_to_use_negative:
- When candidate links have not yet been scored by standardised Metcalf scoring —
  the NPClassScore filter is applied downstream of Metcalf, not instead of it.
- When metabolites exceed the ~850 Da CANOPUS mass cut-off or spectra lack class predictions
  and retaining them matters — the article notes Spec2Vec/MS2DeepScore may be better
  suited for bigger metabolites, and not filtering missing-score links changes the
  result (average candidate links per GCF decrease by 49% when kept).
- When building the scoring tables themselves rather than applying them — table construction
  is handled by the mibig_classifications notebooks, not this filtering step.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Platform implementing the NPClassScore scoring and 0.25 cut-off filtering
    of Metcalf-scored BGC–MS/MS links
  repo: github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical class-matching score whose precomputed MIBiG 2.0 tables supply the
    maximum relative frequency per (GCF class, spectrum class) pair
  repo: github.com/louwenjjr/mibig_classifications
- name: CANOPUS
  role: Predicts structure-based classes directly from MS/MS spectra (run via Sirius
    v4.9.3 on the GNPS mgf) so links can be scored
  repo: github.com/kaibioinfo/canopus_treemap
- name: MolNetEnhancer
  role: Alternative source of structure-based classes; must be run externally on the
    GNPS platform
- name: BiG-SCAPE
  role: Groups BGCs into GCFs whose genome-based classes feed the scoring lookup
- name: antiSMASH
  role: Provides BGC (sub)classes used as the genome-based side of the class matching
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:a5c6fa9bd34e42a628175e5b4ca48a5e7c0fbd24b030abc8dda96c9c22e3e88a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - CANOPUS
  - MolNetEnhancer
  - NPClassScore
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bgc-msms-link-filtering-threshold-application

## Summary

Application of the NPClassScore chemical class-matching cut-off (0.25) to filter candidate BGC–MS/MS spectrum links produced by standardised Metcalf scoring (cut-off 2.5) within the NPLinker workflow, reducing false-positive links by chemical class consistency.

## When to use

Use when a Metcalf-scored list of candidate BGC–MS/MS spectrum links must be reduced and prioritised using genome-based (BiG-SCAPE/antiSMASH) versus structure-based (CANOPUS/MolNetEnhancer) chemical class agreement.

## When NOT to use

- When candidate links have not yet been scored by standardised Metcalf scoring — the NPClassScore filter is applied downstream of Metcalf, not instead of it.
- When metabolites exceed the ~850 Da CANOPUS mass cut-off or spectra lack class predictions and retaining them matters — the article notes Spec2Vec/MS2DeepScore may be better suited for bigger metabolites, and not filtering missing-score links changes the result (average candidate links per GCF decrease by 49% when kept).
- When building the scoring tables themselves rather than applying them — table construction is handled by the mibig_classifications notebooks, not this filtering step.

## Inputs

- Metcalf-scored candidate BGC–MS/MS spectrum link list (from NPLinker get_links)
- Genome-based chemical classes for each GCF (derived from BiG-SCAPE and antiSMASH outputs)
- Structure-based chemical classes for each MS/MS spectrum (CANOPUS via Sirius v4.9.3 on the GNPS mgf, or MolNetEnhancer predictions)
- MIBiG 2.0-derived precomputed (genome-based class, structure-based class) scoring tables of relative frequencies

## Outputs

- Filtered and scored link list with NPClassScore values per BGC–MS/MS link
- Machine-readable table of retained links after the 0.25 NPClassScore cut-off

## How to apply

First generate candidate links with standardised Metcalf scoring at a cut-off of 2.5 (lowered to 2 only if validated links in a small dataset score below 2.5, as happened for the Nocardia dataset). For each candidate link, look up the pair of genome-based classes (from BiG-SCAPE/antiSMASH annotations of the GCF) and structure-based classes (CANOPUS run via Sirius v4.9.3 on the GNPS mgf, or MolNetEnhancer run externally on the GNPS platform) in the MIBiG 2.0-derived precomputed scoring tables, and assign the maximum relative frequency as the NPClassScore value. Filter out links scoring below the default cut-off of 0.25; optionally use the filter_missing_scores toggle to discard links whose spectra lack a CANOPUS or MolNetEnhancer prediction. Rationale: MIBiG 2.0 SMILES were classified with ClassyFire and NPClassifier and matched to MIBiG BGC (sub)classes, so the score measures how frequently a matched class pair co-occurs in validated BGC-product pairs.

## Related tools

- **NPLinker** (Platform implementing the NPClassScore scoring and 0.25 cut-off filtering of Metcalf-scored BGC–MS/MS links) — github.com/NPLinker/nplinker
- **NPClassScore** (Empirical class-matching score whose precomputed MIBiG 2.0 tables supply the maximum relative frequency per (GCF class, spectrum class) pair) — github.com/louwenjjr/mibig_classifications
- **CANOPUS** (Predicts structure-based classes directly from MS/MS spectra (run via Sirius v4.9.3 on the GNPS mgf) so links can be scored) — github.com/kaibioinfo/canopus_treemap
- **MolNetEnhancer** (Alternative source of structure-based classes; must be run externally on the GNPS platform)
- **BiG-SCAPE** (Groups BGCs into GCFs whose genome-based classes feed the scoring lookup)
- **antiSMASH** (Provides BGC (sub)classes used as the genome-based side of the class matching)

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Filtering should reduce the number of candidate BGC–MS/MS links by on average ~63% relative to co-occurrence (Metcalf) scoring alone.
- 96% of experimentally validated links in the three PoDP datasets (MSV000078836, MSV000078839, MSV000079284) should be retained and prioritised after filtering.
- No retained link in the output table should have an NPClassScore below 0.25.
- Links whose spectra lack CANOPUS or MolNetEnhancer predictions should be absent when filter_missing_scores is enabled (1988 such spectra existed across the datasets).
- Ranked positions of validated links should be recorded before and after filtering to confirm prioritisation, not just removal.

## Limitations

- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction; discarding them is controlled by filter_missing_scores, and not filtering decreases the average candidate links per GCF by only 49% rather than 63%.
- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring the Metcalf threshold to be lowered from 2.5 to 2 to include them.
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally, e.g. on the GNPS platform.
- CANOPUS coverage is limited by an 850 Da mass cut-off; larger metabolites are not well handled and mass-spectral embedding methods (Spec2Vec, MS2DeepScore) may be better suited.
- Genome inputs for antiSMASH 6 were merged with antiSMASH 3 output for 50 other strains, introducing version inconsistency in BGC classes.

## Evidence

- [discussion] Metcalf cut-off then NPClassScore cut-off: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] Average 63% link reduction: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [abstract] 96% validated links retained: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] Discarding links without structure-based classes: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [readme] Relative counts basis of matched class pairs (README): [evidence span withheld — anchor status: external_doc]

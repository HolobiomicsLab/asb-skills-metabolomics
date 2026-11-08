---
name: lipidblast-in-silico-spectral-matching
description: Use when annotating lipid-class features in untargeted AIF LC–MS/MS datasets
  where no experimental MS/MS spectra are available and in-silico coverage of lipids
  is needed.
when_to_use_negative:
- Input already consists of experimental MS/MS spectra that can be matched directly
  against METLIN/MassBank/GNPS spectral databases without in-silico emulation.
- The biological matrix is dominated by nonlipid metabolites and a nonlipid in-silico
  library is required, since low recall (10–88%) is expected with the small nonlipid
  fragment libraries.
- The aim is to annotate tens of thousands of features in an untargeted manner without
  prior statistical feature selection; the workflow focuses on a subset of features
  rather than the full dataset.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: MetaboAnnotatoR
  role: Performs correlation-based parent–fragment linking and matches resulting fragment
    lists against LipidBlast to score lipid annotations
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: Alternative deconvolution tool that reconstructs pseudo-MS/MS spectra via
    MS2Dec and performs LipidBlast spectral matching
- name: RAMClustR
  role: R package used to reconstruct in-source parent-fragment pseudo-MS/MS spectra
    prior to LipidBlast matching
- name: CAMERA
  role: R package used to reconstruct in-source parent-fragment pseudo-MS/MS spectra
    prior to LipidBlast matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:fec3ae19a57f92d4a065a5386781fadc5af99c6257560d1dd86f95936330902d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - CAMERA
  - GNPS
  - LipidBlast
  - METLIN
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  - RAMClustR
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lipidblast-in-silico-spectral-matching

## Summary

Annotate untargeted LC–MS features by matching reconstructed pseudo-MS/MS or fragment-peak m/z lists against LipidBlast, an in-silico lipid MS/MS library containing over 1 million spectra, as used by MetaboAnnotatoR to score lipid annotations in AIF datasets.

## When to use

Use when annotating lipid-class features in untargeted AIF LC–MS/MS datasets where no experimental MS/MS spectra are available and in-silico coverage of lipids is needed.

## When NOT to use

- Input already consists of experimental MS/MS spectra that can be matched directly against METLIN/MassBank/GNPS spectral databases without in-silico emulation.
- The biological matrix is dominated by nonlipid metabolites and a nonlipid in-silico library is required, since low recall (10–88%) is expected with the small nonlipid fragment libraries.
- The aim is to annotate tens of thousands of features in an untargeted manner without prior statistical feature selection; the workflow focuses on a subset of features rather than the full dataset.

## Inputs

- AIF LC–MS/MS chromatograms acquired/transformed in centroid mode
- Pseudo-MS/MS spectra reconstructed from parent–fragment ion relationships
- Feature m/z–RT lists (e.g. from XCMS, RAMClustR, or MS-DIAL MS2Dec)
- LipidBlast in-silico lipid spectral library (over 1 million spectra)

## Outputs

- Ranked lipid annotation assignments per feature (rank 1–5 scores)
- Precision and recall metrics evaluated against manually annotated expert lists
- Overlaps of automated annotations with manual annotations (e.g. 121/161 for Lipid+ vs 89/161 for MS-DIAL)

## How to apply

Run feature detection on the AIF chromatograms and reconstruct parent–fragment ion relationships (e.g. via MetaboAnnotatoR correlation-based parent–fragment linking or MS-DIAL MS2Dec deconvolution) to build pseudo-MS/MS spectra for each feature. Search these spectra — or the raw fragment m/z lists extracted from MS scans — against LipidBlast in-silico lipid spectra, which contain over 1 million spectra covering lipid classes. Assign a rank score to each match and use the rank distribution to evaluate precision (82–92%) and recall (82–85%) at ranks 1–5 for human serum datasets. If nonlipid metabolites dominate, expect modest-to-good precision (60–88%) but low recall because the nonlipid fragment library is small; consider supplementing LipidBlast with METLIN, MassBank, GNPS or custom .txt/.msp fragment libraries.

## Related tools

- **MetaboAnnotatoR** (Performs correlation-based parent–fragment linking and matches resulting fragment lists against LipidBlast to score lipid annotations) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (Alternative deconvolution tool that reconstructs pseudo-MS/MS spectra via MS2Dec and performs LipidBlast spectral matching)
- **RAMClustR** (R package used to reconstruct in-source parent-fragment pseudo-MS/MS spectra prior to LipidBlast matching)
- **CAMERA** (R package used to reconstruct in-source parent-fragment pseudo-MS/MS spectra prior to LipidBlast matching)

## Examples

```
library(MetaboAnnotatoR); annot <- annotateAIF(xcms_features, pseudospectra = pseudo_MSMS, fragmentLib = LipidBlast, rankThreshold = 5)
```

## Evaluation signals

- Precision at ranks 1–5 for human serum datasets should fall in the 82–92% range.
- Recall at ranks 1–5 for human serum datasets should fall in the 82–85% range.
- Lipid annotations should overlap with manually curated expert annotation lists, e.g. 121/161 features for Lipid+ or 37/50 for HILIC+ when compared against MS-DIAL reference annotations.
- Match rank distribution should show a concentration of rank-1 hits for well-covered lipid classes, with a tail of lower ranks reflecting library size limitations.

## Limitations

- Low recall (10–88%) for matrices dominated by nonlipid metabolites, attributable to the small size of nonlipid fragment libraries.
- Incomplete pseudo-MS/MS spectra may arise for co-eluting compounds with similar chemical groups, producing nonspecific fragment ions that reduce matching quality.
- Orbitrap-based validation (higher mass accuracy instruments) was not performed; annotation of such datasets is possible but untested.
- Only AIF DIA schemes are supported; SWATH and other DIA schemes are not yet implemented.

## Evidence

- [intro] The default lipidomics databases (LipidBlast35) containing over 1 million spectra: "The default lipidomics databases (LipidBlast35) containing over 1 million spectra"
- [abstract] Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5): "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [discussion] good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5).: "good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5)."
- [readme] the fragment libraries from Github (https://github.com/gggraca/MetaboAnnotatoR): "together with the fragment libraries from Github (https://github.com/gggraca/MetaboAnnotatoR)"

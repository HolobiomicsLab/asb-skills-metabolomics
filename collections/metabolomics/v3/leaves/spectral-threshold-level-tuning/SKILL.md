---
name: spectral-threshold-level-tuning
description: Use when a processed 2D or pseudo-3D NMR spectrum needs publication-quality
  contour rendering or 3D surface visualization, and the contour threshold/levels
  must be matched to the spectrum's intensity range.
when_to_use_negative:
- Input is a raw Bruker time-domain FID (`fid`, `ser`) that still needs WebAssembly
  processing (phase correction, apodization, zero filling, Fourier transformation)
  before contour rendering
- The task is quantitative peak deconvolution/line-shape fitting (DEEP Picker / Voigt
  Fitter), not visual inspection and rendering
- Spectrum is 1D — use the 1D viewing mode rather than 2D contour/3D mesh rendering
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: COLMARvista
  role: WebGL-based renderer used to set contour threshold/levels and produce 2D contour
    and 3D surface mesh images of processed spectra
  repo: https://github.com/lidawei1975/colmarvista
- name: JavaScript
  role: Underlying technology making the graphics rendering browser-based and OS-independent
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:a4226e7932e195dc424942f3cad4a5d222e05a56dd9daedef405383ac11e6cb3
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - JavaScript
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spectral-threshold-level-tuning

## Summary

Tune contour threshold and contour levels in COLMARvista for a processed 2D NMR spectrum, then render the spectrum with WebGL 2D contours and as a 3D surface mesh, exporting images of both for visual cross-checking of peak content.

## When to use

Use when a processed 2D or pseudo-3D NMR spectrum needs publication-quality contour rendering or 3D surface visualization, and the contour threshold/levels must be matched to the spectrum's intensity range.

## When NOT to use

- Input is a raw Bruker time-domain FID (`fid`, `ser`) that still needs WebAssembly processing (phase correction, apodization, zero filling, Fourier transformation) before contour rendering
- The task is quantitative peak deconvolution/line-shape fitting (DEEP Picker / Voigt Fitter), not visual inspection and rendering
- Spectrum is 1D — use the 1D viewing mode rather than 2D contour/3D mesh rendering

## Inputs

- Processed 2D or pseudo-3D NMR spectrum (e.g. Bruker processed data loaded into COLMARvista)
- User-set contour threshold value
- User-set contour level count/spacing

## Outputs

- 2D WebGL contour plot image (PNG or SVG screenshot)
- 3D surface mesh rendering image (PNG or equivalent screenshot)
- Visual confirmation that both renderings show equivalent peak content

## How to apply

Load the already-processed 2D spectrum into COLMARvista (a JavaScript/WebGL tool that runs client-side in any recent browser, so no OS-dependent installation is needed). Set the contour threshold and number of contour levels appropriate for the spectrum's intensity range — noise floor at the lower bound, brightest cross-peaks not saturating at the upper bound. Render the WebGL 2D contour plot and export/save the contour image (e.g. PNG or SVG screenshot). Switch the renderer to 3D surface mesh mode and render the same spectrum as a mesh, then export/save the mesh image. Finally, visually verify the two renderings show equivalent peak content (same peak positions and relative intensities represented in both); if contours clip weak peaks or the mesh drowns in noise, revisit the threshold/level settings and re-render.

## Related tools

- **COLMARvista** (WebGL-based renderer used to set contour threshold/levels and produce 2D contour and 3D surface mesh images of processed spectra) — https://github.com/lidawei1975/colmarvista
- **JavaScript** (Underlying technology making the graphics rendering browser-based and OS-independent)

## Evaluation signals

- 2D contour plot renders with peak positions matching the spectrum's known cross-peak coordinates
- 3D surface mesh rendering shows the same peak content (positions/intensities) as the 2D contour plot
- Contour threshold is low enough that weak cross-peaks are visible but high enough that noise is not contoured
- Exported images (PNG/SVG) successfully saved from both contour and mesh modes
- Rendering succeeds in a recent web browser with no local installation, confirming architecture/OS independence

## Limitations

- Threshold/level selection is manual and spectrum-dependent; a poorly chosen threshold can clip weak peaks or contour noise, requiring iteration
- Local (offline) use requires browser configuration changes (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari 'Disable Local File Restrictions'), which the README flags as a security risk
- Rendering relies on WebGL/WebAssembly support in the browser; older browsers may not support it

## Evidence

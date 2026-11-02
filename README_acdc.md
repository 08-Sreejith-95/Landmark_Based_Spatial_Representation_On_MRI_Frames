# Cardiac MRI Geometry: LV/RV Landmarks and Affine Alignment (ACDC)

Exploratory, self-directed project (04/2023 – 05/2023) on the geometry of cardiac MRI segmentations from the [ACDC dataset](https://www.creatis.insa-lyon.fr/Challenge/acdc/).

**Status: exploratory prototype, not a finished method.** The extraction and alignment stages are implemented. The anomaly-detection model that motivated the project was never built (see [Why it stopped](#why-it-stopped)).

## Motivation

The original idea was to detect abnormal heartbeat cycles from how the heart's geometry changes over time:

1. Extract keypoints from the segmentations of every frame, in particular on the border between the right and left ventricle (the septal wall).
2. Track how these points move over the cardiac cycle, and how the wall expands and contracts.
3. Build a temporal graph from the points and train a graph CNN to separate normal cycles from abnormal ones.

## What is implemented

- **Loading and layout.** ACDC segmentations in NIfTI format are loaded with `nibabel` and converted to PyTorch tensors, with axes reordered to (time, depth, height, width). Cine sequences have 30–40 frames, depending on the patient, and were normalised to **34 frames**.
- **Landmarks.** The centroid of the left-ventricle (LV) and right-ventricle (RV) label is computed from the mask's voxel indices.
- **Alignment.** Each slice is rotated and translated so that the LV centre is at the image centre and the LV→RV direction is at a fixed angle. This uses a 2D affine transform (`affine_grid` / `grid_sample`, nearest-neighbour interpolation to keep labels intact).
- **Point transformation.** The RV centroid is moved with the inverse of the affine matrix, which is the correct direction for the `grid_sample` convention, and plotted on the transformed mask for checking.
- **Visualisation.** A row of six rotations (60° steps) around the LV, with the LV centre (red) and the RV centroid (green) marked.

## What is not implemented

- Extraction of keypoints on the RV/LV border or septal wall (an outline is left as a commented block in the code).
- Temporal trajectories over all frames and patients.
- Any graph construction, graph CNN, or anomaly-detection model.
- Quantitative evaluation of any kind.

## Why it stopped

I stopped before modelling because I expected the border keypoints to be too noisy. Points on a mask boundary move in whole pixels, and choosing the same anatomical point across frames is hard. A model trained on such data would inherit that noise. This was my judgement and I did not test it.

A test I would run first: measure the frame-to-frame jitter of the extracted points within healthy patients and compare it to the differences between patients and pathology groups. ACDC provides pathology labels for this. Options to reduce noise are sub-pixel contours, a smooth spline fit to the wall, anatomically defined points such as the RV insertion points, and temporal smoothing over the cycle.

## Repository structure

```
Datapreprocess.py   # loading, label handling, affine and landmark helpers
pathmanager.py      # paths to the ACDC files (edit for your setup)
Parser.py           # [describe]
<notebook/script>   # [name of the demo that produces the six-rotation figure]
```

*(Update this section to match the repository.)*

## Setup

```bash
pip install numpy nibabel torch matplotlib ipython
```

1. Download ACDC and accept its terms (the data is not included here).
2. Edit `pathmanager.py` so the paths point to your copy.
3. Run the demo to produce the aligned-slice figure: `[command or notebook name]`.

## Label values

The code uses these label values: `Left_Ventricle = 3`, `Right_Ventricle = 5`, `Myocardium = 1`. [State here how the labels were produced or remapped from the ACDC originals.]

## Possible next steps

- Test the noise assumption as described above.
- Extract the septal-wall contour and sample points at fixed arc-length positions.
- Use wall thickness, curvature or LV–RV distance over time as features, since they average over many pixels.
- Only then build the temporal graph and a graph CNN.

## Reference

Bernard et al., "Deep Learning Techniques for Automatic MRI Cardiac Multi-structures Segmentation and Diagnosis: Is the Problem Solved?", IEEE Transactions on Medical Imaging, 2018.

## License

[Add a license, e.g. MIT. Note that ACDC has its own terms of use.]

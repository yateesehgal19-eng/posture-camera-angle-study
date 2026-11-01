# Posture Assessment from Phone Photos: Effect of Camera Angle

**A pilot computer-vision study investigating how camera viewpoint affects 2D pose-based posture classification and joint-angle estimation.**

## Overview

Camera-based posture assessment systems rely on detecting body landmarks to estimate posture. However, the camera viewpoint can influence which body movements are visible and how accurately these landmarks represent the person's position.

This project investigates how three camera views — front, 45° and side — affect posture classification and angle estimation using MediaPipe Pose.

The study focuses on understanding where pose-based assessment performs well, where it becomes unreliable, and how performance changes when a model is applied to a camera view different from the one it was trained on.

## Research Question

**How does camera angle (front, 45°, and side) affect posture classification and joint-angle estimation using MediaPipe Pose landmarks?**

### Hypothesis

Side-view images are expected to provide more useful information for identifying forward/backward postural changes, while frontal views may struggle to capture movements occurring along the camera's depth axis.

This is a hypothesis being investigated, not an assumed result.

## Experimental Design

The initial pilot dataset consists of:

| Parameter | Description |
|---|---|
| Participants | 5 volunteers |
| Total photographs | 45 |
| Posture categories | Upright, slouched, lean |
| Camera views | Front, 45°, side |
| Images per participant | 9 |
| Pose estimation model | MediaPipe Pose Landmarker (Heavy) |

Each participant was photographed in three posture categories from three camera views.

The photographs were collected in a controlled indoor setting. The intended setup used a consistent phone position, plain background and similar lighting conditions.

## Methodology

### 1. Pose Landmark Extraction

MediaPipe Pose Landmarker was used to detect body landmarks from each photograph.

The study uses three landmarks from the detected pose:

- Right ear
- Right shoulder
- Right hip

OpenCV was used for image processing and for generating landmark-overlay images.

### 2. Angle Calculation

Two angles were calculated from the detected landmarks:

- **Neck angle:** Angle of the ear-to-shoulder line relative to the vertical.
- **Trunk angle:** Angle of the shoulder-to-hip line relative to the vertical.

Angles were calculated from 2D image coordinates and expressed in degrees.

These measurements are image-plane projections and should not be interpreted as direct measurements of the person's true 3D anatomical angles.

### 3. Data Processing

The extracted measurements and associated labels were stored in a CSV file using pandas.

The dataset was examined for:

- Missing pose detections
- Missing angle measurements
- Participant-wise image counts
- Differences in angle distributions across posture categories and camera views

### 4. Posture Classification

A logistic regression classifier was implemented using:

- Neck angle
- Trunk angle

The features were standardised using `StandardScaler`, with the preprocessing and classifier combined in a scikit-learn pipeline.

**Leave-One-Person-Out cross-validation** was used for within-view evaluation. In each fold, the model was trained on four participants and tested on the remaining participant.

### 5. Cross-View Evaluation

Additional experiments investigated whether a classifier trained using one camera view could classify postures captured from another view.

This was used to examine cross-view generalisation and the sensitivity of the angle-based features to camera placement.

## Preliminary Results

The initial within-view classification results were:

| Camera view | Images evaluated | Accuracy |
|---|---:|---:|
| Side | 15 | 100% |
| 45° | 15 | 100% |
| Front | 15 | 26.7% |

The front-view result was close to the 33.3% chance-level accuracy for a three-class classification task.

### Cross-View Classification

The following table shows the preliminary cross-view results using both neck and trunk angles.

| Training view | Test view | Accuracy |
|---|---|---:|
| Side | 45° | 87% |
| Side | Front | 60% |
| 45° | Side | 100% |
| 45° | Front | 33% |

The results suggest that classification performance can change substantially when the camera view changes.

### Observations

**1. Limited trunk-angle variation in frontal images**

The mean trunk angles for the three posture categories in the frontal view were:

| Posture | Mean trunk angle |
|---|---:|
| Lean | -6.8° |
| Slouched | -7.0° |
| Upright | -6.6° |

The measurements showed very little separation between the three categories. One possible explanation is that forward/backward movement occurs partly along the camera's depth axis, which is not well represented by a frontal 2D projection.

**2. Camera-dependent angle measurements**

The mean neck angle for the upright category differed across views:

- Side: 8.2°
- 45°: 22.0°
- Front: 25.2°

This indicates that angle measurements depend on camera viewpoint and that fixed angle thresholds may not transfer reliably between views.

**3. Landmark detection errors at 45°**

Visual inspection identified six photographs in which the detected hip landmark appeared to fall on the adjacent hand.

A sensitivity analysis was conducted after excluding these six photographs. The results changed, demonstrating that landmark localisation errors can influence downstream posture classification.

Both the original and exclusion-based results are retained for interpretation rather than treating the excluded images as if they had never existed.

## Limitations

This is an exploratory pilot study. Its findings should be interpreted cautiously.

- **Small sample:** Only five participants and 45 photographs were included.
- **Limited generalisability:** The dataset is too small to establish performance on a wider population.
- **2D measurements:** Angles are calculated from image coordinates and are affected by camera projection.
- **Controlled postures:** Participants held instructed postures rather than being observed during natural everyday activity.
- **Single model:** Only one pose-estimation model was evaluated.
- **Single environment:** Images were collected in a controlled indoor setting.
- **No repeated captures:** The initial dataset does not measure within-person variation across repeated photographs.
- **Landmark reliability:** The visibility score was 1.0 for all detected photographs and therefore did not provide a useful quality filter in this dataset.

The results do not establish clinical validity or demonstrate that the system can independently diagnose poor posture.

## Future Work

Potential extensions include:

- Increasing the number of participants.
- Collecting repeated photographs under each experimental condition.
- Establishing a more objective reference protocol for posture and angle measurements.
- Investigating MediaPipe's 3D world landmarks alongside 2D image coordinates.
- Comparing additional pose-estimation models.
- Exploring more robust landmark-quality checks.
- Evaluating performance across different lighting conditions, camera distances and environments.

## Technologies Used

- Python
- MediaPipe Pose Landmarker
- OpenCV
- NumPy
- pandas
- scikit-learn
- Matplotlib
- Google Colab
- GitHub

## Repository Contents

```text
posture-camera-angle-study/
│
├── results/
│   ├── angles.csv
│   └── generated visualisations
│
├── 02_extract_angles.ipynb
├── posture_test (1).ipynb
└── README.md
```

The notebooks contain the pose-detection, angle-extraction, exploratory analysis, classification and evaluation workflow.

## Reproducibility and Data Privacy

The analysis was developed in Google Colab. The notebook currently expects the image dataset to be available in a Google Drive folder named `posture_study/photos`.

The original participant photographs are not included in this public repository. The extracted measurements and derived outputs are used for analysis, subject to appropriate privacy and consent considerations.

To reproduce the analysis, the required Python libraries must be installed and the image folder structure configured.

## Acknowledgements

Pose estimation was performed using [MediaPipe](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker).

The study uses the MediaPipe Pose Landmarker model and associated documentation.

---

**Project status:** Initial pilot analysis completed; methodology refinement and further validation are ongoing.

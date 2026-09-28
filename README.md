# FutBotMX-Seg

**Visually Annotated Image Sequences for Automated Analysis of Educational Robot Soccer Competitions**

FutBotMX-Seg is a dataset of annotated image sequences acquired during educational robot soccer matches at the *Torneo Mexicano de Robótica 2026*. It contains images captured from complementary viewpoints and annotations for robots, balls, goals, and playing-field regions.

The dataset is intended to support research on object detection, instance segmentation, tracking, and automated match analysis in educational robot soccer.

## Dataset overview

| Property | Description |
|---|---|
| Images | 36,027 |
| Source videos | 127 |
| Original video frames | 359,714 |
| Recording duration | 136.12 minutes |
| Acquisition dates | 17 and 18 April 2026 |
| Capture devices | Camera and wearable Glasses |
| Image resolutions | 1,080 x 1,920 and 1,360 x 1,808 pixels |
| Annotated classes | 4 |
| Dataset split | Approximately 70% training, 20% validation, and 10% testing |
| Annotation formats | Binary PNG masks, semantic masks, YOLO polygons, and COCO-based JSON |
| Dataset version | 1.0 |

## Data acquisition

The source recordings were organised into three acquisition sessions:

- `17Abril`
- `17Abril_tarde`
- `18Abril`

Two capture devices were used:

- **Camera:** 1,080 x 1,920 pixels.
- **Glasses:** 1,360 x 1,808 pixels.

The Camera provided external views of the playing field, whereas the wearable Glasses provided a first-person perspective.

| Session | Device | Resolution | Videos | Duration (min) | Extracted images |
|---|---|---:|---:|---:|---:|
| `17Abril` | Camera | 1,080 x 1,920 | 20 | 11.94 | 2,335 |
| `17Abril` | Glasses | 1,360 x 1,808 | 32 | 24.44 | 6,882 |
| `17Abril_tarde` | Camera | 1,080 x 1,920 | 24 | 12.89 | 2,328 |
| `17Abril_tarde` | Glasses | 1,360 x 1,808 | 13 | 8.93 | 3,203 |
| `18Abril` | Camera | 1,080 x 1,920 | 8 | 47.85 | 10,494 |
| `18Abril` | Glasses | 1,360 x 1,808 | 30 | 30.07 | 10,785 |
| **Total** | - | - | **127** | **136.12** | **36,027** |

The videos contained 359,714 frames recorded at approximately 30 or 60 frames per second. To reduce temporal redundancy, one frame out of every ten was retained, resulting in 36,027 images.

Of these images, 15,157 were obtained with the Camera and 20,870 with the Glasses.

## Dataset partitioning

The target distribution was:

- 70% training.
- 20% validation.
- 10% testing.

The split was performed at the video level. All frames extracted from the same video were assigned exclusively to one subset. This prevents consecutive or visually similar frames from appearing in both training and evaluation data.

Because the source videos contain different numbers of frames, the final number of images in each subset may differ slightly from the target 70/20/10 distribution.

The acquisition session, capture device, and source-video hierarchy are preserved within each subset.

## Annotated classes

FutBotMX-Seg defines four final classes:

| YOLO ID | Semantic-mask value | Class |
|---:|---:|---|
| 0 | 1 | `soccer_robot` |
| 1 | 2 | `soccer_ball` |
| 2 | 3 | `soccer_field` |
| 3 | 4 | `soccer_goal` |

The value `0` is reserved for the background in semantic masks.

Different prompt formulations describing the same object type were mapped to a single final class. For example, the prompts `ball`, `soccer ball`, `small ball`, and `ball on the floor` were consolidated into the `soccer_ball` class.

## Annotation process

Initial segmentation masks were generated using SAM 3 with descriptive text prompts.

| Final class | SAM 3 prompts |
|---|---|
| `soccer_robot` | `robot`, `soccer robot`, `mobile robot` |
| `soccer_ball` | `ball`, `soccer ball`, `small ball`, `ball on the floor` |
| `soccer_field` | `soccer field` |
| `soccer_goal` | `goal`, `soccer goal`, `goalpost`, `goal frame` |

The automatic annotation process used the following parameters:

| Parameter | Value |
|---|---:|
| Minimum confidence threshold | 0.40 |
| Binary mask threshold | 0.20 |
| Minimum mask area | 20 pixels |
| Internal processing resolution | 1,008 pixels |
| Duplicate-mask IoU threshold | 0.90 |

The processing resolution of 1,008 pixels was used only internally by SAM 3. The released images, masks, and annotations retain the original image dimensions.

Same-class masks with an intersection over union of at least 0.90 were considered duplicate proposals, and only the mask with the highest confidence score was retained.

The resulting annotations were subsequently reviewed and manually refined to:

- Remove false-positive detections.
- Remove remaining duplicate masks.
- Correct class assignments.
- Correct mask boundaries.
- Complete partially segmented regions.
- Add objects missed during automatic annotation.

The prompt associated with each initial detection was preserved in the processing metadata.

## Annotation formats

### Instance masks

Each annotated instance is represented by a binary PNG mask:

- `255` represents the annotated object.
- `0` represents the background.

These masks are retained as the reference segmentation annotations.

### Semantic masks

A semantic mask is provided for each annotated image using the class values defined above. Larger regions are inserted first so that smaller objects remain visible where masks overlap.

### YOLO instance segmentation

The external contours of the binary masks were converted into polygons. Each image has a text annotation file containing one line per instance:

```text
<class_id> <x1> <y1> <x2> <y2> ... <xn> <yn>
```

Polygon coordinates are normalised by image width and height. Empty annotation files represent images without labelled instances.

The `data.yaml` file defines the dataset paths and class names.

### COCO-based annotations for SAM 3

The instance masks were encoded using run-length encoding and stored in COCO-based JSON files.

Each annotation may contain:

- Image identifier.
- Category identifier.
- Segmentation.
- Area.
- Bounding box.
- `iscrowd`.
- Confidence score.
- `noun_phrase`.
- Original prompt.

COCO category identifiers start at `1`, whereas YOLO class identifiers start at `0`.

Separate JSON files are provided for the training, validation, and test subsets, as well as for the complete dataset.

## Directory structure

```text
FutBotMX-Seg/
├── metadata/
│   ├── acquisition.csv
│   ├── classes.json
│   ├── prompts.json
│   └── splits.csv
├── images/
│   ├── train/
│   │   └── group/device/video/
│   ├── val/
│   │   └── group/device/video/
│   └── test/
│       └── group/device/video/
└── annotations/
    ├── yolo/
    │   ├── data.yaml
    │   └── labels/
    │       ├── train/
    │       ├── val/
    │       └── test/
    ├── instance_masks/
    ├── semantic_masks/
    └── coco_sam3/
        ├── instances_all.json
        ├── instances_train.json
        ├── instances_val.json
        └── instances_test.json
```

This organisation allows each image to be associated with its acquisition session, capture device, source video, dataset subset, and corresponding annotations.

## Download

The complete FutBotMX-Seg dataset is available from the [FutBotMX-Seg repository](https://github.com/QuetzalCpp/FutBotMX-Seg).

```bash
git clone https://github.com/QuetzalCpp/FutBotMX-Seg.git
```

## Usage

For YOLO instance-segmentation training, use the supplied `data.yaml` configuration and the polygon annotations under `annotations/yolo/`.

For SAM 3 fine-tuning or other COCO-compatible workflows, use the JSON files under `annotations/coco_sam3/`.

The PNG instance masks should be considered the reference annotations when an application requires the original mask boundaries.

## Citation

If you use FutBotMX-Seg in academic work, please cite the associated publication. The complete BibTeX entry will be added after publication.

Until then, the dataset may be referenced as:

```text
FutBotMX-Seg, version 1.0, 2026.
https://github.com/QuetzalCpp/FutBotMX-Seg
```

## License

This dataset is released under the [Creative Commons license (CC BY-NC-SA 3.0)](http://creativecommons.org/licenses/by-nc-sa/3.0/), which is free for non-commercial use, including research.

## Contact

Questions, corrections, and suggestions can be submitted through the repository's [issue tracker](https://github.com/QuetzalCpp/FutBotMX-Seg/issues).

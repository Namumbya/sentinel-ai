# ML Research Notes

## Research Objective

Build a reliable damage assessment pipeline that can detect and localize infrastructure damage from satellite imagery while exposing uncertainty and operational limitations. The research track should produce reproducible experiments, not isolated notebooks.

## Candidate Datasets

- xBD: large-scale building damage assessment dataset with pre-disaster and post-disaster imagery and damage labels.
- SpaceNet: building footprints, roads, and geospatial computer vision benchmarks.
- xView2: disaster damage assessment challenge data derived from xBD.
- Sentinel-2 public imagery: useful for broader area monitoring, lower resolution than commercial imagery.
- Landsat and MODIS: useful for coarse disaster context, less suitable for fine infrastructure damage.
- OpenStreetMap: useful for roads, buildings, and infrastructure context, but labels may be incomplete or stale.
- Copernicus Emergency Management Service products: useful for validation and operational comparison.

Dataset trade-off: xBD is the strongest starting point for supervised building damage, but real deployments must handle domain shift across sensors, geographies, seasons, disasters, and image resolutions.

## Data Pipeline

### Ingestion

- Store raw imagery immutably in object storage.
- Capture metadata: sensor, capture time, disaster type, resolution, CRS, bounding box, cloud cover, and license.
- Version dataset manifests with DVC, LakeFS, or object versioning plus manifest hashes.

### Preprocessing

- Reproject imagery to a consistent coordinate reference system when required.
- Align pre-disaster and post-disaster imagery for change detection workflows.
- Tile large rasters into model-compatible windows with overlap.
- Normalize channels using training-set statistics.
- Preserve geospatial transforms for reconstructing predictions.
- Reject or flag unusable tiles due to clouds, no-data regions, corrupt files, or missing bands.

### Augmentation

- Random crops and flips.
- Rotation when label semantics allow it.
- Color jitter and brightness shifts for sensor and lighting variation.
- Gaussian noise and blur for robustness.
- CutMix or MixUp only after verifying label behavior for segmentation masks.
- Disaster-specific augmentation such as smoke or haze should be used carefully to avoid unrealistic artifacts.

## Model Approaches

### CNN Baselines

Use ResNet or EfficientNet classifiers on image patches to establish a simple benchmark.

Pros:

- Fast to train.
- Easy to debug.
- Useful for triage-level damage classification.

Cons:

- Poor spatial precision.
- Harder to produce map-ready damage boundaries.

### UNet Segmentation

Use UNet or UNet++ with ResNet or EfficientNet encoders for pixel-level damage segmentation.

Pros:

- Strong baseline for geospatial masks.
- Mature implementation ecosystem.
- Easier to serve than large transformer models.

Cons:

- Can struggle with global context.
- Requires high-quality segmentation masks.

### SegFormer

Use SegFormer for transformer-based semantic segmentation.

Pros:

- Strong accuracy on segmentation tasks.
- Good multi-scale representation.
- Often robust across visual contexts.

Cons:

- More expensive than UNet.
- Requires careful memory management for large imagery.

### Vision Transformers

Evaluate ViT, Swin Transformer, or hybrid CNN-transformer models for classification and segmentation.

Pros:

- Strong representation learning.
- Can improve generalization with enough data.

Cons:

- Higher compute cost.
- More sensitive to dataset size and training recipe.

### Object Detection

Evaluate YOLO, Faster R-CNN, RetinaNet, or DETR-style models for building, road, bridge, and infrastructure detection.

Pros:

- Good for object-level counting and severity summaries.
- Useful when labels are boxes or polygons.

Cons:

- Less suitable for diffuse damage such as flooding or burn scars.
- Requires object annotations and careful geospatial postprocessing.

## Recommended MVP Model Path

Start with:

1. Patch-level CNN classifier as a sanity-check baseline.
2. UNet segmentation model for damage masks.
3. SegFormer as the first advanced segmentation model.

This sequence gives a fast baseline, a production-relevant map output, and a stronger model family for later comparison.

## Evaluation

Core metrics:

- IoU for segmentation mask overlap.
- Precision for false-positive control.
- Recall for missed damage control.
- F1 score for balanced damage classification.
- mAP for object detection approaches.
- Expected calibration error for confidence reliability.
- Confusion matrix by damage severity class.

Operational metrics:

- Inference latency per square kilometer or per tile.
- Cost per assessment job.
- Percentage of low-confidence predictions requiring analyst review.
- Performance by disaster type, geography, sensor, and resolution.

Evaluation guidance:

- Keep a locked validation set for regression testing.
- Use geographically separated test splits to estimate domain shift.
- Report metrics by disaster type, not only aggregate averages.
- Track false positives on visually similar but undamaged infrastructure.
- Review calibration before exposing confidence thresholds to users.

## Experiment Tracking

Use MLflow or Weights & Biases to track:

- Git commit.
- Dataset version.
- Preprocessing config.
- Model architecture and hyperparameters.
- Training metrics and validation metrics.
- Evaluation artifacts.
- Calibration plots.
- Example predictions.
- Model card draft.

MLflow is easier to self-host and integrates well with model registries. Weights & Biases is excellent for collaboration and visualization. For an open-source-first repo, MLflow is a conservative default.

## Model Versioning

Each production candidate should have:

- Immutable artifact URI.
- Model registry version.
- Training dataset manifest hash.
- Evaluation report.
- Model card.
- Inference container version.
- Approval status: experimental, staging, approved, deprecated.

## Reproducible Training

- Pin Python dependencies.
- Store training configuration as YAML.
- Set random seeds where possible.
- Save dataset manifests.
- Run training through a script or pipeline, not manually edited notebooks.
- Use Docker images for training and inference environments.
- Keep notebooks for exploration only, then promote stable logic into package code.

## Model Monitoring

Monitor:

- Input image resolution and channel distribution.
- Cloud cover and no-data rates.
- Prediction class distribution.
- Confidence distribution.
- Calibration drift.
- Analyst override rate.
- Job failure rate by model version.

Retraining triggers:

- Sustained drop in analyst-validated precision or recall.
- New disaster type or region with poor performance.
- Sensor or resolution shift.
- Labeling improvements or expanded datasets.

## Model Card Template

Each model card should include:

- Intended use.
- Out-of-scope use.
- Training data.
- Evaluation data.
- Metrics by class and disaster type.
- Known limitations.
- Confidence calibration notes.
- Ethical and safety considerations.
- Deployment version and rollback plan.

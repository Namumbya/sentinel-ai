# Product Requirements Document

## Product Summary

Sentinel AI is a web platform for uploading or registering satellite imagery, running AI damage assessment jobs, visualizing geospatial damage maps, and exporting response-ready reports. It is designed for disaster response workflows where speed matters, but transparency and analyst review are required.

## Goals

- Provide a production-style workflow for post-disaster infrastructure damage assessment.
- Support asynchronous ML inference over large geospatial images.
- Make model output interpretable through map layers, confidence scores, and downloadable reports.
- Establish engineering practices for reproducible ML, secure APIs, monitoring, and deployment.

## Non-Goals

- Replace official field inspection.
- Guarantee perfect damage classification from a single image source.
- Build a general GIS platform.
- Train foundation models from scratch in the MVP.

## Personas

- Analyst: uploads imagery, reviews damage layers, validates uncertain results, and exports reports.
- Incident commander: views summarized damage severity by region and prioritizes response.
- ML engineer: manages datasets, experiments, model versions, and evaluation reports.
- Platform engineer: operates APIs, workers, queues, storage, monitoring, and deployments.

## MVP Features

- Account login with analyst and admin roles.
- Create an assessment project for a disaster event and area of interest.
- Upload GeoTIFF or register cloud-hosted imagery.
- Start asynchronous analysis jobs.
- Track job states: queued, preprocessing, inferencing, postprocessing, completed, failed.
- Display damage predictions on an interactive map.
- Filter predictions by damage class and confidence threshold.
- Export assessment results as JSON, GeoJSON, and a basic PDF report.
- View historical assessments for the same region.
- Record model version, dataset version, and preprocessing configuration for each job.

## Functional Requirements

- The API must accept imagery metadata, area of interest geometry, disaster type, and project context.
- The preprocessing service must tile large images, normalize channels, validate metadata, and store tile manifests.
- The inference service must load an approved model version from the model registry.
- The system must persist job status transitions and error details.
- The dashboard must refresh job status without requiring a page reload.
- The map must render predicted damage masks or vectorized polygons.
- Reports must include model version, timestamp, area of interest, class summary, confidence notes, and limitations.
- Admin users must be able to view system health and model versions.

## Non-Functional Requirements

- Reliability: failed jobs should be retryable without duplicating completed artifacts.
- Scalability: workers should scale independently from the API and frontend.
- Security: all user actions should be authenticated, authorized, logged, and scoped by tenant or organization.
- Observability: services should expose metrics, structured logs, traces, and model-specific monitoring signals.
- Reproducibility: each result should be traceable to input imagery, code version, model artifact, and configuration.
- Performance: small-area jobs should produce first usable output within minutes under normal load.
- Compliance readiness: data access and report generation should be auditable.

## User Stories

### Upload and Analyze Imagery

As an analyst, I want to upload a satellite image and start an assessment so that I can quickly identify likely damage.

Acceptance criteria:

- Given a valid GeoTIFF, when the user starts an assessment, then the system creates a job and shows `queued`.
- Given an invalid image, when upload validation runs, then the user sees a clear error message.
- Given a completed job, when the user opens the assessment, then the map shows damage overlays and confidence filters.

### Monitor Assessment Jobs

As an analyst, I want to monitor job progress so that I know when results are ready.

Acceptance criteria:

- The dashboard shows current status, started time, updated time, and failure reason when applicable.
- Job status updates appear without a full page refresh.
- Failed jobs can be retried by an authorized user.

### Review Damage Map

As a response lead, I want to view damage by severity and location so that I can prioritize response areas.

Acceptance criteria:

- The map supports pan, zoom, layer toggles, and confidence threshold filtering.
- The summary panel shows affected area, detected objects or regions, and class distribution.
- Results include a visible model version and assessment timestamp.

### Generate Report

As a government or humanitarian coordinator, I want to export a report so that I can share findings with stakeholders.

Acceptance criteria:

- The report includes event metadata, map snapshot, damage summary, model details, and limitations.
- JSON and GeoJSON exports are available for downstream systems.
- Report generation is logged as an auditable action.

## Risks and Trade-Offs

- Segmentation gives spatially precise damage maps but requires high-quality masks and careful geospatial postprocessing.
- Object detection is useful for buildings and roads but may miss diffuse flood or burn damage.
- Vision transformers can improve accuracy but are more expensive to train and serve than CNN baselines.
- Fully automated outputs are faster but risky; the MVP should keep human review in the loop.

## Open Questions

- Which first disaster type should the MVP optimize for: earthquake, flood, wildfire, hurricane, or mixed events?
- Will the first dataset be xBD, SpaceNet, Sentinel-2 public imagery, commercial imagery, or a combination?
- Should tenancy be single-organization for MVP or multi-tenant from day one?
- What regulatory or data-sharing constraints apply to target users?

# VMS Presentation Notes

## Main Goal

This section should focus on only three messages:

1. The VMS is the center of the video review workflow.
2. The strongest completed VMS contribution is the playback-centered workflow.
3. The VMS connects edge upload, cloud playback, and AI-assisted analysis in one chain.

## Slide: Research Questions for VMS

- Our VMS section is driven by three research questions.
- First, how to provide efficient playback of recorded video from cloud storage.
- Second, how to receive clips reliably from edge or upload sources in a web-compatible way.
- Third, how to integrate with VSS without forcing the frontend to understand AI internals.
- Key message: the VMS is not just a storage module; it turns video clips into usable investigation resources.

## Slide: VMS Solution Overview

- The VMS is designed as an orchestration layer between the frontend, storage, edge input, and VSS.
- The frontend talks only to VMS APIs, not directly to GCS or VSS.
- The FastAPI backend validates uploads, converts video, proxies playback, and forwards analysis requests.
- GCS stores the playback-ready video objects.
- Pub/Sub is used as the asynchronous handoff for downstream ingest.
- Key message: playback is the center, and the surrounding services are organized around it.

## Slide: Edge-to-VMS Upload Workflow

- The edge sends clips to the VMS through HTTP upload.
- The VMS receives both the binary file and metadata via multipart/form-data.
- It validates file type, file size, and file naming.
- If valid, it stores the raw input, converts it to MP4, and then continues cloud synchronization.
- Key message: this is the normalization gateway for the entire playback pipeline.

## Slide: VMS Video Ingest Workflow

- After receiving the clip, the VMS stores the raw file in `uploads_raw/`.
- Then it uses FFmpeg to normalize the clip into a browser-compatible MP4.
- The normalized file is saved in `uploads/`, and then uploaded to GCS when auto-ingest is enabled.
- The system also prepares metadata for downstream processing.
- Key message: ingest here is not just storage; it transforms a clip into a playback-ready asset.

## Slide: VMS Playback Workflow

- The frontend first calls the API that lists video objects from GCS.
- When the user selects a clip, the frontend generates a playback URL through the VMS backend.
- The backend does not force the browser to download the whole file.
- Instead, it streams the clip through HTTP Range requests.
- That is what makes seeking practical and efficient.
- Key message: the most important VMS playback improvement is cloud-backed playback with HTTP Range support.

## Slide: VMS AI Integration Workflow

- When the user opens Deep Analyze, the frontend calls the analysis API through VMS.
- The VMS acts as the intermediary and forwards the request to the AI service.
- When the user asks a clip-specific question, the VMS enriches the request with clip context such as `video_id`, `camera_id`, `date`, and `location`.
- Then it forwards the query to VSS and returns `answer`, `sources`, and `raw`.
- Key message: playback is no longer the last step; it becomes the entry point for AI-assisted investigation.

## Slide: VMS Solution

- If I summarize the VMS contribution in one sentence:
- The VMS receives clips, normalizes them, plays them back efficiently, and connects them to the AI workflow.
- It is also the clearest operator-facing workflow in the current capstone.
- This is why the VMS playback path is the strongest part to defend.

## Slide: Evaluation Results - VMS

- The VMS evaluation is focused on the playback-centered workflow, not on the full commercial VMS product space.
- We evaluate four main indicators:
- event-to-playback availability,
- playback open time,
- seek latency,
- media error rate.
- Key message: we intentionally evaluate the strongest validated workflow core.

## Slide: Evaluation Results - VMS (metrics)

- The retained playback open time is about 2.64 seconds on average.
- Seek latency is about 1.83 seconds median and 2.51 seconds at p95.
- The retained media error rate is 0% in the controlled test set.
- Event-to-playback availability is 100% in the retained controlled path.
- Key message: these results show that the playback workflow is already usable for review and demonstration.

## Slide: VMS Layer

- To close the VMS section, I should frame the scope carefully.
- The strongest VMS contribution today is centralized storage and playback.
- Other areas such as dashboard, live view, and event-management breadth are still less mature.
- So the correct claim is: the playback workflow is the strongest validated VMS core, not that the whole VMS is fully complete.

## Transition to the Next Section

- From the VMS point of view, the workflow does not stop at watching a clip.
- The value of the system continues when that clip becomes input for VSS analysis, retrieval, and question answering.

## Short Closing Version

- In summary, the VMS solves three practical problems:
- receiving clips from edge sources,
- providing efficient cloud-backed playback,
- and connecting reviewed clips to AI analysis.
- That is why the VMS playback workflow is the clearest and most defensible contribution in the current capstone.

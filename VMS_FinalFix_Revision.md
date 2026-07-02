# VMS Final Fix Revision

This file is a revision patch for `/home/huy/drive-download-20260330T022318Z-3-001/M2/M3 Final_Fix.docx.md`. It does not replace the original file. It only lists the sections that should be revised, while preserving the citation markers so the text can be copied back into the original report without losing reference alignment.

---

## Section: Motivation

### Original text

```md
The target surveillance context of this project requires more than basic camera viewing.
Operators need to review past incidents quickly, search for people or objects across recorded footage, inspect event evidence, and connect playback with AI-assisted follow-up [1], [2].
Manual review becomes slow when the number of cameras, locations, and time ranges increases, and response-time expectations become more difficult to satisfy as the review workflow becomes fragmented [18], [19], [23]-[25].
It is also difficult to reconstruct what happens when the operator must move separately between camera screens, file storage, playback interfaces, and analysis tools.
The problem is therefore not only how to display video, but how to make video evidence operationally usable.
The current project requirements point to several recurring needs.
The system should support centralized monitoring and management, AI-assisted access control and identity or vehicle verification, event detection and operational alerting, investigation through playback and tracing, statistical reporting, scalable multi-site deployment, and later customization for customer-specific workflows [1], [2].
These needs indicate that the project should not be designed as only a VMS, only an edge detector, or only an AI search tool.
It should be designed as an integrated surveillance platform where capture, review, and analysis reinforce each other.
```

```md
| Scalable deployment and centralized multi-site rollout | Eagle Eye, Verkada, Milestone XProtect, and similar systems show multi-site and centralized deployment directions [7], [30] | A clear edge-cloud workflow that preserves local continuity while still supporting centralized playback and AI-assisted investigation is not always visible [3]-[7], [27], [30] | The system is cloud-connected and modular, but only partially cloud-native/scalable because it still uses personal GCS, MongoDB/Neo4j, and VM infrastructure. |
```

```md
From this benchmark, the motivation of our project becomes clear.
The project does not attempt feature parity with commercial VMS products.
Its defensible contribution is the integrated evidence pipeline: camera stream to event clip, event clip to cloud-backed playback, and playback evidence to searchable VSS knowledge [3]-[14], [20]-[22], [30], [31].
The current prototype demonstrates this integration path, while production-grade scalability, security hardening, and deeper VSS evaluation remain future work.

This direction is motivated by the need to transform surveillance from passive video storage into an evidence-centered investigation workflow.
The Edge Agent reduces unnecessary video handling by converting continuous camera streams into event-oriented clips [3]-[5], [27], [29].
The VMS makes those clips usable through upload handling, normalization, cloud-backed storage, playback, and service orchestration [6], [7], [30], [31], [34], [35].
The VSS enriches selected clips with searchable summaries, identity evidence, structured relationships, and answer support [8]-[14], [20]-[22].
Together, these components form a pipeline in which raw camera streams become event clips, event clips become reviewable evidence, and reviewable evidence becomes searchable surveillance knowledge.
The proposed solution is therefore not intended to replace all existing surveillance products or to claim that each subsystem is fully production-complete.
Instead, it addresses a specific integration gap: how to connect edge capture, centralized playback, and AI-assisted investigation into one defensible capstone system [1], [2].
This is the core motivation of the thesis.
The project is valuable because it demonstrates that surveillance review can move beyond isolated camera viewing and manual clip inspection toward a unified workflow where evidence capture, operator review, and intelligent retrieval are designed together from the beginning [18], [19], [23]-[25].
```

### Revised text

```md
The target surveillance context of this project requires more than basic camera viewing.
Operators need a workflow that can connect incident review, recorded evidence playback, and AI-assisted follow-up in one operational chain [1], [2], [18], [19], [23]-[25].
For this reason, the capstone is positioned as an integrated surveillance prototype rather than as a standalone VMS, edge detector, or AI search tool.
Its main objective is to make video evidence operationally usable by linking capture, playback, and analysis within one cloud-connected workflow.
```

```md
| Scalable deployment and centralized multi-site rollout | Eagle Eye, Verkada, Milestone XProtect, and similar systems show multi-site and centralized deployment directions [7], [30] | A clear edge-cloud workflow that preserves local continuity while still supporting centralized playback and AI-assisted investigation is not always visible [3]-[7], [27], [30] | The current system is modular and cloud-connected, so it is suitable for limited scaling in later stages, but the present prototype still depends on personal GCS, MongoDB/Neo4j, and VM infrastructure and should not be described as fully cloud-native or scalability-validated. |
```

```md
From this benchmark, the project motivation can be stated more precisely.
The capstone does not claim feature parity with commercial surveillance platforms.
Its defensible contribution is a cloud-oriented evidence pipeline that connects event capture, cloud-backed playback, and VSS-based knowledge extraction [3]-[14], [20]-[22], [30], [31].
In other words, the prototype demonstrates integration across Edge, VMS, and VSS, while stronger security governance, larger-scale validation, and deeper VSS evaluation remain future work.
```

### Reason for revision

The revised version removes the stronger claim that the project is already a broadly scalable or cloud-native platform. It keeps the same academic direction, but reframes the contribution as an integrated, cloud-oriented prototype whose strongest evidence is workflow integration rather than production readiness.

---

## Section: Current system overview

### Original text

```md
For the project titles: cloud-native does not simply mean running on an Internet-accessible virtual machine.
It means using clear service boundaries, cloud object storage for large video assets, API-based orchestration, asynchronous handoff for long-running analysis, and deployment-ready configuration [6], [7], [30], [43]-[46].
Scalable means the architecture can grow by separating Edge capture, VMS playback, VSS analysis, media storage, vector search, and graph storage.
In the current capstone, this is a prototype-readiness claim, not a claim of production autoscaling, multi-tenant access control, complete observability, or production SLOs.
Components running on personal GCS, MongoDB/Neo4j, or VM resources should therefore be described as cloud-hosted or cloud-connected unless stronger operational controls are demonstrated.
```

```md
| VMS playback API | Partially cloud-native | Uses FastAPI, GCS-backed playback, HTTP Range, and service boundaries, but still runs in a personal VM-style prototype environment [6], [7], [30], [31], [40]. |
| VSS MongoDB vector store | Cloud-hosted / partially scalable | Supports vector and event retrieval, but deployment and index governance remain prototype-level [9], [50]. |
| VSS Neo4j graph store | Cloud-hosted / partially scalable | Supports graph facts and relationships, but graph evaluation and production governance remain incomplete [14]. |
| Security and privacy controls | Partial / future work | Sensitive surveillance data exists, but personal cloud accounts and VM resources require stronger IAM, encryption, retention, audit, and access controls [47], [52]. |
```

```md
The current system follows a hybrid workflow: Edge -> centralized VMS -> VSS analysis.
Cameras connect at the site edge, event-related clips are forwarded to the VMS for storage and playback, and selected evidence is processed by VSS to generate searchable metadata, identity support, captions, and AI-assisted responses [3]-[14], [29]-[31].
The current system is best described as a working integration of selected core flows, not as a complete surveillance product.
Edge event capture has prototype-level validation.
VMS playback is the strongest operator-facing workflow.
VSS metadata extraction and vector retrieval are meaningful but still need deeper graph, identity, and answer-faithfulness evaluation [1], [2], [6]-[14]
```

### Revised text

```md
In the current thesis, the system should be described as an integrated, cloud-oriented prototype rather than as a fully cloud-native platform.
The implemented architecture already uses service separation, cloud object storage, API-based orchestration, and asynchronous handoff patterns [6], [7], [30], [43]-[46], but several operational controls still remain at prototype level because the deployment depends on personal GCS, MongoDB/Neo4j, and VM resources.
Accordingly, the present evidence supports the terms cloud-connected or cloud-hosted more strongly than a complete cloud-native claim.
```

```md
| VMS playback API | Cloud-oriented prototype service | Uses FastAPI, GCS-backed playback, HTTP Range, and service boundaries, but still runs in a prototype VM-style environment [6], [7], [30], [31], [40]. |
| VSS MongoDB vector store | Cloud-hosted / limited-scale prototype | Supports vector and event retrieval, but deployment, governance, and larger-scale validation remain prototype-level [9], [50]. |
| VSS Neo4j graph store | Cloud-hosted / limited-scale prototype | Supports graph facts and relationships, but graph evaluation and production governance remain incomplete [14]. |
| Security and privacy controls | Partial / future work | Sensitive surveillance data exists, but personal cloud accounts and VM resources still require stronger authentication, authorization, encryption, retention, audit, and access controls [47], [52]. |
```

```md
The current system follows a hybrid workflow: Edge -> centralized VMS -> VSS analysis.
In its present form, it should be understood as a working integration of selected core flows, not as a complete surveillance platform.
Edge event capture has prototype-level validation, VMS playback is the strongest operator-facing workflow, and VSS metadata extraction is meaningful but still requires deeper graph, identity, and answer-faithfulness evaluation [1], [2], [3]-[14], [29]-[31].
The architecture is modular and suitable for limited scaling in later stages, but current evidence only supports small-scale validation rather than a full scalability claim.
```

### Reason for revision

The revised wording aligns the overview with the actual maturity of the deployment. It keeps the architectural value of service separation and cloud usage, but avoids overclaiming cloud-native completeness or proven scalability beyond the current prototype evidence.

---

## Section: Chapter 2 VMS security and governance clarification

### Original text

```md
Taken together, these responsibility-specific comparisons clarify the theoretical boundary of the VMS section.
The VMS playback workflow is not trying to solve every possible video transport problem.
It is solving the narrower but central problem of reliable recorded-clip review in a browser, with asynchronous handoff to later analysis.
Within that target, HTTP/REST, multipart/form-data, HTTP Range, object storage, and Pub/Sub form a coherent protocol foundation [31], [38], [40], [43]-[46].
```

### Revised text

```md
Taken together, these responsibility-specific comparisons clarify the theoretical boundary of the VMS section.
The VMS playback workflow is not trying to solve every possible video transport problem.
It is solving the narrower but central problem of reliable recorded-clip review in a browser, with asynchronous handoff to later analysis.
Within that target, HTTP/REST, multipart/form-data, HTTP Range, object storage, and Pub/Sub form a coherent protocol foundation [31], [38], [40], [43]-[46].

Because the VMS handles surveillance clips and related metadata, the protocol foundation must also be read together with baseline security and data-governance requirements.
First, authentication is needed to verify which actor is calling the upload, playback, analysis, or internal integration endpoint, while authorization is needed to restrict which actions each actor may perform [31], [47], [52].
Second, encryption should protect clip transfer in transit through HTTPS/TLS and should also protect stored objects and databases at rest through the underlying cloud or storage controls [31], [43], [44], [47].

Third, audit logging is required so that clip upload, playback access, query submission, and asynchronous publish events can later be traced for operational review and incident analysis [45]-[47], [52].
Fourth, retention and deletion policy are required because surveillance clips, metadata, and derived analysis artifacts should not be kept indefinitely without a defined operational purpose, storage limit, or compliance basis [43], [44], [47], [52].
In the current capstone, these controls should be described as future hardening directions of the VMS prototype rather than as fully implemented governance capabilities.
```

### Reason for revision

This addition answers the council's request directly at the theory level. It explains why each control exists in a VMS context, keeps the language academic, and clearly states that these are governance requirements and next-step hardening directions rather than completed features.

---

## Section: Chapter 3 VMS security and governance clarification

### Original text

```md
The endpoint layer further makes the VMS contract explicit.
Each endpoint corresponds to one workflow responsibility rather than one vague "video API".
This makes the implementation easier to evaluate because upload intake, cloud listing, playback streaming, analysis retrieval, and contextual query forwarding can be discussed separately [31], [38], [40].
```

### Revised text

```md
The endpoint layer further makes the VMS contract explicit.
Each endpoint corresponds to one workflow responsibility rather than one vague "video API".
This makes the implementation easier to evaluate because upload intake, cloud listing, playback streaming, analysis retrieval, and contextual query forwarding can be discussed separately [31], [38], [40].

From a design-governance perspective, this endpoint separation also identifies where security and lifecycle controls should later be enforced.
In the current architecture, FastAPI functions as the API-layer control point, so future hardening should add explicit authentication for uploader, operator, and internal service identities, together with authorization rules that distinguish upload, playback, analysis access, and administrative operations [6], [31], [47], [52].
The same design direction applies to storage: the GCS-backed playback path should later enforce stronger object-access control and encryption governance for raw uploads, normalized MP4 files, and playback-ready cloud objects [7], [30], [43], [44], [47].

The operational workflow should also be extended with audit and data-lifecycle controls.
Upload attempts, playback access, clip-query requests, and Pub/Sub publish events should be logged so that evidence access and system actions can be reviewed later [45]-[47], [52].
Likewise, retention and deletion rules should later be defined separately for raw upload files, normalized playback files, GCS objects, and VSS-derived analysis artifacts.
In this capstone, these mechanisms are not yet fully implemented and should therefore be presented as concrete design directions for the next stage of the VMS rather than as completed production controls.
```

### Reason for revision

This insertion moves the discussion from theory to architecture. It ties each missing control to a real design boundary already present in the system, which makes the report easier to defend while avoiding any false claim that the controls are already complete.

---

## Section: Conclusion

### Original text

```md
The main conclusion is that the project has already validated a meaningful operational chain rather than merely a conceptual architecture.
The Edge subsystem can transform continuous RTSP streams into event-based clips.
The VMS subsystem can receive, normalize, store, and play back real video evidence with workable latency and high reliability.
The VSS subsystem can enrich selected clips with metadata, identity-aware records, retrieval structures, and grounded question-answering support.
These three contributions are strongest not when viewed separately, but when considered as one evidence pipeline that connects capture, review, and AI-assisted follow-up.
At the same time, the work has several limitations.
The Edge subsystem still requires stronger retry persistence and wider cloud-resilience handling.
The VMS subsystem remains most mature in playback, while other operator-facing modules still need fuller backend completion.
The VSS subsystem already supports grounded metadata and answer generation, but graph-dominant end-user retrieval and richer investigative reasoning are still less mature than the playback-centered workflow.
VSS also still requires stronger ground-truth evaluation, retrieval ablation, identity false-merge/false-split analysis, answer-faithfulness scoring, and security/privacy hardening for surveillance video, face embeddings, and relationship graphs [20]-[22], [47], [51]-[54].
These limitations mean that the current system should be described as an integrated and defensible prototype rather than as a complete production-grade surveillance platform.
Based on these findings, future work should focus on four directions.
First, the Edge-to-VMS path should be strengthened through more complete synchronization control, retry durability, and broader device-management feedback.
Second, the VMS should extend backend completion beyond playback so that dashboard, event-management, and live-view related features reach the same operational maturity as the playback workflow.
Third, the VSS should improve graph-oriented retrieval quality, deeper cross-event reasoning, and richer end-user evidence explanation.
Fourth, the full platform should continue its movement toward stronger cloud-native deployment readiness, cleaner service separation, and more scalable multi-site operation.
The implications of this work for future research are also clear.
Surveillance-system development benefits from treating edge processing, operator review, and AI-assisted retrieval as one continuous architecture rather than as separate research topics.
The capstone shows that hybrid database retrieval, identity-aware evidence handling, and selective query routing are promising directions for practical video-investigation systems.
It also suggests that academically defensible surveillance research does not require claiming full subsystem completion at every stage; it can instead demonstrate value through carefully validated workflow cores that later research and engineering can extend.
The broader conclusion of the thesis is therefore not only that the current prototype works in selected flows, but that the chosen decomposition of the surveillance problem is sound.
Legacy cameras can remain at the capture layer.
Edge gateways can perform event filtering and bounded buffering near the site.
A centralized VMS can serve as the operator's coordination point for playback and evidence access.
A VSS layer can then build identity-aware and relation-aware knowledge from selected clips instead of from the full raw stream.
This decomposition is practically significant because it respects the constraints of hardware, networks, and human investigation workflow at the same time.
From a system-design perspective, the capstone also demonstrates the value of keeping uncertainty explicit.
The edge layer does not pretend to perform all possible analytics.
The VMS layer does not pretend to complete the entire surveillance product space at once.
The VSS layer does not pretend to resolve every identity or produce every answer with full graph-level certainty.
Instead, each subsystem exposes what it can reliably do now and leaves room for later extension.
This makes the overall platform more trustworthy than a design that hides unresolved maturity gaps behind generalized claims.
```

### Revised text

```md
The main conclusion is that the project has validated a meaningful integrated workflow rather than only a conceptual architecture.
The Edge subsystem can transform continuous RTSP streams into event-based clips, the VMS can receive and play back real video evidence, and the VSS can enrich selected clips with metadata and grounded follow-up support [20]-[22], [47], [51]-[54].
Among these flows, VMS playback is currently the strongest operator-facing result.

At the same time, the present evidence supports only prototype-level and limited-scale claims.
The architecture is modular and suitable for later scale-out, but the current capstone should not describe scalability as fully demonstrated.
In current validation, stable parallel operation was confirmed with two real cameras, whereas a continuous ten-camera simulation exhausted GPU resources and caused system failure.
This result indicates that resource hardening, stress validation, and broader operational stability are still incomplete.

For that reason, the system is more accurately described as an integrated, cloud-oriented prototype than as a complete cloud-native surveillance platform.
Future work should therefore focus on stronger synchronization and resilience from Edge to VMS, broader backend completion beyond playback, deeper VSS evaluation, and more complete security and data-governance controls across authentication, authorization, encryption, audit logging, retention, and deletion policy [47], [52].
```

### Reason for revision

The revised conclusion keeps the main achievement of the capstone, but narrows the maturity claim to what has actually been validated. It also converts the scalability issue into a precise limitation statement that is easier to defend under questioning because it is tied to observed test behavior instead of general aspiration.

---

## Bibliography refinement note

The original bibliography is broader than necessary for the final report. To reduce citation overload, the bibliography can be consolidated to **35 references** while keeping the sources that are most central to requirements, architecture, protocols, evaluation, and governance.  
Important note: if this compact bibliography is adopted in the original report, all in-text citation numbers must be renumbered consistently.

### Removed references

- `[4]`: ONVIF, "Profile G"
- `[12]`: Google AI for Developers, "Gemini Models"
- `[13]`: OpenAI, "Models"
- `[15]`: Google Cloud, "Spanner Graph overview"
- `[16]`: Milvus, "Milvus vector database documentation"
- `[18]`: Google, "Core Web Vitals"
- `[24]`: J. Nielsen, "Usability Metrics"
- `[25]`: J. Brooke, "SUS: A quick and dirty usability scale"
- `[26]`: Axis Communications, *Latency in Live Network Video Surveillance*
- `[32]`: ONVIF, "ONVIF Releases Profile G for Video Storage and Recording"
- `[33]`: ITU-T, "P.1203"
- `[34]`: WHATWG, "HTML Standard: Media elements"
- `[35]`: W3C, "Media Source Extensions"
- `[36]`: IETF, "RFC 6455: The WebSocket Protocol"
- `[39]`: IETF, "RFC 4648: The Base16, Base32, and Base64 Data Encodings"
- `[44]`: Google Cloud, "Cloud Storage consistency"
- `[48]`: OpenAI, "Pricing"
- `[49]`: OpenAI, "Images and vision"
- `[54]`: Google Cloud, "Metric prompt templates for model-based evaluation"

### Renumbering map for retained references

- `([1] -> [1])`
- `([2] -> [2])`
- `([3] -> [3])`
- `([5] -> [4])`
- `([6] -> [5])`
- `([7] -> [6])`
- `([8] -> [7])`
- `([9] -> [8])`
- `([10] -> [9])`
- `([11] -> [10])`
- `([14] -> [11])`
- `([17] -> [12])`
- `([19] -> [13])`
- `([20] -> [14])`
- `([21] -> [15])`
- `([22] -> [16])`
- `([23] -> [17])`
- `([27] -> [18])`
- `([28] -> [19])`
- `([29] -> [20])`
- `([30] -> [21])`
- `([31] -> [22])`
- `([37] -> [23])`
- `([38] -> [24])`
- `([40] -> [25])`
- `([41] -> [26])`
- `([42] -> [27])`
- `([43] -> [28])`
- `([45] -> [29])`
- `([46] -> [30])`
- `([47] -> [31])`
- `([50] -> [32])`
- `([51] -> [33])`
- `([52] -> [34])`
- `([53] -> [35])`

### Revised bibliography list (35 references)

```md
[1] CMC Telecom Da Nang, "Nhu cầu AI Cloud Camera-2," internal requirement collection spreadsheet, n.d.

[2] CMC Telecom Da Nang, "CMC Intern Project Initial BRD," internal business requirements document, n.d.

[3] Kerberos.io, "kerberos-io," GitHub, n.d. Online. Available: https://github.com/kerberos-io. Accessed: Apr. 23, 2026.

[4] ONVIF, "ONVIF Profiles," ONVIF, n.d. Online. Available: https://www.onvif.org/profiles/. Accessed: Apr. 23, 2026.

[5] FastAPI, "First Steps," FastAPI Documentation, n.d. Online. Available: https://fastapi.tiangolo.com/tutorial/first-steps/. Accessed: Apr. 29, 2026.

[6] Google Cloud, "About Cloud Storage objects," Google Cloud Documentation, n.d. Online. Available: https://cloud.google.com/storage/docs/objects. Accessed: Apr. 23, 2026.

[7] S. I. Serengil, "deepface," GitHub repository, n.d. Online. Available: https://github.com/serengil/deepface. Accessed: Apr. 23, 2026.

[8] MongoDB, "MongoDB Vector Search Overview," MongoDB Docs, n.d. Online. Available: https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/. Accessed: Apr. 23, 2026.

[9] LangChain, "LangGraph overview," Docs by LangChain, n.d. Online. Available: https://docs.langchain.com/oss/python/langgraph. Accessed: Apr. 23, 2026.

[10] LangChain, "LangSmith docs," Docs by LangChain, n.d. Online. Available: https://docs.langchain.com/langsmith/reference-overview. Accessed: Apr. 23, 2026.

[11] Neo4j, "Neo4j AuraDB overview," Neo4j Aura Documentation, n.d. Online. Available: https://neo4j.com/docs/aura/auradb/. Accessed: Apr. 23, 2026.

[12] National Institute of Standards and Technology, *Contingency Planning Guide for Federal Information Systems (NIST SP 800-34 Rev. 1)*, Gaithersburg, MD, USA, 2010. Online. Available: https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-34r1.pdf. Accessed: Apr. 23, 2026.

[13] J. Nielsen, "Website Response Times," Nielsen Norman Group, n.d. Online. Available: https://www.nngroup.com/articles/website-response-times/. Accessed: Apr. 23, 2026.

[14] S. Es et al., "RAGAS: Automated Evaluation of Retrieval Augmented Generation," arXiv:2309.15217, 2023. Online. Available: https://arxiv.org/abs/2309.15217. Accessed: Apr. 23, 2026.

[15] Ragas, "Faithfulness," Ragas Documentation, n.d. Online. Available: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/faithfulness/. Accessed: Apr. 23, 2026.

[16] Ragas, "Answer Relevancy," Ragas Documentation, n.d. Online. Available: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/answer_relevance/. Accessed: Apr. 23, 2026.

[17] International Organization for Standardization, "ISO 9241-11: Ergonomics of human-system interaction - Part 11: Usability: Definitions and concepts," ISO, n.d. Online. Available: https://www.iso.org/obp/ui/. Accessed: Apr. 23, 2026.

[18] Kerberos.io, "Machinery," Kerberos.io Documentation, n.d. Online. Available: https://doc.kerberos.io/opensource/machinery/. Accessed: Apr. 23, 2026.

[19] ISO/IEC, "ISO/IEC 25010 - System and Software Quality Models," ISO 25000, n.d. Online. Available: https://iso25000.com/index.php/en/iso-25000-standards/iso-25010. Accessed: Apr. 23, 2026.

[20] ONVIF, "ONVIF Core Specification," ONVIF, n.d. Online. Available: https://www.onvif.org/specs/core/ONVIF-Core-Specification.pdf. Accessed: Apr. 23, 2026.

[21] Google Cloud, "Cloud Storage best practices," Google Cloud Documentation, n.d. Online. Available: https://cloud.google.com/storage/docs/best-practices. Accessed: Apr. 23, 2026.

[22] R. Fielding, M. Nottingham, and J. Reschke, "RFC 9110: HTTP Semantics," Internet Engineering Task Force, 2022. Online. Available: https://datatracker.ietf.org/doc/html/rfc9110. Accessed: Apr. 29, 2026.

[23] IETF, "RFC 7826: Real-Time Streaming Protocol Version 2.0," Internet Engineering Task Force, 2016. Online. Available: https://datatracker.ietf.org/doc/html/rfc7826. Accessed: Apr. 29, 2026.

[24] IETF, "RFC 7578: Returning Values from Forms: multipart/form-data," Internet Engineering Task Force, 2015. Online. Available: https://datatracker.ietf.org/doc/html/rfc7578. Accessed: Apr. 29, 2026.

[25] MDN Web Docs, "HTTP range requests," Mozilla, n.d. Online. Available: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Range_requests. Accessed: Apr. 29, 2026.

[26] MDN Web Docs, "Media type and format guide," Mozilla, n.d. Online. Available: https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats. Accessed: Apr. 29, 2026.

[27] MDN Web Docs, "The Video Embed element," Mozilla, n.d. Online. Available: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video. Accessed: Apr. 29, 2026.

[28] Google Cloud, "Cloud Storage introduction," Google Cloud Documentation, n.d. Online. Available: https://docs.cloud.google.com/storage/docs/introduction. Accessed: Apr. 29, 2026.

[29] Google Cloud, "Pub/Sub basics," Google Cloud Documentation, n.d. Online. Available: https://docs.cloud.google.com/pubsub/docs/pubsub-basics. Accessed: Apr. 29, 2026.

[30] Google Cloud, "Pub/Sub architecture," Google Cloud Documentation, n.d. Online. Available: https://docs.cloud.google.com/pubsub/architecture. Accessed: Apr. 29, 2026.

[31] OWASP Foundation, "File Upload Cheat Sheet," OWASP Cheat Sheet Series, n.d. Online. Available: https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html. Accessed: Apr. 29, 2026.

[32] OpenAI, "Vector embeddings," OpenAI API Documentation, n.d. Online. Available: https://developers.openai.com/api/docs/guides/embeddings. Accessed: Jun. 17, 2026.

[33] OpenAI, "Working with evals," OpenAI API Documentation, n.d. Online. Available: https://developers.openai.com/api/docs/guides/evals. Accessed: Jun. 17, 2026.

[34] OpenAI, "Usage policies," OpenAI Policies, Oct. 29, 2025. Online. Available: https://openai.com/policies/usage-policies/. Accessed: Jun. 17, 2026.

[35] Google Cloud, "Define your evaluation metrics," Gemini Enterprise Agent Platform Documentation, n.d. Online. Available: https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/determine-eval. Accessed: Jun. 17, 2026.
```

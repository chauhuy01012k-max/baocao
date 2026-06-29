# Defense Notes for the VMS Evaluation Section

## 1. Scope

This file is the English defense version for the VMS evaluation section. It is based on:

- [M3CP2026_Group4_LeCongThanhKhoa_TranChauHuy_PhungDoAnhKhoa_CMCTelecom.docx.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/final_temp-and_VSS/M3CP2026_Group4_LeCongThanhKhoa_TranChauHuy_PhungDoAnhKhoa_CMCTelecom.docx.md>)
- [VMS_TEST_CASE_MATRIX.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/VMS_TEST_CASE_MATRIX.md>)
- [VMS_TEST_CASE_MATRIX_vi.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/VMS_TEST_CASE_MATRIX_vi.md>)
- [M3_VMS_playback_report_en.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/M3_VMS_playback_report_en.md>)
- [`main.py`](/home/huy/vss_ai_workflow/main.py)
- [`front-end/vms_epic3.js`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js)

## 2. Safe overall answer

If the panel asks whether the VMS evaluation has a real basis, the safest answer is:

1. The evaluation criteria are not arbitrary.
2. They are supported by:
   - the actual playback workflow of the implemented system,
   - protocol and browser-media references,
   - usability and response-time references.
3. However, the references justify why the metrics are meaningful; they do not always directly prescribe the exact numeric threshold values.
4. Therefore, the exact thresholds should be described as prototype acceptance thresholds guided by literature-informed engineering judgment.

## 3. What the evaluation is actually measuring

The retained VMS test cases are:

1. event-to-playback availability
2. playback open time
3. playback seek latency
4. playback media error rate

This is a reasonable evaluation scope because the strongest implemented VMS workflow is playback-centered evidence review.

## 4. Why these metrics are academically reasonable

## 4.1 Event-to-playback availability

This measures whether a surfaced event or clip entry can actually be opened as playback.

It is meaningful because the VMS is not only storing files. It is supposed to turn evidence entries into reviewable playback actions.

## 4.2 Playback open time

This measures the time from clip selection to usable playback visibility.

It is relevant because operator review loses value if opening a clip is slow or disruptive.

## 4.3 Seek latency

This measures the time from a seek action to the appearance of the target or near-target frame.

It is one of the most important playback metrics because investigation workflows rely heavily on jumping to suspicious moments rather than watching linearly from the beginning.

## 4.4 Media error rate

This measures playback failures such as load, decode, unsupported-format, or other browser-facing playback errors.

It is essential because a fast system that frequently fails to play video is still unusable.

## 5. Which references are being used and why

The VMS test matrix uses the following sources:

- ONVIF Profile G
- ONVIF Profile G press release
- Google Core Web Vitals
- Nielsen Norman Group response-time guidance
- ITU-T P.1203
- WHATWG HTML Standard for media elements
- W3C Media Source Extensions
- RFC 9110 HTTP Semantics

These sources do not all serve the same purpose. Some justify the relevance of the metric, while others justify the protocol or browser-media concepts behind the metric.

## 6. Reference-by-reference defense

## 6.1 ONVIF Profile G

### What it supports

It supports the domain relevance of recorded video playback and review in surveillance systems.

### What it does not do

It does not directly prescribe a `>=95%` event-to-playback threshold.

### Safe wording

ONVIF supports why the metric matters in the surveillance domain, not the exact numeric pass criterion.

## 6.2 Google Core Web Vitals

### What it supports

It supports the general principle that user-facing responsiveness should be evaluated.

### What it does not do

It does not directly define the VMS playback-open threshold for this capstone.

### Safe wording

Core Web Vitals helps justify why response time matters, but the exact acceptance threshold remains our prototype-level engineering decision.

## 6.3 Nielsen Norman Group response times

### What it supports

It supports the usability importance of response time and perceived waiting.

### What it does not do

It does not act as a strict surveillance-specific playback standard.

### Safe wording

This source supports the human-factors relevance of open time and seek latency, not a single mandatory threshold value for all systems.

## 6.4 ITU-T P.1203

### What it supports

It supports the idea that media startup and delivery behavior are legitimate quality-of-experience concerns.

### What it does not do

It does not directly generate the exact VMS thresholds used in the retained test cases.

### Safe wording

P.1203 supports why playback startup quality matters in media delivery, but not our exact capstone pass numbers by itself.

## 6.5 WHATWG HTML media elements

### What it supports

It strongly supports the relevance of:

- playback startup behavior,
- seeking behavior,
- browser media error states.

### What it does not do

It does not define exact timing thresholds for this capstone.

### Safe wording

The HTML media specification justifies the type of metrics, not the precise numeric targets.

## 6.6 W3C Media Source Extensions

### What it supports

It supports the broader technical context of browser-based media playback behavior.

### Important caution

The current codebase does not directly implement a custom MSE pipeline. It mostly relies on browser `<video>` playback over HTTP delivery.

### Safe wording

MSE is useful as broader browser-media context, but it should not be presented as if the current code explicitly implements a full MSE-based playback architecture.

## 6.7 RFC 9110 HTTP Semantics

### What it supports

This is one of the strongest references for the playback path because the code uses:

- `Range`
- `206 Partial Content`
- `416 Range Not Satisfiable`

### What it does not do

It does not prescribe the actual latency values for seek behavior.

### Safe wording

RFC 9110 justifies the protocol correctness of byte-range playback, while the measured seek latency remains an empirical system result.

## 7. Where the numeric thresholds come from

This is the most important methodological point.

The thresholds in the retained VMS matrix include:

- `>=95%` event-to-playback availability
- `median <=5s` playback open time
- `median <=2s`, `p95 <=5s` seek latency
- `<=5%` media error rate

## 7.1 The honest answer

These numbers are not copied directly from one external standard.

They are:

- acceptance thresholds for the prototype,
- informed by literature,
- aligned with the implemented playback workflow,
- and chosen as reasonable engineering targets for a controlled capstone demonstration.

### Safe academic statement

The literature supports the choice of the metric categories, while the exact numeric thresholds are prototype acceptance criteria chosen through engineering judgment.

## 7.2 Why the thresholds are still defensible

### `>=95%` availability

This is a high reliability target for controlled playback review without claiming absolute perfection.

### `<=5s` open time

This is a usability-oriented target for clip startup in a review workflow.

### `<=2s median`, `<=5s p95` seek latency

This reflects the fact that seek operations are more critical to investigation than many ordinary UI actions. Using both median and p95 is stronger than using only average.

### `<=5%` media error rate

This creates a controlled tolerance threshold for prototype acceptance, while allowing the retained result to still outperform it.

## 8. How to interpret the retained results correctly

The retained VMS results include:

- average open time = `2.64 s`
- median seek latency = `1.83 s`
- p95 seek latency = `2.51 s`
- media error rate = `0%`
- event-to-playback availability = `100%`

### What can be defended

- the playback-centered workflow is practically usable in the evaluated controlled path,
- the Range-based seek design appears to function properly,
- the browser-facing playback path is stable in the retained test set.

### What should not be overclaimed

- these results do not by themselves prove production-scale readiness,
- they do not prove universal multi-browser, multi-network robustness,
- they do not prove fully automated instrumentation inside the repository code.

## 9. Important methodological limitations to admit

## 9.1 Retained evidence is not the same as fully instrumented benchmarking

The codebase itself does not visibly contain a complete built-in benchmarking subsystem for these metrics. Therefore, the evaluation should be described as retained controlled-test evidence, not as a fully self-proving runtime telemetry system.

## 9.2 References justify metric relevance more directly than exact thresholds

This distinction should be stated clearly.

The references are strongest when used to explain:

- why startup time matters,
- why seek behavior matters,
- why media errors matter,
- why playback availability matters.

The references are weaker if used as if they directly impose the exact threshold numbers in the test table.

## 9.3 The evaluation is strongest for the playback workflow core, not for the entire VMS product space

This is the correct capstone framing and should be preserved in the defense.

## 10. Strong short answers for panel questions

### “Did you invent these evaluation criteria?”

No. The metric categories were derived from the actual playback workflow and supported by references on surveillance recording, HTTP partial content, browser media behavior, and usability.

### “Did the references directly give you the exact numbers?”

Not exactly. The references justify why those metrics matter. The exact numeric thresholds are our prototype acceptance thresholds, set through literature-informed engineering judgment.

### “Why use median and p95?”

Because average can hide tail behavior. In playback review, a few abnormally slow seeks still matter to the operator experience.

### “Does 0% error rate mean the system is finished?”

No. It means no playback failures were observed in the retained controlled test set. It does not by itself prove full production readiness.

## 11. Safe closing statement

The VMS evaluation section is defensible when presented carefully. Its strength is that the metrics are tied to the real playback workflow and supported by appropriate protocol, browser-media, and usability references. Its necessary limitation is that the references justify the importance of the metrics more directly than they justify the exact threshold values. So the correct academic framing is that the VMS evaluation is literature-informed, workflow-grounded, and appropriate for a playback-centered capstone prototype.

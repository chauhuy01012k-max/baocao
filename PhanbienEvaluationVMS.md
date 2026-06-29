# Phản Biện Phần Evaluation Của VMS

## 1. Phạm vi của tài liệu này

Tài liệu này chỉ tập trung vào **phần evaluation của VMS** trong báo cáo M3, đặc biệt là:

- mục `4.4 VMS playback evaluation` trong file [M3CP2026_Group4_LeCongThanhKhoa_TranChauHuy_PhungDoAnhKhoa_CMCTelecom.docx.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/final_temp-and_VSS/M3CP2026_Group4_LeCongThanhKhoa_TranChauHuy_PhungDoAnhKhoa_CMCTelecom.docx.md>)
- bảng test case trong [VMS_TEST_CASE_MATRIX.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/VMS_TEST_CASE_MATRIX.md>) và [VMS_TEST_CASE_MATRIX_vi.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/VMS_TEST_CASE_MATRIX_vi.md>)
- tài liệu phụ [M3_VMS_playback_report_en.md](</home/huy/drive-download-20260330T022318Z-3-001/M2/M3_VMS_playback_report_en.md>)
- code thật trong [`main.py`](/home/huy/vss_ai_workflow/main.py) và [`front-end/vms_epic3.js`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js)

Mục tiêu của file này là trả lời ba câu hỏi:

1. Tiêu chí evaluation của VMS có hợp lý không?
2. Các tiêu chí đó có căn cứ từ ref hay không?
3. Khi bị phản biện sâu, cần nói thế nào để vừa học thuật vừa không bịa?

## 2. Kết luận nhanh để dùng khi bị hỏi tổng quát

Nếu hội đồng hỏi:

`Evaluation của VMS có cơ sở hay không, hay nhóm tự đặt ra?`

câu trả lời an toàn là:

1. Evaluation của VMS **không phải tự nhiên đặt ra hoàn toàn tùy ý**.
2. Nhóm đã dựa trên ba loại căn cứ:
   - căn cứ **kiến trúc và giao thức** của hệ thống thật,
   - căn cứ **usability / response time** từ các tài liệu tham chiếu,
   - căn cứ **video/web playback semantics** từ chuẩn HTTP và tài liệu media web.
3. Tuy nhiên, các ref này **không trực tiếp sinh ra đúng các con số threshold cụ thể** như `median <= 2s` hoặc `median <= 5s` theo kiểu bắt buộc tuyệt đối. Chúng chủ yếu cung cấp nền tảng học thuật để nói rằng các tiêu chí như open time, seek latency, media error rate, event-to-playback availability là hợp lý để đo.
4. Vì vậy, khi bảo vệ, phải nói rằng:
   - các metric là **evaluation criteria derived from literature-informed engineering judgment**,
   - còn số threshold cụ thể là **acceptance thresholds cho prototype playback workflow**, không phải industry law bất biến.

## 3. Những gì evaluation của VMS đang đo

Theo M3 và test matrix, phần evaluation đang tập trung vào 4 retained test cases:

1. `VMS-TC-01` event-to-playback availability
2. `VMS-TC-02` playback open time
3. `VMS-TC-03` playback seek latency
4. `VMS-TC-04` playback media error rate

Ngoài ra, M3 còn chia logic test thành 4 nhóm:

- functional
- integration
- boundary and error
- usability and acceptance

Điều quan trọng là phải giải thích được:

- nhóm không đo “video có phát được hay không” theo kiểu sơ sài,
- mà đang đo các khía cạnh tương ứng với workflow playback-centered thật của hệ thống.

## 4. Vì sao các metric này là hợp lý về mặt học thuật

## 4.1 Event-to-playback availability

Metric này đo:

- từ một event entry hoặc clip entry surfaced trong workflow,
- người dùng có mở playback thành công hay không.

Đây là metric hợp lý vì:

- VMS không chỉ là file host,
- nó là tầng review evidence,
- nên tính khả dụng của đường đi từ “evidence surfaced” đến “playback thực sự mở được” là cốt lõi.

Nếu bị hỏi:

`Tại sao không chỉ kiểm tra file tồn tại trong storage là đủ?`

Trả lời:

Vì tồn tại trong storage chưa chứng minh được người dùng mở được clip qua frontend/backend chain. Event-to-playback availability kiểm tra một điều có ý nghĩa vận hành hơn: evidence object có thực sự chuyển thành playback action usable hay không.

## 4.2 Playback open time

Metric này đo thời gian từ lúc chọn clip đến khi video hoặc timeline khả dụng trên giao diện.

Nó hợp lý vì:

- playback có ý nghĩa vận hành nếu người dùng không phải chờ quá lâu,
- startup delay là một chỉ báo trực tiếp của tính usable trong tác vụ điều tra.

Đây không chỉ là metric frontend thuần túy, vì nó nén nhiều thành phần:

- object listing,
- object retrieval,
- response path,
- browser decode readiness,
- phần nào đó cả MP4 suitability và delivery behavior.

## 4.3 Seek latency

Metric này đo thời gian từ lúc seek đến khi frame gần timestamp mục tiêu được hiển thị.

Đây là metric rất quan trọng vì:

- nó kiểm tra đúng use case investigation,
- người vận hành video surveillance thường không xem tuần tự từ đầu đến cuối,
- họ nhảy đến đoạn nghi vấn.

Về kỹ thuật, seek latency còn là chỉ báo gián tiếp cho:

- HTTP Range correctness,
- byte-range parsing,
- partial streaming,
- browser/player interaction.

## 4.4 Media error rate

Metric này đo:

- lỗi load video,
- lỗi decode,
- lỗi unsupported format,
- lỗi playback-failure.

Đây là metric đúng vì nếu playback open time nhanh nhưng media error rate cao thì hệ thống vẫn không usable.

Nó đánh giá tính ổn định end-to-end của đường:

`normalized file -> GCS object -> VMS proxy -> browser decode`

## 5. Các ref nào đang được dùng để biện minh cho evaluation

Từ `VMS_TEST_CASE_MATRIX`, các nguồn được dùng là:

- ONVIF Profile G
- ONVIF Profile G press release
- Google Core Web Vitals
- Nielsen Norman Group response times
- ITU-T P.1203
- WHATWG HTML Standard: media elements
- W3C Media Source Extensions

Trong M3 còn dẫn thêm:

- RFC 9110 HTTP Semantics
- FastAPI docs
- Google Cloud Storage docs

Phải phân tích đúng vai trò của từng nhóm ref.

## 6. Phản biện theo từng ref và mức độ phù hợp

## 6.1 ONVIF Profile G và ONVIF press release

### 6.1.1 Ref này dùng để làm gì

Trong test matrix, ONVIF Profile G được dùng để support cho `event-to-playback availability`.

Ý tưởng nền là:

- surveillance recording/replay là bài toán chính danh trong hệ sinh thái camera/VMS,
- event-based recorded video phải mở được để phục vụ review.

### 6.1.2 Điểm hợp lý

Ref này hợp lý ở mức:

- nó cho thấy recorded-video access và replay là một yêu cầu chính đáng trong surveillance domain,
- tức là đo availability của event-to-playback không phải là metric bịa ra vô căn cứ.

### 6.1.3 Điểm phải nói cẩn thận

ONVIF Profile G **không trực tiếp cho ra threshold `>=95%`**.

Nó chỉ là căn cứ domain-level rằng:

- recording và playback là năng lực cốt lõi cần kiểm tra.

Do đó phải nói:

`ONVIF Profile G supports the relevance of the metric, not the exact numeric acceptance threshold.`

Đây là câu rất quan trọng.

## 6.2 Google Core Web Vitals

### 6.2.1 Ref này dùng để làm gì

Trong test matrix, Core Web Vitals được dùng để support cho `playback open time`.

### 6.2.2 Điểm hợp lý

Ref này hợp lý ở mức rộng:

- nó cho thấy trải nghiệm người dùng có thể và nên được lượng hóa bằng các chỉ báo thời gian,
- tức là phản hồi giao diện không nên được nói bằng cảm giác.

### 6.2.3 Điểm phải nói cẩn thận

Core Web Vitals không phải chuẩn chuyên biệt cho playback-open-time của video surveillance.

Nó là framework web performance tổng quát.

Vì vậy:

- dùng nó để biện minh cho việc “nên đo độ trễ perceptible user-facing” là hợp lý,
- nhưng không nên nói Core Web Vitals ra lệnh rằng playback open time phải là 5 giây hay 2.64 giây.

## 6.3 Nielsen Norman Group response times

### 6.3.1 Ref này dùng để làm gì

Ref này thường được dùng để biện minh cho:

- open time,
- seek latency,
- phản hồi giao diện có còn trong vùng chấp nhận được cho người dùng hay không.

### 6.3.2 Điểm hợp lý

Nielsen Norman Group là tài liệu kinh điển về ngưỡng cảm nhận phản hồi của người dùng.

Nó cho cơ sở để nói:

- tốc độ phản hồi ảnh hưởng usability,
- vài giây chờ đợi là một yếu tố thiết kế phải quan tâm.

### 6.3.3 Điểm phải nói cẩn thận

Nó vẫn là tài liệu HCI/usability tổng quát, không phải chuẩn chuyên ngành VMS playback.

Vì vậy nên nói:

`This source justifies why response time matters to operators; it does not uniquely determine our exact playback thresholds.`

## 6.4 ITU-T P.1203

### 6.4.1 Ref này dùng để làm gì

Trong test matrix, P.1203 được dùng support cho playback open time.

### 6.4.2 Điểm hợp lý

ITU-T P.1203 là tài liệu về quality assessment cho progressive/adaptive streaming. Nó có giá trị vì:

- playback trên web là trải nghiệm media có thể đánh giá bằng latency và delivery behavior,
- metric open time có liên quan đến QoE.

### 6.4.3 Điểm phải nói cẩn thận

P.1203 không phải sinh ra trực tiếp metric “open clip in VMS from event entry”.

Nó chỉ cung cấp nền tảng rằng:

- startup behavior và playback quality là hợp lệ để đánh giá media delivery.

Do đó:

- ref này hợp lý để hỗ trợ **tại sao playback startup matters**,
- nhưng không phải bằng chứng duy nhất cho exact threshold.

## 6.5 WHATWG HTML media elements

### 6.5.1 Ref này dùng để làm gì

Ref này support cho:

- seek behavior,
- media error rate,
- semantics của browser video playback.

### 6.5.2 Điểm hợp lý

Đây là ref rất mạnh cho evaluation VMS vì playback cuối cùng xảy ra trên browser media element.

Nó cho cơ sở học thuật rằng:

- media element có vòng đời playback,
- có loading, seeking, decode, error states,
- nên việc đo startup, seek, media errors là chính đáng.

### 6.5.3 Điểm phải nói cẩn thận

Spec WHATWG định nghĩa hành vi và khái niệm, không cấp ngưỡng performance cụ thể.

Nó phù hợp nhất để biện minh cho:

- **loại metric**,
- chứ không phải **số threshold**.

## 6.6 W3C Media Source Extensions

### 6.6.1 Ref này dùng để làm gì

Ref này được dùng để support:

- seek/playback behavior,
- browser-oriented media delivery reasoning.

### 6.6.2 Điểm hợp lý

Nó hợp lý vì:

- cho thấy browser playback là một bài toán kỹ thuật có quy ước chính thức,
- media delivery trên web không phải khái niệm ad hoc.

### 6.6.3 Điểm phải nói cẩn thận

Trong code hiện tại, hệ thống không tự triển khai MSE trực tiếp bằng JavaScript API phức tạp; nó chủ yếu dựa vào browser `<video>` + HTTP delivery.

Vì vậy:

- MSE chỉ nên xem là ref hỗ trợ bối cảnh media-web playback rộng hơn,
- không nên nói code hiện tại “implements W3C MSE pipeline”.

## 6.7 RFC 9110 HTTP Semantics

### 6.7.1 Ref này dùng để làm gì

Đây là ref rất quan trọng cho `seek latency` và `partial content behavior`, vì code dùng:

- `Range`
- `206 Partial Content`
- `416 Range Not Satisfiable`

### 6.7.2 Điểm hợp lý

Đây là ref chuẩn nhất để nói rằng:

- byte-range playback là đúng giao thức,
- việc dùng `206`, `Content-Range`, `Accept-Ranges` là có căn cứ chuẩn.

### 6.7.3 Điểm phải nói cẩn thận

RFC 9110 hợp thức hóa semantics giao thức, nhưng không tự quyết định seek latency phải bao nhiêu giây.

Nó giúp bảo vệ:

- vì sao metric seek là hợp lý,
- vì sao nếu seek hoạt động tốt thì điều đó phản ánh Range design đúng.

## 7. Phản biện phần pass criteria: từ đâu ra các threshold?

Đây là chỗ dễ bị hội đồng bắt nhất.

Các threshold hiện có:

- `>= 95%` event-to-playback availability
- `median <= 5s` playback open time
- `median <= 2s`, `p95 <= 5s` seek latency
- `<= 5%` media error rate

## 7.1 Phải trả lời trung thực

Các threshold này **không phải được copy trực tiếp 1:1 từ một chuẩn duy nhất**.

Chúng là:

- ngưỡng chấp nhận được do nhóm đặt ra cho prototype,
- được định hình bởi:
  - tính chất nhiệm vụ điều tra video,
  - tài liệu về usability/response time,
  - semantics của media playback và HTTP,
  - và mức kỳ vọng hợp lý cho một demo-capable capstone.

Nói cách khác:

`the metrics are literature-informed, but the exact thresholds are engineering acceptance thresholds`

## 7.2 Vì sao `>= 95%` availability là hợp lý

Đây là ngưỡng thể hiện:

- workflow playback phải đáng tin cậy cao trong controlled demo,
- nhưng vẫn chừa khoảng đệm thay vì đòi 100% tuyệt đối như production hard guarantee.

Nó không đến từ ONVIF như một con số bắt buộc.

### Câu trả lời mẫu

`The 95% threshold is not an ONVIF-prescribed number. It is our acceptance threshold for a prototype that is supposed to support consistent operator review in controlled testing.`

## 7.3 Vì sao `median <= 5s` cho open time là hợp lý

Ngưỡng này có thể được bảo vệ theo hướng usability:

- startup quá chậm sẽ làm gián đoạn review,
- một playback path dùng cho operator investigation cần phản hồi trong khoảng vẫn chấp nhận được về mặt thao tác.

Nhưng phải tránh nói:

`Nielsen says 5 seconds exactly`

Điều đúng hơn là:

- Nielsen/Core Web Vitals cho ta cơ sở rằng response time matters,
- còn `5s` là ngưỡng acceptance nhóm chọn cho context capstone prototype.

## 7.4 Vì sao `median <= 2s`, `p95 <= 5s` cho seek là hợp lý

Lập luận học thuật:

- seek là thao tác investigation-critical,
- thao tác này phải nhanh hơn open time,
- vì nó là điều chỉnh trong khi người dùng đang ở giữa workflow review.

Việc dùng cả median và p95 là hợp lý vì:

- median đo common case,
- p95 đo tail behavior,
- với playback, tail latency quan trọng vì vài lần seek chậm bất thường cũng có thể làm hỏng trải nghiệm điều tra.

### Câu trả lời mẫu

`The thresholds are not directly mandated by a single standard. They reflect a usability-oriented acceptance target for surveillance review, supported by literature on response time and by the protocol importance of efficient byte-range playback.`

## 7.5 Vì sao `<= 5%` media error rate là hợp lý

Ngưỡng này thể hiện:

- hệ thống playback phải đủ ổn định để demo và review,
- nhưng nhóm không claim zero-failure universally.

Trong retained result nhóm có `0%`, nhưng ngưỡng pass là `<= 5%`.

Điều này thực ra là hợp lý về phương pháp:

- pass criterion không nên bị thiết kế sau khi nhìn kết quả,
- cần chừa mức tolerance hợp lý cho controlled test.

## 8. Phản biện tính đúng đắn của cách diễn giải kết quả

## 8.1 “Average open time = 2.64s”

### Điều có thể bảo vệ

- chứng minh workflow từ chọn clip đến mở clip là usable,
- chứng minh delivery chain không quá chậm trong controlled path.

### Điều không nên overclaim

- không chứng minh scale production,
- không chứng minh multi-user concurrency,
- không chứng minh mọi network condition.

## 8.2 “Median seek = 1.83s, p95 = 2.51s”

### Điều có thể bảo vệ

- seek behavior thực dụng cho investigation,
- Range-based playback likely đang hoạt động đúng ở tested path.

### Điều không nên overclaim

- không chứng minh tối ưu tuyệt đối,
- không chứng minh behavior dưới object lớn hơn nhiều hoặc mạng yếu hơn,
- không chứng minh browser diversity rộng nếu chưa nêu rõ test environment.

## 8.3 “Error rate = 0%”

### Điều có thể bảo vệ

- trong controlled retained test set, không quan sát thấy lỗi playback.

### Điều phải nói cẩn thận

Nên dùng đúng câu:

`No playback failures were observed in the retained controlled test set.`

Không nên nói:

`The playback system has zero errors.`

## 8.4 “Event-to-playback availability = 100%”

### Điều có thể bảo vệ

- mapping từ event/clip surfaced đến playback action thành công trong test set là đáng tin cậy.

### Điều phải nói cẩn thận

Nó vẫn phụ thuộc:

- kích thước test set,
- cách chọn mẫu,
- điều kiện kiểm soát,
- việc “event entries” được định nghĩa ra sao.

Nếu bị hỏi sâu mà không có số mẫu ngay:

phải thừa nhận đó là retained test evidence, không phải universal theorem.

## 9. Những giới hạn phương pháp luận phải chủ động nêu ra

Đây là phần rất quan trọng để tăng độ tin cậy học thuật.

## 9.1 Các số liệu là retained evaluation, không phải benchmark framework tự động hóa toàn phần trong code

Code hiện tại trong repo không cho thấy:

- một subsystem benchmark tích hợp sẵn,
- hay một telemetry pipeline nội bộ tự log đầy đủ tất cả metric đó.

Do đó phải nói:

- đây là `retained evaluation evidence`,
- có thể đến từ controlled tests, matrix, hoặc supplementary measurement notes.

Đây không làm evaluation vô giá trị, nhưng phải được nói đúng bản chất.

## 9.2 Ref biện minh cho metric, không phải luôn biện minh trực tiếp cho con số threshold

Đây là điểm cốt lõi nhất.

Phải phân biệt:

- `metric validity`
- `threshold choice`

Ref thường biện minh rất tốt cho việc:

- vì sao nên đo startup,
- vì sao nên đo seek,
- vì sao media errors là quan trọng,
- vì sao playback availability là meaningful.

Nhưng threshold cụ thể vẫn là do nhóm đặt ra như acceptance criteria cho prototype.

## 9.3 Evaluation mạnh nhất cho playback-centered core, không phải cho toàn bộ VMS product space

M3 đã viết đúng hướng này, và cần tiếp tục giữ.

Nếu bị hỏi:

`Evaluation này có chứng minh toàn bộ VMS hoàn chỉnh không?`

Trả lời:

Không. Nó chứng minh phần playback-centered workflow core là mạnh và usable trong controlled scenarios.

## 10. Cách trả lời khi hội đồng hỏi “dựa vào đâu mà đánh giá như vậy?”

Đây là phần có thể dùng gần như nguyên văn.

### 10.1 Câu trả lời ngắn

Phần evaluation của VMS không được đặt ra tùy ý. Nhóm xây dựng tiêu chí dựa trên ba tầng căn cứ. Thứ nhất là căn cứ từ workflow thật của hệ thống: upload, normalize, cloud playback, Range seek, analysis handoff. Thứ hai là căn cứ từ chuẩn giao thức và media-web như RFC 9110, WHATWG media elements, W3C media specs để xác định loại hành vi nào là quan trọng để đo. Thứ ba là căn cứ từ usability/performance references như Nielsen Norman Group, Core Web Vitals, ITU-T P.1203 để biện minh rằng response time và playback quality là các chỉ báo có ý nghĩa đối với trải nghiệm người dùng.

### 10.2 Câu trả lời sâu hơn

Các ref chủ yếu được dùng để chứng minh rằng những metric như open time, seek latency, media error rate, và event-to-playback availability là hợp lý để đánh giá một playback-centered VMS. Còn các threshold số học cụ thể như `5 giây`, `2 giây`, `95%` là acceptance thresholds cho prototype, được chọn bằng kỹ thuật judgment có tham chiếu tài liệu, chứ không phải là các giá trị được một chuẩn duy nhất áp đặt.

## 11. Bộ câu hỏi phản biện mẫu và câu trả lời

### 11.1 “Các số này có từ đâu ra, code có tự log ra không?”

Trả lời:

Không nên nói code tự log toàn bộ nếu chưa có bằng chứng instrumentation. Cách nói đúng là đây là retained evaluation results của playback workflow trong controlled testing, được trình bày lại trong M3 và test matrix.

### 11.2 “Dựa vào tài liệu nào mà nói open time hay seek latency quan trọng?”

Trả lời:

Về mặt media delivery và browser playback, có HTTP semantics, HTML media behavior, và media playback specs làm căn cứ cho việc playback startup và seeking là các hành vi cốt lõi. Về mặt usability, các tài liệu như Nielsen Norman Group và web-performance guidance cho thấy response time là một chỉ báo có ý nghĩa đối với trải nghiệm người dùng.

### 11.3 “Vậy 5 giây hay 2 giây là chuẩn nào?”

Trả lời:

Đó không phải là con số được copy nguyên xi từ một chuẩn duy nhất. Đó là acceptance thresholds nhóm chọn cho prototype, dựa trên literature-informed engineering judgment và mục tiêu rằng playback phải đủ nhanh cho review điều tra trong controlled scenarios.

### 11.4 “Tại sao dùng median và p95 mà không chỉ dùng average?”

Trả lời:

Average dễ che lấp tail behavior. Với video seek, một số lần chậm bất thường vẫn ảnh hưởng mạnh đến trải nghiệm operator. Median cho common case, còn p95 cho near-tail behavior, nên cách này học thuật hơn và trung thực hơn.

### 11.5 “Tại sao 0% error rate chưa đủ để nói hệ thống production-ready?”

Trả lời:

Vì 0% đó là trong retained controlled test set. Production readiness còn đòi hỏi coverage rộng hơn: nhiều mạng, nhiều browser, concurrency, monitoring, auth, failure recovery, và vận hành dài hạn.

## 12. Cách phát biểu an toàn trong slide hoặc khi bảo vệ

Nên nói:

- `Our VMS evaluation criteria were derived from the actual playback workflow and supported by protocol and usability references.`
- `The literature supports the relevance of the metrics; the exact thresholds are prototype acceptance thresholds chosen by engineering judgment.`
- `The retained results show that the playback-centered workflow is practically usable in controlled scenarios.`
- `These results validate the strongest workflow core of the current VMS, not the completeness of every VMS module.`

Không nên nói:

- `These thresholds are mandated directly by standards.`
- `The code automatically proves all evaluation values.`
- `0% error means the system is complete.`
- `The references directly prescribe our exact numeric targets.`

## 13. Kết luận cuối cùng

Phần evaluation của VMS có thể bảo vệ tốt nếu được trình bày đúng bản chất. Điểm mạnh của nó là metric không bị chọn ngẫu nhiên: chúng bám chặt vào workflow playback thật của hệ thống và được biện minh bởi các ref về surveillance recording, HTTP partial content, browser media behavior, và usability/performance. Điểm phải giữ kỷ luật học thuật là không được thổi phồng vai trò của ref: các ref làm cơ sở cho việc chọn loại metric và đánh giá tính quan trọng của chúng, còn các threshold số học cụ thể là acceptance criteria của nhóm cho một prototype playback-centered capstone. Đây là cách nói vừa đúng, vừa chắc, vừa khó bị phản biện bắt lỗi.

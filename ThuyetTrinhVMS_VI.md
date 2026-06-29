# Ghi Chú Thuyết Trình Phần VMS

## Mục tiêu

Phần này chỉ tập trung vào VMS. Khi thuyết trình, cần nhấn mạnh 3 ý:

1. VMS là trung tâm của workflow review video.
2. Điểm mạnh nhất của VMS hiện tại là playback-centered workflow.
3. VMS kết nối edge upload, cloud playback, và AI-assisted analysis trong một luồng thống nhất.

## Slide: Research Questions for VMS

- Phần VMS của nhóm xoay quanh 3 câu hỏi.
- Câu hỏi thứ nhất là làm sao playback video từ cloud đủ nhanh để phục vụ điều tra.
- Câu hỏi thứ hai là làm sao nhận clip từ edge một cách tin cậy và web-compatible.
- Câu hỏi thứ ba là làm sao tích hợp với VSS mà không làm frontend phải hiểu logic AI nội bộ.
- Ý chốt: VMS không chỉ lưu video, mà là tầng tổ chức lại video thành tài nguyên điều tra có thể sử dụng được.

## Slide: VMS Solution Overview

- VMS được thiết kế như một lớp orchestration ở giữa frontend, storage, edge input, và VSS.
- Frontend chỉ gọi API của VMS, không truy cập trực tiếp GCS hay VSS.
- Backend FastAPI chịu trách nhiệm validate upload, convert video, proxy playback, và forward request sang VSS.
- GCS là nơi lưu object playback-ready.
- Pub/Sub là cơ chế bàn giao bất đồng bộ cho ingest phía sau.
- Ý chốt: kiến trúc này giúp playback là trung tâm, còn các thành phần khác nối vào quanh nó.

## Slide: Edge-to-VMS Upload Workflow

- Edge gửi clip lên VMS qua HTTP upload.
- VMS nhận file và metadata cùng lúc bằng multipart/form-data.
- Sau đó VMS kiểm tra loại file, kích thước, và tên file.
- Nếu hợp lệ thì hệ thống lưu file gốc, chuẩn hóa sang MP4, rồi mới tiếp tục đồng bộ cloud.
- Ý chốt: đây là cổng vào chuẩn hóa cho toàn bộ pipeline playback sau này.

## Slide: VMS Video Ingest Workflow

- Sau khi nhận file, VMS lưu raw upload vào `uploads_raw/`.
- Tiếp theo VMS dùng FFmpeg để chuẩn hóa thành MP4 tương thích trình duyệt.
- File chuẩn hóa được lưu vào `uploads/`, sau đó upload lên GCS nếu bật auto-ingest.
- Hệ thống cũng tạo metadata để phục vụ các bước downstream.
- Ý chốt: ingest ở đây không chỉ là lưu file, mà là biến clip thành tài nguyên playback-ready.

## Slide: VMS Playback Workflow

- Frontend gọi API lấy danh sách object video từ GCS.
- Khi người dùng chọn clip, frontend tạo playback URL qua backend VMS.
- Backend không bắt browser tải toàn bộ file, mà stream theo HTTP Range.
- Nhờ đó người dùng có thể seek nhanh tới đoạn cần xem.
- Ý chốt: cải tiến kỹ thuật quan trọng nhất của phần VMS là cloud-backed playback với HTTP Range.

## Slide: VMS AI Integration Workflow

- Khi người dùng mở Deep Analyze, frontend gọi API analysis thông qua VMS.
- VMS đóng vai trò trung gian, gọi service AI phía sau và trả dữ liệu về giao diện.
- Nếu người dùng đặt câu hỏi về clip, VMS bổ sung clip context như `video_id`, `camera_id`, `date`, `location`.
- Sau đó VMS forward query sang VSS và trả về `answer`, `sources`, `raw`.
- Ý chốt: playback không còn là bước cuối, mà là điểm khởi đầu cho AI-assisted investigation.

## Slide: VMS Solution

- Nếu cần tóm tắt toàn bộ phần VMS trong một câu:
- VMS nhận clip, chuẩn hóa clip, phát lại clip, rồi kết nối clip đó sang AI workflow.
- VMS là phần operator-facing rõ nhất trong toàn hệ thống hiện tại.
- Đây cũng là phần có độ hoàn thiện cao nhất để demo.

## Slide: Evaluation Results - VMS

- Evaluation của VMS tập trung vào playback-centered workflow, không dàn trải sang toàn bộ sản phẩm VMS thương mại.
- Nhóm đo 4 chỉ số chính:
- event-to-playback availability,
- playback open time,
- seek latency,
- media error rate.
- Ý chốt: nhóm đánh giá đúng phần mạnh nhất của hệ thống, thay vì claim toàn bộ VMS đã hoàn thiện.

## Slide: Evaluation Results - VMS (số liệu)

- Kết quả chính là open time trung bình khoảng 2.64 giây.
- Seek latency median khoảng 1.83 giây, p95 khoảng 2.51 giây.
- Media error rate trong test set giữ lại là 0%.
- Event-to-playback availability là 100% trong controlled retained test.
- Ý chốt: các số liệu này cho thấy playback path đã đủ usable cho review và demo điều tra.

## Slide: VMS Layer

- Nếu cần kết thúc phần VMS, hãy nhấn mạnh đúng phạm vi.
- Điểm mạnh nhất của VMS hiện tại là centralized storage and playback.
- Những phần như dashboard, live-view, event management vẫn chưa cùng mức maturity.
- Vì vậy, cách framing đúng là: playback là workflow mạnh nhất và là core đã được kiểm chứng.

## Câu chuyển sang phần tiếp theo

- Từ VMS, hệ thống không dừng ở việc xem lại clip.
- Giá trị tiếp theo là clip đó trở thành đầu vào cho VSS để phân tích, truy xuất, và trả lời câu hỏi.

## Phiên bản kết thúc ngắn

- Tóm lại, phần VMS của nhóm giải quyết 3 việc chính:
- nhận clip từ edge,
- phát lại clip hiệu quả từ cloud,
- và nối clip đó sang AI analysis.
- Đây là phần có workflow hoàn chỉnh và thuyết phục nhất trong capstone hiện tại.

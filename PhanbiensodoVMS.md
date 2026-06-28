# Phản Biện 6 Sơ Đồ VMS Dựa Trên Code Thực Tế

## 1. Phạm vi và nguyên tắc của tài liệu này

Tài liệu này được viết để phục vụ phản biện capstone cho phần VMS. Mọi kết luận bên dưới chỉ dựa trên:

- File hướng dẫn vẽ sơ đồ: `/home/huy/drive-download-20260330T022318Z-3-001/M2/HUONG_DAN_VE_SO_DO_VMS_PLAYBACK.md`
- Code backend thật đang chạy trong repo: [`main.py`](/home/huy/vss_ai_workflow/main.py)
- Code frontend playback thật: [`front-end/vms_epic3.js`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js)

Tài liệu này không dựa nguyên văn vào `README.md` vì `README.md` đang mô tả một số flow cũ, trong khi `main.py` đã đổi sang hướng GCS + Pub/Sub + gọi service AI từ xa. Nếu đem README ra trình bày như hiện trạng hệ thống, người phản biện có thể bắt lỗi.

## 2. Kết luận tổng quát về 6 sơ đồ

Nếu hỏi ngắn gọn "6 sơ đồ trong file hướng dẫn có đúng với code không?" thì câu trả lời chính xác là:

1. Sơ đồ 1 `Upload request từ edge vào VMS`: đúng về ý chính, nhưng chưa đủ chính xác ở phần điều kiện lỗi.
2. Sơ đồ 2 `Chuẩn hóa video và đồng bộ cloud`: đúng về logic cốt lõi, nhưng thực tế không phải một service tách rời mà đang nằm trong cùng request upload.
3. Sơ đồ 3 `Playback clip từ GCS lên trình duyệt`: đúng một phần, nhưng endpoint lấy danh sách clip trong code không phải `/api/playback/clips`.
4. Sơ đồ 4 `Seek và tua video bằng HTTP Range`: đây là sơ đồ khớp nhất với code backend hiện tại.
5. Sơ đồ 5 `Deep Analyze và clip-specific query`: đúng về vai trò trung gian của VMS, nhưng cần nói rõ VMS đang proxy sang service AI từ xa chứ không tự phân tích AI bên trong.
6. Sơ đồ 6 `Publish ingest event qua Pub/Sub`: đúng về ý tưởng, nhưng cần nói rõ đây là bước tùy thuộc biến môi trường `AUTO_INGEST_ON_UPLOAD`, và code hiện tại chỉ publish, không chứa logic subscriber.

## 3. Những khái niệm nền tảng phải giải thích được khi bị hỏi vặn

### 3.1 FastAPI là gì

FastAPI là một web framework cho Python dùng để xây dựng HTTP API. Nó được thiết kế theo chuẩn ASGI, tức là giao diện chuẩn cho ứng dụng web bất đồng bộ trong Python.

Trong repo này, FastAPI được khởi tạo tại [`main.py:30`](/home/huy/vss_ai_workflow/main.py:30) bằng:

```python
app = FastAPI(
    title="Video Receiver",
    docs_url=None,
    redoc_url=None,
)
```

### 3.2 FastAPI được tạo ra như thế nào về mặt kỹ thuật

Nếu bị hỏi theo hướng học thuật, có thể trả lời:

- FastAPI không phải là một web server tự thân.
- FastAPI là tầng framework định nghĩa route, validation, dependency injection, response model.
- Nó chạy trên Starlette, tức là framework ASGI nền tảng phụ trách routing, middleware, request/response lifecycle.
- Nó dùng Pydantic để kiểm tra và ép kiểu dữ liệu đầu vào.
- Nó thường được phục vụ bởi một ASGI server như Uvicorn.

Trong repo này, Uvicorn được gọi ở cuối file [`main.py:939`](/home/huy/vss_ai_workflow/main.py:939).

Nói ngắn gọn:

`Client -> Uvicorn (ASGI server) -> FastAPI/Starlette app -> hàm route trong main.py`

### 3.3 Request upload vào FastAPI là gì

Khi edge gửi video lên hệ thống, nó đang gửi một HTTP request theo kiểu `POST`, nội dung ở dạng `multipart/form-data`. Kiểu dữ liệu này cho phép một request mang cả file lẫn các field văn bản như `name`, `client_id`, `caption`.

Trong code, endpoint upload là:

- [`main.py:867`](/home/huy/vss_ai_workflow/main.py:867) `POST /api/videos`
- [`main.py:924`](/home/huy/vss_ai_workflow/main.py:924) `POST /upload` là alias

Field file được nhận bằng `UploadFile = File(...)`. Đây là abstraction của FastAPI/Starlette cho file upload.

### 3.4 MIME type là gì

MIME type là nhãn chuẩn để mô tả bản chất nội dung của dữ liệu truyền qua mạng. Ví dụ:

- `video/mp4`
- `video/webm`
- `video/quicktime`

MIME type không phải là tên file. Nó là metadata nói cho server hoặc browser biết dữ liệu nên được hiểu là loại gì.

Trong code, các MIME type hợp lệ được định nghĩa tại [`main.py:42`](/home/huy/vss_ai_workflow/main.py:42):

```python
ALLOWED_VIDEO_TYPES = {
    "video/mp4": ".mp4",
    "video/webm": ".webm",
    "video/quicktime": ".mov",
}
```

### 3.5 Phần mở rộng file hợp lệ là gì

Phần mở rộng file là suffix như:

- `.mp4`
- `.webm`
- `.mov`

Trong code, tập suffix hợp lệ là [`main.py:47`](/home/huy/vss_ai_workflow/main.py:47):

```python
ALLOWED_VIDEO_SUFFIXES = set(ALLOWED_VIDEO_TYPES.values())
```

Điểm quan trọng để phản biện:

- Hệ thống không chỉ tin vào phần mở rộng file.
- Hàm `sanitize_video_upload_name()` kiểm tra cả suffix lẫn MIME type.
- Nếu suffix sai nhưng MIME type hợp lệ, hệ thống vẫn có thể suy ra suffix từ MIME type.

Hàm này nằm ở [`main.py:182`](/home/huy/vss_ai_workflow/main.py:182).

### 3.6 HTTP status 400, 413, 404, 416, 502, 500 nghĩa là gì

Đây là mã trạng thái HTTP, do chuẩn HTTP định nghĩa.

- `400 Bad Request`: request sai về mặt cú pháp hoặc dữ liệu đầu vào không hợp lệ đối với business rule của API.
- `413 Content Too Large` (trước đây thường gọi `Payload Too Large`): nội dung gửi lên vượt quá giới hạn server chấp nhận.
- `404 Not Found`: tài nguyên được yêu cầu không tồn tại.
- `416 Range Not Satisfiable`: client yêu cầu một khoảng byte không hợp lệ đối với tài nguyên.
- `500 Internal Server Error`: lỗi nội bộ phía server.
- `502 Bad Gateway`: service hiện tại đóng vai trò trung gian, nhưng khi gọi service phía sau thì gặp lỗi hoặc không lấy được dữ liệu hợp lệ.

Trong hệ thống này:

- `400` dùng cho loại file không hợp lệ hoặc thiếu câu hỏi.
- `413` dùng cho video vượt 100 MB.
- `404` dùng khi object trên GCS không tồn tại hoặc clip local không tồn tại.
- `416` dùng cho Range header không hợp lệ.
- `500` dùng cho lỗi nội bộ như không có FFmpeg.
- `502` dùng khi VMS gọi GCS hoặc gọi AI service phía sau thất bại.

## 4. Sơ đồ 1: Upload request từ edge vào VMS

### 4.1 Kết luận

Sơ đồ 1 đúng về trục chính nhưng chưa chính xác hoàn toàn nếu trình bày là "chỉ cần kiểm tra MIME type và phần mở rộng, sai thì trả 400 hoặc 413".

Code thật cho thấy:

- `400` là lỗi loại file không hợp lệ.
- `413` là lỗi kích thước vượt ngưỡng 100 MB.
- Hai lỗi này không cùng phát sinh từ một decision duy nhất.

### 4.2 Luồng thực tế theo code

Luồng backend thật như sau:

1. Edge gửi `POST /api/videos` hoặc `POST /upload`.
2. FastAPI nhận file `video` và các field form.
3. Hàm `store_video()` được gọi tại [`main.py:874`](/home/huy/vss_ai_workflow/main.py:874).
4. `sanitize_video_upload_name()` kiểm tra tên file, suffix và MIME type tại [`main.py:495-497`](/home/huy/vss_ai_workflow/main.py:495).
5. Nếu loại file không hợp lệ, ném `HTTPException(400)`.
6. Nếu hợp lệ, file được ghi dần vào `uploads_raw/` theo từng chunk 1 MB tại [`main.py:503-509`](/home/huy/vss_ai_workflow/main.py:503).
7. Trong lúc ghi, nếu tổng kích thước vượt `MAX_VIDEO_SIZE = 100 * 1024 * 1024`, server trả `413` tại [`main.py:507-508`](/home/huy/vss_ai_workflow/main.py:507).
8. Sau khi lưu xong bản raw, server gọi `convert_video_to_mp4()` trong threadpool tại [`main.py:510`](/home/huy/vss_ai_workflow/main.py:510).
9. Nếu convert thành công, trả về thông tin file đã lưu trong `uploads/`.

### 4.3 Vì sao phải tách `uploads_raw/` và `uploads/`

Đây là điểm rất dễ bị hỏi.

Giải thích học thuật:

- `uploads_raw/` là vùng lưu dữ liệu gốc, tức là artifact trước chuẩn hóa.
- `uploads/` là vùng lưu dữ liệu đã được chuyển về định dạng phục vụ phát trên trình duyệt.
- Tách hai thư mục này giúp bảo toàn nguyên bản đầu vào, dễ debug pipeline, và tách biệt clearly giữa "ingest artifact" và "presentation artifact".

Nói cách khác:

- `uploads_raw/` phục vụ tính truy vết và xử lý hậu kỳ.
- `uploads/` phục vụ tính tương thích phát lại.

### 4.4 Chuẩn hóa tên file là gì và vì sao cần

Hàm `sanitize_video_upload_name()`:

- lấy `basename` thay vì tin toàn bộ path client gửi lên,
- loại bỏ ký tự không an toàn,
- giới hạn độ dài stem còn 60 ký tự,
- quyết định suffix raw hợp lệ.

Điều này có ý nghĩa:

- tránh path traversal logic đơn giản,
- tránh ký tự lạ gây lỗi filesystem hoặc URL,
- tạo tên file ổn định cho các bước sau.

### 4.5 Nếu trình bày sơ đồ 1 cho đúng với code thì nên sửa gì

Nên đổi sơ đồ 1 thành:

1. `(Bắt đầu)`
2. `<Edge gửi multipart/form-data>`
3. `[FastAPI nhận UploadFile và form fields]`
4. `[Kiểm tra MIME type/suffix và chuẩn hóa tên file]`
5. `{Loại file hợp lệ?}`
6. Nếu `Không` -> `[Trả 400 Bad Request]` -> `(Kết thúc)`
7. Nếu `Có` -> `[Ghi file vào uploads_raw/ theo từng chunk]`
8. `{Kích thước vượt 100 MB?}`
9. Nếu `Có` -> `[Xóa file tạm và trả 413 Content Too Large]` -> `(Kết thúc)`
10. Nếu `Không` -> `[Chuyển sang bước convert MP4]`

### 4.6 Câu hỏi phản biện có thể gặp

`Tại sao không chỉ kiểm tra phần mở rộng .mp4 là đủ?`

Trả lời:
Phần mở rộng chỉ là tên file, không phải bằng chứng mạnh về nội dung. Hệ thống kiểm tra thêm MIME type để giảm rủi ro nhận nhầm loại dữ liệu. Tuy nhiên, cần nói trung thực rằng code hiện tại vẫn chưa phân tích chữ ký nhị phân của file, nên đây là validation mức ứng dụng, chưa phải kiểm tra nội dung ở mức forensic.

`Vì sao trả 413 chứ không phải 400 khi file quá lớn?`

Trả lời:
Vì lỗi ở đây không phải cú pháp request sai, mà là request hợp lệ nhưng nội dung vượt ngưỡng chấp nhận của server. Theo semantics HTTP, đó là trường hợp của `413 Content Too Large`.

## 5. Sơ đồ 2: Chuẩn hóa video và đồng bộ cloud

### 5.1 Kết luận

Sơ đồ 2 đúng với logic xử lý thực tế, nhưng cần nói rõ rằng trong code hiện tại nó không chạy như một service độc lập. Nó nằm trong cùng request lifecycle của `POST /api/videos`, sau đó mới tùy chọn upload lên GCS và publish Pub/Sub.

### 5.2 Luồng thực tế theo code

1. Sau khi raw file được lưu, hàm `convert_video_to_mp4()` được gọi.
2. Hàm này dựng lệnh FFmpeg tại [`main.py:289-305`](/home/huy/vss_ai_workflow/main.py:289).
3. Video được mã hóa lại bằng:
   - codec hình `libx264`
   - `pix_fmt yuv420p`
   - codec âm thanh `aac`
   - `+faststart`
4. Nếu FFmpeg không có, trả `500`.
5. Nếu convert thất bại hoặc file kết quả rỗng, trả `400`.
6. Nếu thành công, file MP4 nằm trong `uploads/`.
7. Nếu `AUTO_INGEST_ON_UPLOAD=true`, code tiếp tục:
   - upload file MP4 lên GCS
   - dựng metadata
   - publish ingest event lên Pub/Sub

### 5.3 FFmpeg là gì và tại sao dùng nó

FFmpeg là bộ công cụ xử lý media phổ biến trong lĩnh vực multimedia engineering. Nó hỗ trợ:

- decode/encode video,
- chuyển container,
- đổi codec,
- tối ưu metadata để stream.

Trong hệ thống này, FFmpeg được dùng vì video đầu vào có thể là `mp4`, `webm`, `mov`, nhưng browser playback cần một cấu hình tương thích hơn.

### 5.4 Vì sao phải chuyển sang MP4 tương thích trình duyệt

Một file "đuôi mp4" chưa chắc đã tương thích tốt với mọi trình duyệt. Browser không chỉ quan tâm container mà còn quan tâm codec profile, pixel format, audio codec, và metadata placement.

Code đang chuẩn hóa bằng:

- `libx264`: codec H.264, mức tương thích trình duyệt cao.
- `yuv420p`: pixel format rất phổ biến, tương thích tốt hơn nhiều so với một số format màu khác.
- `aac`: audio codec phổ biến cho MP4.
- `+faststart`: dời metadata quan trọng của MP4 lên đầu file, giúp browser có thể phát sớm hơn khi stream qua HTTP.

Nếu bị hỏi sâu:

`+faststart` đặc biệt quan trọng vì nó tối ưu progressive playback, tức là client không cần tải toàn bộ file mới đọc được thông tin cần thiết để bắt đầu phát.

### 5.5 GCS là gì

GCS là Google Cloud Storage, một object storage service.

Phải giải thích đúng bản chất:

- GCS không phải filesystem truyền thống.
- Nó lưu object trong bucket.
- Mỗi video trên GCS được truy cập như một blob/object với tên object.

Trong code:

- bucket được lấy bởi `storage.Client().bucket(bucket_name)` tại [`main.py:197-202`](/home/huy/vss_ai_workflow/main.py:197)
- upload bằng `blob.upload_from_filename(...)` tại [`main.py:826-830`](/home/huy/vss_ai_workflow/main.py:826)

### 5.6 Nếu trình bày sơ đồ 2 cho đúng với code thì nên sửa gì

Nên ghi rõ đây là "hậu xử lý trong cùng flow upload", không phải batch job độc lập.

Nên sửa thành:

1. `(Bắt đầu)`
2. `[Đọc file gốc trong uploads_raw/]`
3. `[Gọi FFmpeg chuyển sang MP4 chuẩn browser]`
4. `{FFmpeg thành công?}`
5. Nếu `Không` -> `[Trả lỗi 400 hoặc 500 tùy nguyên nhân]`
6. Nếu `Có` -> `[Lưu MP4 vào uploads/]`
7. `{AUTO_INGEST_ON_UPLOAD có bật không?}`
8. Nếu `Không` -> `<Trả trạng thái stored>`
9. Nếu `Có` -> `[Upload MP4 lên GCS]` -> `[Chuẩn bị metadata ingest]` -> `[Publish Pub/Sub]`

### 5.7 Câu hỏi phản biện có thể gặp

`Tại sao convert xong mới upload GCS, không upload raw luôn?`

Trả lời:
Thiết kế hiện tại chọn MP4 chuẩn hóa làm artifact chính cho playback và ingest event. Như vậy object trên cloud là bản đã đồng nhất định dạng, giảm khác biệt giữa nguồn đầu vào và nguồn phục vụ downstream.

`Vì sao lỗi convert trả 400 chứ không phải 500?`

Trả lời:
Trong code hiện tại, nếu FFmpeg chạy nhưng không tạo được MP4 hợp lệ thì được xem như đầu vào không chuyển đổi được sang định dạng kỳ vọng, nên trả `400`. Nếu FFmpeg không tồn tại trên server thì đó là lỗi môi trường hệ thống, nên trả `500`.

## 6. Sơ đồ 3: Playback clip từ GCS lên trình duyệt

### 6.1 Kết luận

Sơ đồ 3 đúng ý tưởng tổng thể nhưng không khớp hoàn toàn với tên endpoint hiện có trong code.

Phải phân biệt hai việc:

- Lấy danh sách object video trên GCS
- Stream nội dung video của một object cụ thể

Trong code hiện tại:

- danh sách video lấy từ `GET /api/gcs/videos` tại [`main.py:566`](/home/huy/vss_ai_workflow/main.py:566)
- stream video lấy từ `GET /api/playback/clips?name=...` tại [`main.py:666`](/home/huy/vss_ai_workflow/main.py:666)

### 6.2 Luồng thực tế frontend và backend

Luồng thật là:

1. Frontend gọi `GET /api/gcs/videos` trong `loadRecordings()` tại [`front-end/vms_epic3.js:383`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js:383).
2. Backend list blob trong GCS, lọc theo suffix video, rồi trả về danh sách `objects`.
3. Frontend biến mỗi object name thành một recording model.
4. Khi người dùng chọn clip, frontend tạo URL stream bằng hàm `createGcsStreamUrl()` tại [`front-end/vms_epic3.js:78`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js:78).
5. URL này có dạng `/api/playback/clips?name=<object_name>`.
6. Browser nạp URL đó vào thẻ `<video>`.
7. Backend lấy blob tương ứng từ GCS và trả `StreamingResponse`.

### 6.3 Browser playback hoạt động như thế nào

Thẻ `<video>` của HTML không tự hiểu GCS. Nó chỉ hiểu một URL HTTP.

Do đó VMS đóng vai trò:

- lớp trung gian che giấu cấu trúc storage,
- lớp kiểm soát quyền truy cập và format response,
- lớp hỗ trợ HTTP Range để seek.

Nói học thuật:

VMS ở đây đóng vai trò `application-layer streaming gateway`, không chỉ là một static file host.

### 6.4 Object tồn tại hay không được kiểm tra ra sao

Khi request tới `/api/playback/clips?name=...`, backend gọi `get_gcs_video_blob(name)`:

- nếu tên object không hợp lệ hoặc suffix không hợp lệ -> `400`
- nếu object không tồn tại -> `404`
- nếu lỗi truy cập GCS -> `502`

Đây là logic ở [`main.py:209-227`](/home/huy/vss_ai_workflow/main.py:209).

### 6.5 Một điểm rất quan trọng cần phản biện trung thực

File hướng dẫn nói:

- frontend gọi API lấy danh sách clip
- VMS truy cập GCS để list objects
- frontend chọn clip
- frontend gán URL playback

Ý này đúng.

Nhưng nếu nói "frontend gọi `/api/playback/clips` để lấy danh sách clip" thì sai với code hiện tại.

Trong code:

- `/api/playback/clips` là endpoint stream dữ liệu video.
- `/api/gcs/videos` mới là endpoint list.

Nếu bị hỏi:

`Tại sao cùng tên /api/playback/clips mà vừa list vừa stream được?`

Trả lời đúng là:
Không. Trong code hiện tại, nó không làm cả hai. Tên endpoint `/api/playback/clips` đang được dùng cho stream theo query parameter `name`. Danh sách object được lấy qua endpoint khác là `/api/gcs/videos`.

### 6.6 Nếu trình bày sơ đồ 3 cho đúng với code thì nên sửa gì

Sơ đồ 3 nên đổi thành:

1. `(Bắt đầu)`
2. `[Frontend gọi /api/gcs/videos]`
3. `[VMS list blobs trên GCS]`
4. `<Trả danh sách object video>`
5. `[Người dùng chọn một clip]`
6. `[Frontend tạo URL /api/playback/clips?name=...]`
7. `[Browser gửi request phát video]`
8. `[VMS lấy blob từ GCS]`
9. `{Blob có tồn tại không?}`
10. Nếu `Không` -> `[Trả 404]`
11. Nếu `Có` -> `[Stream dữ liệu cho browser]`

## 7. Sơ đồ 4: Seek và tua video bằng HTTP Range

### 7.1 Kết luận

Đây là sơ đồ đúng nhất so với code. Nếu cần chọn một sơ đồ làm điểm nhấn kỹ thuật khi bảo vệ, đây là sơ đồ nên trình bày kỹ nhất.

### 7.2 HTTP Range là gì

HTTP Range là cơ chế để client chỉ yêu cầu một phần byte của tài nguyên thay vì toàn bộ file.

Ví dụ:

```http
Range: bytes=1000-1999
```

Điều này rất quan trọng cho video playback vì:

- browser không cần tải toàn bộ file trước khi phát,
- khi người dùng kéo seek bar, browser có thể yêu cầu đoạn byte gần vị trí mới,
- giảm băng thông thừa và giảm độ trễ cảm nhận.

### 7.3 Luồng thực tế theo code

Luồng ở backend:

1. Browser request `/api/playback/clips?name=...`.
2. Backend lấy `blob.size` để biết tổng kích thước object tại [`main.py:672`](/home/huy/vss_ai_workflow/main.py:672).
3. Backend đọc header `Range` từ `request.headers.get("range")` tại [`main.py:677`](/home/huy/vss_ai_workflow/main.py:677).
4. Hàm `parse_range_header()` phân tích range tại [`main.py:230-256`](/home/huy/vss_ai_workflow/main.py:230).
5. Nếu không có Range:
   - trả toàn bộ object từ byte `0` đến `total_size - 1`
   - status code là `200`
6. Nếu có Range hợp lệ:
   - xác định `start`, `end`
   - status code là `206 Partial Content`
7. Nếu Range không hợp lệ:
   - ném `416`
   - kèm header `Content-Range: bytes */total_size`
8. Dữ liệu được đọc theo chunk 1 MB bằng `iter_gcs_blob_range()` tại [`main.py:259-264`](/home/huy/vss_ai_workflow/main.py:259).
9. Response được bọc trong `StreamingResponse` tại [`main.py:694-699`](/home/huy/vss_ai_workflow/main.py:694).

### 7.4 Vì sao cần `206 Partial Content`

`206 Partial Content` là response chuẩn của HTTP khi server chỉ gửi một phần tài nguyên theo yêu cầu Range.

Nếu bị hỏi:

`Tại sao không phải lúc nào cũng trả 200?`

Trả lời:
Vì `200 OK` về mặt semantics biểu thị representation đầy đủ của tài nguyên. Khi server chỉ gửi một đoạn byte, response đúng chuẩn phải là `206 Partial Content`.

### 7.5 `416 Range Not Satisfiable` nghĩa là gì

Lỗi `416` xảy ra khi khoảng byte client yêu cầu không nằm trong tài nguyên thật.

Ví dụ:

- file chỉ có 5000 byte
- client đòi `bytes=9000-10000`

Code kiểm tra:

- `start < 0`
- `end < start`
- `start >= total_size`

Nếu vi phạm, backend trả `416` tại [`main.py:253-254`](/home/huy/vss_ai_workflow/main.py:253).

### 7.6 Vì sao phải đặt các header `Content-Range`, `Accept-Ranges`, `Content-Length`

Đây là phần rất dễ bị hỏi theo giao thức.

- `Accept-Ranges: bytes`
  - báo cho client biết server hỗ trợ yêu cầu theo khoảng byte.
- `Content-Range: bytes start-end/total`
  - mô tả chính xác phần dữ liệu đang được trả.
- `Content-Length`
  - cho biết số byte thực sự nằm trong response body lần này.

Nếu thiếu hoặc sai các header này, browser có thể seek không ổn định hoặc hiểu sai nội dung stream.

### 7.7 Vì sao đọc từng chunk 1 MB

`GCS_CHUNK_SIZE = 1024 * 1024` tại [`main.py:48`](/home/huy/vss_ai_workflow/main.py:48).

Ý nghĩa kỹ thuật:

- tránh tải toàn bộ object vào RAM một lần,
- giữ bộ nhớ server ổn định hơn,
- phù hợp với mô hình streaming generator.

Đây là một trade-off:

- chunk quá nhỏ -> nhiều request nội bộ tới storage, overhead tăng
- chunk quá lớn -> tốn bộ nhớ và tăng latency ở mỗi lần phát chunk

1 MB là một lựa chọn cân bằng đơn giản cho prototype/capstone.

### 7.8 Nếu trình bày sơ đồ 4 cho đúng với code thì nên sửa gì

Chỉ cần sửa một chi tiết nhỏ:

- khi không có Range thì code hiện tại trả `200`, không phải `206`

Do đó nên vẽ:

- nhánh `Không có Range` -> `[Đặt start=0, end=total_size-1]` -> `[Trả StreamingResponse 200 OK]`
- nhánh `Có Range hợp lệ` -> `[Trả StreamingResponse 206 Partial Content]`

## 8. Sơ đồ 5: Deep Analyze và clip-specific query

### 8.1 Kết luận

Sơ đồ 5 đúng về kiến trúc trung gian, nhưng phải nói rất rõ rằng VMS hiện tại chủ yếu là lớp orchestration/proxy, không tự thực hiện AI reasoning nội bộ trong `main.py`.

### 8.2 Luồng `Deep Analyze` thực tế

Frontend:

- người dùng bấm `Deep Analyze`
- frontend gọi `GET /api/playback/clips/{video_id}/analysis`

Điểm này nằm trong:

- nút mở modal ở [`front-end/vms_epic3.js:1133-1168`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js:1133)
- hàm `fetchAnalysis()` tại [`front-end/vms_epic3.js:566`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js:566)

Backend:

1. Workflow server nhận request tại [`main.py:702`](/home/huy/vss_ai_workflow/main.py:702).
2. Nó tạo `httpx.AsyncClient`.
3. Nó gọi sang AI graph service từ xa:
   - `GET {AI_GRAPH_URL}/api/playback/clips/{video_id}/analysis`
4. Nếu response `200`, server trả JSON đó về.
5. Nếu dữ liệu thiếu `title` hoặc `event`, workflow server thử bổ sung `title` từ clip local trong `uploads/`.
6. Nếu service từ xa lỗi HTTP hoặc không reachable, workflow server trả `502`.

### 8.3 `httpx` là gì và tại sao dùng

`httpx` là HTTP client library cho Python hỗ trợ cả sync và async.

Trong trường hợp này dùng `AsyncClient` vì route FastAPI là `async`, nên khi gọi service từ xa có thể dùng I/O bất đồng bộ thay vì block thread chính.

### 8.4 "Chuẩn hóa response" trong code hiện tại có mạnh không

Phải trả lời trung thực: không mạnh.

Code hiện tại chỉ chuẩn hóa rất nhẹ:

- nếu response đã thành công thì gần như pass-through dữ liệu từ service AI phía sau,
- chỉ cố gắng `setdefault("title", ...)` khi thiếu title,
- frontend tự diễn giải thêm `analysis_status`, `entities_list`, summary.

Vì vậy nếu sơ đồ ghi `[VMS chuẩn hóa response]`, thì câu đó đúng một phần, nhưng không nên mô tả như một tầng transformation phức tạp.

### 8.5 Luồng `clip-specific query` thực tế

Frontend:

- trong popup Deep Analyze, người dùng nhập câu hỏi
- frontend gửi `POST /api/playback/clips/{video_id}/query`

Phần này nằm tại [`front-end/vms_epic3.js:685-697`](/home/huy/vss_ai_workflow/front-end/vms_epic3.js:685).

Backend:

1. Nhận payload JSON.
2. Lấy `question`, làm sạch chuỗi bằng `clean_text()`.
3. Nếu câu hỏi rỗng -> trả `400`.
4. Workflow server cố tìm clip local trong `uploads/` bằng `get_playback_clip(video_id)` để suy ra:
   - `camera_id`
   - `date`
   - `location`
5. Nếu không tìm thấy clip local, các metadata này trở thành `None`.
6. Workflow server gọi AI graph service:
   - `POST {AI_GRAPH_URL}/api/v1/query`
7. Payload gửi đi gồm:
   - `query`
   - `video_id`
   - `camera_id`
   - `date`
   - `location`
   - `top_k = 3`
8. Response được rút ra:
   - `answer`
   - `sources`
   - `raw`

### 8.6 Một điểm phản biện rất quan trọng ở sơ đồ 5

File hướng dẫn nói:

- VMS bổ sung `video_id, camera_id, date, location`

Điều này đúng theo ý đồ code, nhưng cần nói rõ:

- `video_id` luôn có từ path parameter.
- `camera_id, date, location` chỉ bổ sung được nếu clip còn hiện diện trong thư mục local `uploads/`.
- Trong khi playback listing lại đang lấy từ GCS.

Nghĩa là kiến trúc hiện tại có một phụ thuộc ngầm:

- playback source là GCS,
- nhưng metadata enrichment cho query lại cố đọc local upload artifact.

Nếu local artifact không còn, query vẫn chạy nhưng mất bối cảnh metadata.

Đây là điểm nên chủ động nói ra khi phản biện, vì nếu không người chấm tinh ý có thể hỏi ngay:

`Nếu object có trên GCS nhưng local uploads/ đã bị dọn thì VMS còn enrich metadata được không?`

Câu trả lời đúng là:
Không chắc. Khi đó code fallback sang `None` cho `camera_id`, `date`, `location`.

### 8.7 Nếu trình bày sơ đồ 5 cho đúng với code thì nên sửa gì

Nên đổi câu:

- `[VMS chuẩn hóa response]`

thành:

- `[VMS proxy response và bổ sung title nếu thiếu]`

Và nên thêm một decision:

- `{Có tìm được clip local để enrich metadata không?}`

Nếu `Không`:

- `[Gửi query với video_id và question, metadata còn lại có thể là None]`

Như vậy sơ đồ sẽ trung thực hơn với hiện trạng code.

## 9. Sơ đồ 6: Publish ingest event qua Pub/Sub

### 9.1 Kết luận

Sơ đồ 6 đúng với pipeline publish event sau khi clip đã có trên GCS. Tuy nhiên, cần làm rõ ba điểm:

1. Bước này chỉ chạy khi `AUTO_INGEST_ON_UPLOAD` bật.
2. Code hiện tại publish đồng bộ trong cùng request upload, không phải background queue nội bộ.
3. Code này chỉ publish event, không chứa logic subscriber/consumer.

### 9.2 Pub/Sub là gì

Pub/Sub là mô hình giao tiếp bất đồng bộ kiểu `publish-subscribe`.

Các thành phần cơ bản:

- `publisher`: bên phát thông điệp
- `topic`: kênh logic chứa thông điệp
- `subscriber`: bên đăng ký nhận thông điệp

Ý nghĩa kiến trúc:

- tách producer khỏi consumer,
- giảm coupling thời gian thực,
- phù hợp cho pipeline ingest hậu kỳ.

### 9.3 Luồng thực tế theo code

Sau khi upload và convert xong:

1. `create_video()` kiểm tra `auto_ingest_enabled()` tại [`main.py:883`](/home/huy/vss_ai_workflow/main.py:883).
2. Nếu bật:
   - upload MP4 lên GCS bằng `upload_to_gcs()`
   - parse metadata từ tên file bằng `parse_upload_video_name()`
   - tạo dictionary metadata
   - publish event bằng `publish_ingest_event()`
3. `publish_ingest_event()`:
   - lấy `project_id` bằng `google.auth.default()`
   - tạo `PublisherClient`
   - dùng topic `vss-video-ingestion-dev`
   - nếu topic chưa có thì thử tạo
   - `json.dumps(payload).encode("utf-8")`
   - gọi `publisher.publish(...)`
   - chờ `future.result()` để lấy `message_id`
4. Nếu thành công, response trả `auto_ingest = "queued"`.
5. Nếu thất bại, response vẫn trả `status = "stored"` nhưng `auto_ingest = "failed: ..."`

### 9.4 Vì sao nói đây là bất đồng bộ nhưng code vẫn chờ publish xong

Đây là câu hỏi rất hay.

Giải thích chính xác:

- Mô hình tổng thể là bất đồng bộ vì ingest thật diễn ra ở subscriber/downstream worker sau khi nhận event.
- Nhưng thao tác publish trong request hiện tại vẫn là synchronous waiting đối với `future.result()`.

Nghĩa là:

- bất đồng bộ ở cấp hệ thống phân tán,
- nhưng chưa hoàn toàn bất đồng bộ ở cấp HTTP request lifecycle.

### 9.5 Metadata ingest gồm những gì

Code đang gửi các trường:

- `video_id`
- `camera_id`
- `date`
- `location`
- `start_time`
- `uploader_name`
- `client_id`
- `caption`
- `fps_sample`
- cộng thêm `gcs_uri` và `filename`

Điều này có ý nghĩa:

- downstream worker không cần truy hồi toàn bộ ngữ cảnh từ đầu,
- event message mang đủ metadata tối thiểu để khởi động ingest.

### 9.6 Nếu trình bày sơ đồ 6 cho đúng với code thì nên sửa gì

Nên thêm decision đầu tiên:

- `{AUTO_INGEST_ON_UPLOAD có bật không?}`

Nếu `Không`:

- `<Trả trạng thái stored, auto_ingest=disabled>`

Nếu `Có`:

- `[Upload MP4 lên GCS]`
- `[Tạo metadata ingest]`
- `[Publish message lên Pub/Sub topic]`
- `{Publish thành công?}`

Nên nói thêm:

- worker downstream không nằm trong repo này,
- sơ đồ subscriber là sơ đồ kiến trúc logic, không phải code có sẵn trong `main.py`.

## 10. Một số sai khác quan trọng giữa hướng dẫn, README và code thật

### 10.1 Endpoint `/api/playback/clips`

Trong `README.md`, endpoint này từng được mô tả như endpoint liệt kê clip local. Nhưng trong `main.py` hiện tại, endpoint này đang dùng để stream object từ GCS theo `name`.

Vì vậy:

- dùng `README` để mô tả hiện trạng là sai,
- phải dùng `main.py` làm nguồn chân lý.

### 10.2 Playback source là GCS, nhưng enrichment metadata lại dựa local uploads

Đây là một coupling chưa hoàn toàn sạch về kiến trúc.

Nó không làm code sai hoàn toàn, nhưng là một hạn chế thực tế của bản hiện tại.

### 10.3 Sơ đồ 1, 2 và 6 đang được tách logic để dễ vẽ, nhưng trong code chúng nằm trong cùng một request upload

Điểm này không sai về mặt báo cáo nếu nói rõ đây là `functional decomposition for explanation`.

Nếu không nói rõ, người phản biện có thể hỏi:

`Ba sơ đồ này là ba tiến trình độc lập hay chỉ là ba lát cắt của cùng một endpoint?`

Câu trả lời đúng là:
Trong code hiện tại, chúng chủ yếu là ba lát cắt của cùng endpoint `POST /api/videos`, chỉ khác nhau ở góc nhìn chức năng.

## 11. Phiên bản đề xuất của 6 sơ đồ nếu muốn bảo vệ chắc hơn

### 11.1 Sơ đồ 1

Giữ nguyên ý tưởng upload, nhưng tách riêng:

- `Loại file hợp lệ?`
- `Kích thước vượt 100 MB?`

### 11.2 Sơ đồ 2

Giữ nguyên ý tưởng convert + cloud sync, nhưng thêm decision:

- `AUTO_INGEST_ON_UPLOAD?`

### 11.3 Sơ đồ 3

Đổi bước đầu thành:

- `[Frontend gọi /api/gcs/videos]`

### 11.4 Sơ đồ 4

Giữ gần như nguyên vẹn, chỉ sửa:

- không có Range thì thường là `200 OK`
- có Range hợp lệ mới là `206 Partial Content`

### 11.5 Sơ đồ 5

Thêm decision:

- `Có metadata local để enrich query không?`

### 11.6 Sơ đồ 6

Thêm decision:

- `AUTO_INGEST_ON_UPLOAD có bật không?`

## 12. Mẫu trả lời phản biện ngắn gọn theo từng khái niệm

### 12.1 "FastAPI nhận request upload" thì FastAPI là gì

FastAPI là web framework ASGI cho Python. Nó định nghĩa API endpoint, validation và lifecycle request/response. Trong hệ thống này Uvicorn nhận kết nối mạng, sau đó chuyển request cho ứng dụng FastAPI trong `main.py`.

### 12.2 "MIME type và phần mở rộng hợp lệ" nghĩa là gì

MIME type là nhãn chuẩn mô tả loại nội dung truyền qua HTTP, ví dụ `video/mp4`. Phần mở rộng là hậu tố tên file như `.mp4`. Code hiện tại kiểm tra cả hai để giảm trường hợp file bị gắn sai đuôi hoặc metadata gửi lên không nhất quán.

### 12.3 "Trả lỗi 400 hoặc 413" thì hai lỗi đó khác nhau thế nào

`400` nghĩa là request không hợp lệ theo rule của API, ở đây là loại file không được chấp nhận. `413` nghĩa là request hợp lệ nhưng payload vượt ngưỡng kích thước server cho phép, ở đây là lớn hơn 100 MB.

### 12.4 "FFmpeg chuyển sang MP4 tương thích trình duyệt" nghĩa là gì

Không phải cứ đuôi `.mp4` là browser phát tốt. Code dùng FFmpeg để chuẩn hóa codec video sang H.264, audio sang AAC, pixel format sang `yuv420p`, và thêm `+faststart` để browser phát nhanh hơn khi stream.

### 12.5 "GCS object" là gì

GCS lưu dữ liệu dưới dạng object trong bucket, không phải file trong filesystem truyền thống. Mỗi video được truy cập như một blob có tên object, và backend dùng SDK của Google Cloud để thao tác với blob đó.

### 12.6 "HTTP Range" là gì

HTTP Range cho phép client chỉ xin một đoạn byte của file. Đây là nền tảng của seek video trên web, vì browser không cần tải toàn bộ file trước khi tua đến vị trí mong muốn.

### 12.7 "206 Partial Content" là gì

Đó là mã HTTP cho biết server chỉ trả về một phần của tài nguyên theo yêu cầu Range. Nó là response chuẩn cho playback có seek.

### 12.8 "416" đến từ đâu

`416 Range Not Satisfiable` là mã trạng thái chuẩn của HTTP. Nó xảy ra khi client yêu cầu dải byte nằm ngoài kích thước thật của file hoặc có cú pháp range sai.

### 12.9 "502" trong flow analysis/query nghĩa là gì

VMS hiện tại là tầng trung gian gọi sang service AI phía sau. Nếu tầng sau lỗi hoặc không truy cập được, VMS trả `502 Bad Gateway` để biểu thị lỗi không nằm ở client mà nằm ở hệ thống downstream.

## 13. Kết luận cuối cùng để dùng khi bảo vệ

Cách trình bày an toàn nhất là:

- nói rằng 6 sơ đồ trong file hướng dẫn đúng ở mức `functional intent`,
- nhưng khi đối chiếu với code thật thì có vài điểm phải hiệu chỉnh để phản ánh đúng endpoint, đúng semantics HTTP, và đúng mức độ tách rời giữa các bước,
- trong đó sơ đồ Range playback là sơ đồ bám code nhất,
- còn sơ đồ upload, cloud sync và Pub/Sub nên được giải thích như ba lát cắt của cùng pipeline `POST /api/videos`.

Nếu bị hỏi "nhóm có bịa flow không?", câu trả lời trung thực là:

Không nên trình bày nguyên văn file hướng dẫn như hiện trạng 1:1 của code. Cần hiệu chỉnh như tài liệu này đã chỉ ra. Sau khi hiệu chỉnh, các sơ đồ sẽ phản ánh đúng logic cốt lõi của hệ thống và có thể bảo vệ được về mặt kỹ thuật.

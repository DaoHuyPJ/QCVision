# QCVision – PRD

Nov 14, 2026 ·&#32;

QCVision nhận diện biển báo Việt Nam theo thời gian thực trên video từ camera gắn xe, giải thích ý nghĩa theo QCVN 41:2024/BGTVT có trích dẫn, cảnh báo xung đột giữa các biển, và chuyển ca không chắc chắn cho kỹ sư duyệt.&#32;

Mục tiêu là một hệ thống chạy trên video ( mô phỏng camera trên ô tô khi chạy ), nhận đúng biển, giải thích đúng luật và biết nói "không chắc" thay vì đoán.

1. Nhận diện biển báo theo thời gian thực trên luồng camera và video đường phố: 5 nhóm theo Điều 11 và khoảng 40–50 mã biển phổ biến.
2. Đọc điều kiện trên biển phụ (khung giờ, loại xe).
3. Giải thích ý nghĩa biển theo QCVN 41:2024 có trích dẫn điều khoản.
4. Cảnh báo khi các biển liên tiếp mâu thuẫn hoặc hết hiệu lực.
5. Chuyển các ca không chắc chắn cho kỹ sư duyệt (HITL).

**Không làm trong đợt này:**

- Triển khai trên phần cứng nhúng gắn trong xe (demo bằng laptop + dashcam)
- Phân loại toàn bộ mã biển trong quy chuẩn
- Sinh ảnh bằng diffusion, đối chiếu bản đồ (KITTI, OSM)
- Tự động điều khiển xe: hệ thống chỉ cảnh báo và giải thích

## Người dùng và user story

Tài xế dùng hệ thống khi đang lái; kỹ sư và chuyên gia luật dùng web app sau chuyến đi.

| Người dùng | User story |
| --- | --- |
| Tài xế | Khi xe chạy, tôi muốn thấy ngay biển vừa đi qua và ý nghĩa của nó, kể cả điều kiện trên biển phụ |
| Tài xế | Tôi muốn được cảnh báo khi hai biển trên đoạn đường mâu thuẫn nhau |
| Kỹ sư | Tôi muốn xem danh sách ca AI không chắc chắn kèm lý do, để duyệt hoặc sửa nhãn |
| Kỹ sư | Tôi muốn xem chỉ số của mô hình theo từng lớp và từng điều kiện (mờ, bị che, ngược sáng) để biết mô hình sai ở đâu |
| Chuyên gia luật | Tôi muốn kiểm tra phần giải thích của từng mã biển và trích dẫn đi kèm trước khi hệ thống dùng |

## Yêu cầu chức năng

Ưu tiên P0 phải xong trước hạn nộp hồ sơ P1 (06/12); ưu tiên P1 là phần Nâng cao cho Demo Day; ưu tiên P2 chỉ làm nếu còn thời gian.

| ID | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| F1 | Nhận luồng video tải lên; nhận diện ngay khi video chạy (không chờ xử lý xong cả video), hiển thị video có khung biển | P0 |
| F2 | Phát hiện biển và chia 5 nhóm; theo dõi mỗi biển qua nhiều frame | P0 |
| F3 | Xác định mã biển (40–50 mã); mã ngoài danh sách vào lớp "khác" | P0 |
| F4 | Đọc biển phụ: khung giờ, loại xe | P0 |
| F5 | Hiển thị giải thích có trích dẫn điều khoản, lấy từ bản tạo sẵn | P0 |
| F6 | Hàng đợi HITL: kỹ sư đồng ý, sửa nhãn hoặc đánh dấu không chuẩn | P0 |
| F7 | 2 vai trò đăng nhập: kỹ sư, chuyên gia luật | P0 |
| F8 | Báo cáo mAP và accuracy theo lớp | P0 |
| F9 | Rule engine: kiểm tra hình dạng/màu, xung đột và hết hiệu lực giữa các biển liên tiếp, biển tạm thời | P1 |
| F10 | Hỏi đáp tự do về quy chuẩn bằng RAG | P1 |
| F11 | Đánh giá RAG (faithfulness, citation accuracy); benchmark độ bền và độ trễ | P1 |
| F12 | Dashboard chẩn đoán lỗi theo loại và điều kiện | P2 |
| F13 | Trợ lý tra cứu cho kỹ sư (agent, chạy ngoại tuyến) | P2 |

## Yêu cầu phi chức năng

Hệ thống phải đủ nhanh để chạy trên xe và không bao giờ đưa ra giải thích luật không có căn cứ.

- **Thời gian thực:** xử lý kịp luồng video ( mô phỏng camera trên ô tô khi chạy ) trên laptop có GPU; FPS mục tiêu chốt sau khi đo baseline và hỏi mentor.
- **Không gọi LLM khi đang lái:** giải thích cho mỗi mã biển được tạo sẵn và chuyên gia luật duyệt; khi chạy chỉ tra cứu.
- **Trích dẫn bắt buộc:** mọi giải thích có điều khoản QCVN 41:2024; không có căn cứ thì trả lời "Quy chuẩn không quy định".
- **Không đoán:** ca độ tin cậy thấp, lớp "khác" hoặc vi phạm rule chỉ hiển thị mức nhóm biển và vào hàng đợi HITL.
- **Kết quả ổn định:** cùng một video cho cùng một kết quả; luồng chính là workflow cố định, không dùng agent.
- **Quyền riêng tư:** chỉ dùng dữ liệu công khai hoặc tự ghi.
- **Truy vết:** mọi quyết định duyệt của kỹ sư được lưu lại (ai, khi nào, sửa gì).

## Chỉ số thành công

4 chỉ số đầu do đề quy định; mục tiêu con số chốt sau baseline (dự kiến 22/11), đo trên tập test video cố định không dùng để train.

| Chỉ số | Module | Nguồn |
| --- | --- | --- |
| mAP phát hiện (mAP50, mAP50-95) | Detector | Đề |
| Accuracy theo lớp hiếm | Classifier | Đề |
| Độ chính xác trích xuất điều kiện biển phụ | OCR + parser | Đề |
| Faithfulness của giải thích | RAG | Đề |
| FPS và độ trễ mỗi frame | Toàn hệ thống | Yêu cầu thời gian thực |
| Citation accuracy | RAG | Phần Nâng cao |
| mAP theo mức mờ, che, ngược sáng | Detector | Phần Nâng cao |

## Rủi ro và câu hỏi mở

Rủi ro lớn nhất là dữ liệu biển báo Việt Nam còn ít và việc train mô hình dồn vào 2 người.

| Rủi ro | Cách xử lý |
| --- | --- |
| Dữ liệu VN ít, nhiều bộ không rõ giấy phép hoặc dùng mã biển 2019 | Tự quay dashcam, đổi mã sang bản 2024 bằng script, sinh ảnh tổng hợp cho lớp hiếm |
| Train mô hình dồn vào 2 bạn làm CV | Thu hẹp phạm vi CV, dùng Ultralytics thay vì tự viết mô hình |
| Không đạt tốc độ thời gian thực | Dùng YOLO bản nhỏ, xuất TensorRT; dự phòng demo bằng video phát đúng tốc độ thật |
| OCR đọc sai dấu tiếng Việt trên biển phụ | Test sớm, chuẩn bị phương án fine-tune hoặc VLM dự phòng |
| Không có chuyên gia luật thật | Thành viên nhóm đóng vai, ghi rõ trong hồ sơ là vai trò giả lập |

## 

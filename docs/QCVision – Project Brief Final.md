# QCVision – Project Brief

Nov 13, 2026 ·&#32;

QCVision (đề ADAS-08 SignGuard VN) là công cụ giúp tài xế nhận dạng biển báo giao thông Việt Nam trên video đường phố, giải thích ý nghĩa biển theo QCVN 41:2024/BGTVT có trích dẫn điều khoản, và cảnh báo khi các biển trên cùng đoạn đường mâu thuẫn nhau.

## Vấn đề

&#32;Thực trạng : Mô hình train trên dữ liệu nước ngoài nhận nhầm hoặc hiểu sai biển báo Việt Nam, và kỹ sư không biết mô hình sai ở đâu.

- Biển Việt Nam có nhiều loại ít gặp trong dữ liệu quốc tế: biển cấm theo giờ, biển phụ ghi điều kiện (giờ, loại xe), biển tạm thời tại công trường.
- Biển thường bị che, mờ, ngược sáng; có khi hai biển mâu thuẫn trên cùng đoạn đường.
- QCVN 41:2024/BGTVT thay thế bản 2019 từ 01/01/2025, nhiều bộ dữ liệu hiện có vẫn gán nhãn theo mã biển cũ.

## Giải pháp

AI không tự quyết một mình: luật cố định kiểm tra lại AI, và kỹ sư duyệt những ca AI không chắc chắn.

1. **Detector** (YOLO, so sánh với RT-DETR): tìm biển trong từng frame của video (tracking theo dõi mỗi biển qua nhiều frame và gộp kết quả), chia 5 nhóm(  theo Điều 11 QCVN 41:2024 (cấm, hiệu lệnh, nguy hiểm, chỉ dẫn, biển phụ).
2. **Classifier**: xác định mã biển cụ thể (P.123a, W.207…); mã ngoài danh sách vào lớp "khác" của nhóm.
3. **Đọc biển phụ**: OCR + parser ra khung giờ, loại xe, khoảng cách.
4. **Rule engine** (không dùng AI): kiểm tra hình dạng/màu khớp nhóm biển, nhãn ổn định qua các frame, phát hiện xung đột và hết hiệu lực giữa các biển.
5. **RAG trên QCVN 41:2024**: giải thích ý nghĩa biển, bắt buộc trích dẫn điều khoản; giải thích cho từng mã biển được tạo sẵn để hiển thị ngay khi chạy thời gian thực; không có căn cứ thì trả lời "Quy chuẩn không quy định".

Ca có độ tin cậy thấp, thuộc lớp "khác", vi phạm rule hoặc xung đột được đưa vào **hàng đợi HITL** cho kỹ sư duyệt. Các ca đã sửa được dùng để train lại mô hình.

## Chỉ số thành công

Nhóm đo bằng 4 chỉ số đề yêu cầu; mục tiêu con số cụ thể sẽ chốt sau khi có kết quả baseline (dự kiến 22/11).

| Chỉ số | Module | Mục tiêu |  |  |
| --- | --- | --- | --- | --- |
| mAP phát hiện | Detector | Chốt sau baseline |  |  |
| Accuracy theo lớp hiếm | Classifier | Chốt sau baseline |  |  |
| Độ chính xác trích xuất điều kiện biển phụ | OCR + parser | Chốt sau baseline |  |  |
| Faithfulness của giải thích | RAG | Chốt sau baseline |  |  |

**Ràng buộc bắt buộc:** kỹ sư duyệt ca xung đột và ca không chắc chắn trước khi vào dữ liệu đánh giá; giải thích phải trích dẫn điều khoản, không suy diễn ngoài quy chuẩn; chỉ dùng dữ liệu công khai hoặc tự ghi, đã làm mờ mặt người và biển số.

## Phạm vi

Nhóm làm chắc toàn bộ phần Cơ bản của đề và 3 mục Nâng cao; các phần tốn công nhưng ít giá trị demo để sau.

| Làm | Để sau |
| --- | --- |
| Nhận diện thời gian thực trên luồng camera (mô phỏng camera gắn trên xe) và video đường phố; web app, 2 vai trò (kỹ sư, chuyên gia luật) |  |
| Nhận dạng 5 nhóm + khoảng 40–50 mã biển phổ biến |  |
| Đọc biển phụ cơ bản (giờ, loại xe) | Phân loại toàn bộ mã biển trong quy chuẩn |
| Giải thích có trích dẫn, hàng đợi HITL | Triển khai trên phần cứng nhúng (edge device) gắn trong xe |
| Nâng cao: phát hiện xung đột trên video, đánh giá RAG, benchmark độ bền và độ trễ |  |

## Đội ngũ và các mốc

Nhóm 4 người; phân vai dưới đây là đề xuất, chốt trong buổi họp nhóm tuần 1.

| Thành viên | Vai trò đề xuất |
| --- | --- |
| Đào Quang Huy&#32; | Frontend, UI/UX, dữ liệu, prompt cho RAG, điều phối, hồ sơ |
| Đào Thị Ngọc Ánh | Classifier, hiệu chuẩn độ tin cậy, đánh giá mô hình |
| Nguyễn Minh Triết | Detector, OCR biển phụ, tracking video |
| Mai Lâm Tấn Đạt | Backend, RAG, rule engine |

**Các mốc:**

- 15/11: chốt đề tài (Brief, PRD, wireframe, repo)
- 29/11: đóng băng tính năng
- 06/12, 21:00: nộp P1 – hồ sơ sơ bộ
- 13/12, 21:00: nộp P2 – Demo Day

Repo: [github.com/DaoHuyPJ/QCVision](https://github.com/DaoHuyPJ/QCVision)

# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Xuân Trường |
| MSSV | 2A202602761 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/truongapep/K4-L3L4-Track2-Day21-NguyenXuanTruong-2A202602761-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ này có f1_score cao nhất trong 5 lần chạy (hai lần còn lại là 200/0.05/3 cho F1 0.7014 và 100/0.2/4 cho F1 0.7005). Lần có accuracy cao nhất là lần 1 (0.8780), không trùng với lần có F1 cao nhất, cho thấy accuracy không phản ánh được mô hình nào bắt được nhiều người thu nhập cao hơn. Accuracy của 5 lần chỉ dao động từ 0.846 đến 0.878, trong khi F1 dao động từ 0.605 đến 0.715. Lần 2 dùng ít cây và learning_rate nhỏ nên F1 chỉ 0.6051, dưới ngưỡng 0.65 và sẽ bị quality gate chặn. Chênh lệch 0.004 giữa lần 1 và lần 3 khá nhỏ, và holdout chỉ có 500 mẫu nên chưa chắc có ý nghĩa thống kê.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ 24,8% số mẫu thuộc lớp thu nhập trên 50K, nên tập dữ liệu mất cân bằng. Một mô hình luôn trả lời "thu nhập thấp" vẫn đạt accuracy 0.752 nhưng không phát hiện được trường hợp thu nhập cao nào, tức là vô dụng, và con số 0.752 dễ gây hiểu nhầm rằng mô hình tốt. F1 của lớp dương là trung bình điều hòa của precision và recall tính riêng cho lớp thu nhập cao, nên mô hình đoán bừa có F1 bằng 0, còn mô hình thật sự bắt được lớp hiếm mới có F1 cao. Vì vậy ngưỡng chất lượng 0.65 được đặt trên F1. Khi gọi f1_score không dùng average="weighted" hay average="macro", vì weighted bị lớp đa số kéo lên cao, còn macro trộn F1 của lớp đa số vào kết quả, cả hai đều làm mất ý nghĩa của ngưỡng dành cho lớp thu nhập cao.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow báo ImportError FallbackAsyncAdaptedQueuePool khi chạy train.py | pip cài SQLAlchemy bản quá mới, không khớp MLflow 2.13.0 | Ghim sqlalchemy==2.0.30 trong requirements.txt, CI cũng dùng file này |
| Job Train lỗi NoSuchBucket ở bước upload model | Giá trị secret ARTIFACT_BUCKET trên GitHub chưa đúng | Nhập lại đúng tên bucket rồi chạy lại job |
| Job Release lỗi ssh: no key found | Secret SERVER_SSH_KEY không chứa đủ nội dung private key | Dán lại toàn bộ file private key gồm dòng BEGIN và END rồi chạy lại |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** F1 tăng từ 0.7149 lên 0.7354 (khoảng 0.02) và accuracy tăng từ 0.8740 lên 0.8820 khi dữ liệu huấn luyện tăng gấp đôi lên 44.722 mẫu. Mức tăng này nhỏ và holdout chỉ có 500 mẫu, nên không thể kết luận thêm dữ liệu luôn làm mô hình tốt hơn, nhất là khi hai nửa dữ liệu cùng phân phối. Điều được kiểm chứng ở Bước 3 là quy trình tự động: commit file .dvc kích hoạt pipeline, bốn job chạy qua và VM phục vụ mô hình mới.
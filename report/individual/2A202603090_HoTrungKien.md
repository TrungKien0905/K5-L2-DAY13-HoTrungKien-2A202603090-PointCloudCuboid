# Báo cáo Day 13 — Hồ Trung Kiên, 2A202603090

<!-- Báo cáo công khai. Chỉ ghi tên file ảnh, mô tả chung và MSSV của bạn đổi kiểm. Lưu ảnh thật vào gói ZIP nộp LMS. -->

## 1. Kết luận trước

Task cá nhân `D13-2A202603090`.

- **Môi trường lab** — Windows, CVAT 2.81.0. **Trình duyệt** — …
- **10 frame theo thứ tự trong task** — `s01f15`, `s01f40`, `s02f07`, `s02f39`, `s03f09`, `s03f43`, `s04f25`, `s04f53`, `s05f15`, `s05f33`.
- **Bạn đổi kiểm** — ghi MSSV, kết quả ở mục 5.

Bảng A có **101 cuboid** gồm 78 Car, 6 Truck, 4 Bus, 9 Cyclist và 4 Pedestrian. Hai frame có ít nhãn nhất là `s03f43` và `s04f53`, mỗi frame **6 cuboid**. Frame `s03f43` có 4 Car và 2 Cyclist, còn `s04f53` có 6 Car.

Phiếu tự kiểm có **0/101 cuboid bị cờ**. Đáy của Car `s01f15` idx 2 và Cyclist `s02f07` idx 1 cách mặt đường ước tính tại chỗ không quá 0,06 m. Có **3 cuboid** ngoài phạm vi 50 m, lần lượt ở 50,80 m, 54,03 m và 58,63 m. Đây là các nhãn cần xem lại về phạm vi gán.

- **Góc nhìn** — Kéo chuột trái để xoay khung 3D, chuột phải để dời. Xoay đến góc nhìn thẳng từ trên xuống để quét cả cảnh.
- **Cuboid** — Chọn nhãn và **Shape**, nháy đúp để đặt hộp. Chỉnh tâm, dài, rộng và yaw ở **Top**, kiểm đáy và chiều cao ở **Side** và **Front**.
- **Hướng và ảnh camera** — Bật **Appearance → Cuboid orientation** để xem vạch đỏ hướng của hộp. Chạy `python scripts/lab.py preview` để xem hộp chiếu lên ảnh camera.

## 2. Đọc frame (Bước 2)

Hai frame đầu đọc kỹ. Trên ảnh nhìn từ trên xuống, đánh dấu phía trước (`−y`, đối chiếu cam01), `+x` (trái xe) và `+y` (sau xe). Trong bảng, ghi cạnh **thực tế trên ảnh** cho từng hướng.

| Frame và hai tên file ảnh | Cạnh ảnh theo −y, +x, +y | Mặt đường và độ thấp hơn LiDAR (m) | Vật xa nhất trong bảng A và hướng khuất camera nếu thấy | Loại vật có nhãn | Vật phía sau hoặc hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f15` · `t1-s01f15-3d.png` · `t1-s01f15-top.png` | … | Thấp hơn LiDAR khoảng 1,84 m theo phiếu tự kiểm. Độ dốc cần xem ảnh. | Car idx 7 ở 48,86 m. Hướng khuất camera cần đối chiếu ảnh. | 7 Car, 1 Bus | Car idx 2 ở phía sau bên trái, cách 3,75 m. | … |
| `s01f40` · `t1-s01f40-3d.png` · `t1-s01f40-top.png` | … | Thấp hơn LiDAR khoảng 1,84 m theo phiếu tự kiểm. Độ dốc cần xem ảnh. | Car idx 5 ở 50,80 m. Trong phạm vi 50 m, xa nhất là Car idx 6 ở 45,24 m. Hướng khuất camera cần đối chiếu ảnh. | 8 Car | Car idx 7 ở phía sau bên trái, cách 14,58 m. | … |

Độ cao mặt đường trong bảng là trung vị của 7 ước tính cục bộ mỗi frame, lấy `z_bottom − ground_gap_m` tại các hộp có giá trị `ground_gap_m`.

- **Chỉ camera** — ghi 3 quan sát từ hai frame, kèm frame và góc camera.
- **Chỉ LiDAR** — ghi 3 quan sát từ hai frame, kèm frame và vật hoặc vùng quan sát.

**Tám frame lướt**

| Thứ tự trong task | Frame | Số cuboid | Cạnh ảnh hướng về phía trước (−y) | Điều lạ (nếu có) |
|---|---|---|---|---|
| 3 | `s02f07` | 20 | … | … |
| 4 | `s02f39` | 10 | … | … |
| 5 | `s03f09` | 9 | … | … |
| 6 | `s03f43` | 6 | … | … |
| 7 | `s04f25` | 12 | … | … |
| 8 | `s04f53` | 6 | … | … |
| 9 | `s05f15` | 13 | … | … |
| 10 | `s05f33` | 9 | … | Car idx 8 và 9 có tâm ngoài 50 m theo bảng A. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Các dòng dưới đây là mẫu chọn từ bảng A. Cần xác nhận `frame` và `idx` có khớp với từng ảnh. Hai frame đầu không có nhãn Pedestrian hoặc Cyclist trong bảng A, nên mẫu thứ ba lấy từ `s02f07`.

| Vật và tên file ảnh | Frame, idx | Class | Yaw (°) và căn cứ xác định hướng | Đáy so với mặt đường | Kết quả kiểm Top, Side, Front và camera | Kích thước so với bảng 4.3 |
|---|---|---|---|---|---|---|
| Xe con gần · `t2-a-car.png` | `s01f15`, idx 2 | Car | 269,0°. Cần đối chiếu đầu xe trên camera. | Đáy −1,837 m, thấp hơn đường cục bộ khoảng 0,06 m. | … | Dài 5,327 m, rộng 1,829 m, cao 1,572 m. Chiều dài vượt 0,027 m so với mốc trên 5,30 m của nhóm Car dài nhất trong bảng. |
| Xe buýt · `t2-b-bus-truck.png` | `s01f15`, idx 5 | Bus | 281,3°. Cần đối chiếu đầu xe trên camera. | Đáy −2,174 m, thấp hơn đường cục bộ khoảng 0,23 m. | … | Dài 10,752 m, rộng 2,644 m, cao 3,546 m. So với xe buýt thường, dài trong khoảng. Rộng và cao vượt lần lượt 0,044 m và 0,046 m. |
| Xe hai bánh · `t2-c-ped-cyclist.png` | `s02f07`, idx 1 | Cyclist | 275,3°. Cần đối chiếu hướng xe trên camera. | Đáy −1,694 m, cao hơn đường cục bộ khoảng 0,05 m. | … | Dài 1,808 m, rộng 0,605 m, cao 1,602 m. Cả ba chiều nằm trong khoảng tham chiếu của xe đạp có người. |

- **Vòng hiệu chuẩn** — ghi điểm lệch của cuboid mình và quy tắc guideline đã được lớp thống nhất.
- **Vật bỏ qua** — ghi frame, phía và khoảng cách (m), mô tả và loại nghi vấn (`UNCERTAIN_CLASS`, `LOW_VISIBILITY` hoặc loại phù hợp). Nếu thực tế không có, ghi “không có”.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame, nguồn và tên file ảnh CVAT | Vật hoặc vùng thấy trên point cloud | Nguyên nhân (giả thuyết) | Quy tắc guideline áp dụng | Quyết định thực tế |
|---|---|---|---|---|---|
| Bóng che | … · `t4-1-bong-che.png` | … | … | Bật `occluded` khi một vật bị che một phần và đặt hộp bao phần có thể suy luận. | … |
| Thưa ở xa | … · `t4-2-thua-o-xa.png` | … | … | Vẽ khi quan sát được khoảng 15–20% thể tích hoặc có đủ căn cứ suy luận. Nếu thiếu căn cứ, bỏ qua và ghi `LOW_VISIBILITY`. | … |
| Phản xạ thấp hoặc ít điểm bất thường | … · `t4-3-phan-xa-thap.png` | … | … | Đối chiếu hình dạng và ảnh camera trước khi quyết định. Nếu cụm điểm quá ít và ảnh chưa đủ rõ, ghi `LOW_VISIBILITY`. | … |
| Điểm lạ | … · `t4-4-diem-la.png` | … | … | Bỏ qua điểm lẻ hoặc cụm điểm không thuộc vật thể. | … |

<!-- Ghi nguồn là task cá nhân, mã task chung hoặc MSSV bạn cho mượn. Chỉ rõ vị trí và dấu hiệu trong ảnh. Xét occluded khi có vật bị che, quy tắc khoảng 15–20% thể tích và LOW_VISIBILITY với vật thưa, dấu hiệu camera với phản xạ thấp và quy tắc bỏ qua điểm lạ. Tham khảo docs/PHENOMENA.md và docs/GUIDELINE.md. -->

## 5. Số từ cuboid của tôi (Bước 5)

Đối chiếu theo cặp `frame`–`idx` giữa [bảng A](../../submission/D13-2A202603090-bangA.csv) và [phiếu tự kiểm](../../submission/D13-2A202603090-tukiem.csv). Công thức khoảng cách ngang là `dist_m = √(x² + y²)`, khoảng cách 3D là `r = √(x² + y² + z²)`.

| Frame, idx, class | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r (m) | yaw_deg (°) | Chạm đường, lơ lửng hay chìm |
|---|---|---|---|---|---|---|---|
| `s01f15`, idx 2, Car | −1,051 | 1,572 | −1,837 | 3,75 | 3,898 | 269,0 | Gần chạm đường, thấp hơn đường cục bộ 0,06 m. |
| `s01f15`, idx 5, Bus | −0,401 | 3,546 | −2,174 | 29,14 | 29,144 | 281,3 | Thấp hơn mốc −1,70 m 0,474 m. Ước tính cục bộ cho thấy đáy chìm khoảng 0,23 m. |
| `s02f07`, idx 1, Cyclist | −0,893 | 1,602 | −1,694 | 10,40 | 10,443 | 275,3 | Gần chạm đường, cao hơn đường cục bộ 0,05 m. |

- **Gần nhất** — Car `s01f40` idx 1, `dist_m = 3,23 m`, `r = 3,382 m`.
- **Xa nhất** — Car `s05f33` idx 9, `dist_m = 58,63 m`, `r = 58,640 m`. Tâm hộp nằm ngoài phạm vi 50 m.
- **Kích thước xe con** — Car `s01f40` idx 1 dài 4,377 m, rộng 1,793 m và cao 1,445 m. Các số này nằm trong khoảng sedan hoặc hatchback ở bảng 4.3 (dài 3,8–5,0 m, rộng 1,70–1,90 m, cao 1,40–1,60 m).
- **Kích thước người đi bộ** — Pedestrian `s02f07` idx 13 dài 0,538 m, rộng 0,392 m và cao 1,727 m. So với khoảng người lớn (dài 0,4–0,9 m, rộng 0,5–0,9 m, cao 1,50–1,95 m), chiều rộng thấp hơn mốc dưới 0,108 m. Hộp chứa 56 điểm LiDAR. Một khả năng là cụm điểm chỉ thể hiện phần thân hẹp. Cần xem Top và camera để kiểm tra hộp có bỏ sót bề ngang.
- **Hướng** — Trong `s04f25`, Car idx 2 ở phía trước (`y = −16,709 m`) có yaw 268,8°, lệch 1,2° so với hướng cùng chiều 270°. Car idx 4 cũng ở phía trước (`y = −10,876 m`) nhưng có yaw 89,7°, lệch 0,3° so với hướng ngược chiều 90°. Cần đối chiếu đầu xe trên camera để xác nhận hướng thực tế.

**Mật độ theo khoảng cách**

| Frame, idx, class | dist_m (m) | points_in (điểm) |
|---|---|---|
| `s01f40`, idx 1, Car | 3,23 | 2.171 |
| `s01f40`, idx 2, Car | 18,88 | 585 |
| `s01f40`, idx 3, Car | 24,03 | 213 |
| `s01f40`, idx 4, Car | 32,29 | 132 |

- **Xu hướng** — Trong bốn Car cùng frame, xe ở 32,29 m xa gấp khoảng 10 lần xe ở 3,23 m và còn 6,1% số điểm (132 so với 2.171). Cả hai dãy đều giảm theo khoảng cách, với các mốc tương ứng trên slide là 1.968, 718, 214 và 72 điểm.
- **Tự kiểm** — Phiếu hiện tại có 0 cuboid bị cờ. Hai cuboid đã sửa sau `preview` hoặc `selfcheck` cần ghi theo nhật ký thao tác (frame, idx, dấu hiệu và thay đổi).
- **Đổi kiểm** — ghi MSSV bạn kiểm chéo, hai giá trị thực tế đã so với bảng A (frame, idx, cột, đơn vị), kết quả khớp hay lệch và chỉnh sửa nếu có.

<!-- Mốc tham chiếu từ slide cho xe con: khoảng 1.968 điểm ở 0–10 m, 718 điểm ở 10–20 m, 214 điểm ở 20–30 m, 72 điểm ở 30–40 m và 19 điểm ở 60–80 m. -->

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

**(a) Ý nghĩa đối với mô hình**

- Nhãn theo mét giúp mô hình học vị trí và kích thước vật thể trong hệ tọa độ LiDAR, kể cả khi cùng một vật chiếm số pixel khác nhau trên ảnh ở các cự ly khác nhau.
- Vật thưa điểm cần ngưỡng gán nhãn nhất quán để mô hình vẫn học được từ những vật xa còn nhận diện được và giảm nhãn sai khi chưa đủ căn cứ xác định vị trí hoặc kích thước.

**(b) Sử dụng AI**

- Công cụ — Codex.
- Câu hỏi chính — Báo cáo cần những minh chứng nào, hệ tọa độ VF được đọc thế nào và quy tắc gán nhãn hỗ trợ mô hình học ra sao?
- Cách dùng kết quả — Dùng khung sáu mục, danh sách frame, phần giải thích kỹ thuật và các số đo tính từ bảng A và phiếu tự kiểm.

**(c) Điều còn chưa chắc**

Ghi bằng lời trường hợp còn chưa chắc hoặc lỗi phát hiện sau hạn nếu có.

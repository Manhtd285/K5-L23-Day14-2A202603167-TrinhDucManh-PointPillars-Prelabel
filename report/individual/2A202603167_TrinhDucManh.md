# Báo cáo Day 14 — Trịnh Đức Mạnh — 2A202603167

- Mã học viên dùng với `--student`: `2A202603167`
- `seed_tag` trong `frames_A.txt`: `393590736f`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

- Tham số tôi nghĩ giảm công sửa nhiều nhất: `score_thr`, tăng từ 0.1 (mặc định) lên khoảng 0.4.
- Vì sao (nhìn ảnh BEV tập A: box thừa nhiều hay box sót nhiều?): Trên BEV tập A tôi thấy box thừa nhiều hơn box sót. Theo số liệu `pred_A.json`, 53/104 box có score < 0.4, trong khi chỉ 8 box có dưới 10 điểm LiDAR, nên lọc theo `score_thr` sẽ tác động nhiều hơn `min_points`. Các box score thấp, nhất là Cyclist (16 box ở s01f54 và 12 box ở s03f53), phần lớn là box thừa; lúc sửa tôi phải xóa nhiều hơn thêm. Vì vậy tôi dự đoán tăng `score_thr` lên khoảng 0.4 sẽ giảm công sửa.

## 2. Tập A: sửa tay và tune

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | 61 | 9 | 4 | 29.3 |
| `best_config.json` | 15 | 16 | 3 | 22.0 |

- Tham số đã đổi trong `best_config.json`: `score_thr` 0.1 → 0.4 (`min_points` giữ 0; với `score_thr=0.4`, `min_points` = 0/5/10 cho cùng chi phí 22.0).
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra):

```
s01f54: cấu hình chọn {'score_thr': '0.4', 'min_points': '0'} → 12.8 (tốt nhất có thể 11.2)
s03f53: cấu hình chọn {'score_thr': '0.3', 'min_points': '10'} → 8.1 (tốt nhất có thể 6.4)
s04f31: cấu hình chọn {'score_thr': '0.3', 'min_points': '10'} → 3.2 (tốt nhất có thể 2.6)
```
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: Cấu hình chưa ổn định: leave-one-out chỉ chọn lại `score_thr=0.4` ở 1/3 frame (s01f54), 2 frame còn lại (s03f53, s04f31) chọn `score_thr=0.3, min_points=10`. Ở s01f54, chi phí sau tune là 12.8, gần như không đổi so với gốc 12.7. Tôi vẫn chọn 0.4 vì đây là cấu hình có tổng chi phí thấp nhất trên 3 frame (22.0, so với 22.5 của `0.3, min_points=10`), nhưng với chỉ 3 frame thì chênh lệch này nhỏ và chưa đủ để kết luận. Tune `score_thr`/`min_points` cũng không cải thiện được nhiều vì model bỏ sót phần lớn xe Truck và Bus; tôi hầu như phải tự thêm box cho hai loại này.

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202603167/compare/s03f53.jpg)

| Frame | Chi phí gốc | `best_config.json` |
|---|---:|---:|
| s01f54 | 12.7 | 12.8 |
| s03f53 | 12.6 | 6.6 |
| s04f31 | 4.0 | 2.6 |
| **Tổng** | **29.3** | **22.0** |

- `scripts/pp_compare.py` báo giảm 25% công sửa trên tập A (3 frame, theo công thức chi phí của lab).
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: Dự đoán ở mục 1 là **đúng**. Tăng `score_thr` từ 0.1 lên 0.4 là tham số giúp giảm công sửa nhiều nhất, còn `min_points` không làm thay đổi kết quả (ở `score_thr=0.4`, `min_points` = 0, 5 hay 10 đều cho cùng chi phí 22.0). Cụ thể trên 3 frame tập A, số box thừa phải xóa giảm mạnh từ 61 xuống 15 (tiết kiệm `46 × 0.3 = 13.8` điểm) và số box cần sửa giảm từ 4 xuống 3, dù số box bị sót phải thêm tăng từ 9 lên 16 (tăng thêm `7 × 1.0 = 7.0` điểm); tổng chi phí vẫn giảm từ 29.3 xuống 22.0 (giảm 25%). Trên ảnh `s03f53.jpg`, các box màu đỏ (thừa) giảm rõ rệt từ 32 xuống 7, trong khi box xanh dương (bị sót) chỉ tăng nhẹ từ 2 lên 4 (và box vàng giảm từ 2 xuống 1), giúp chi phí frame này giảm từ 12.6 xuống 6.6.

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202603167/frames_B.txt --config d14_2A202603167/best_config.json --gt d14_2A202603167/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| 84 | 18 | 3 | 44.7 |

12 frame có nhãn: GT 136 box, dự đoán 202 box, khớp 118. Lớp trong `pred_B.json`: Car 172, Cyclist 28, Pedestrian 2. Lớp trong `ref_B.json`: Car 114, Cyclist 10, Truck 9, Bus 2, Pedestrian 1.

- Loại box tốn công sửa nhất ở tập B: Theo công thức chi phí, phần lớn chi phí đến từ **xóa box thừa** (84 box × 0.3 = 25.2 điểm, chủ yếu là Car giảm từ 172 → 114 và Cyclist giảm từ 28 → 10), tiếp theo là **thêm box bị sót** (18 box × 1.0 = 18.0 điểm; sửa 3 box chỉ tốn 1.5 điểm). Về mặt thao tác thực tế, các lớp **Truck** (9 box) và **Bus** (2 box) tốn công nhất vì model KITTI không có hai lớp này nên hầu như phải thêm tay hoặc đổi lớp.

## 5. Giới hạn

- **Tập tune quá nhỏ (chỉ 3 frame):** kết quả leave-one-out không ổn định (2/3 frame chọn cấu hình khác), nên cấu hình `score_thr=0.4` dễ bị overfit trên tập A; ngoài ra mức giảm 25% chỉ đo trên chính tập A, chưa có số đo cấu hình gốc trên tập B để đối chiếu trực tiếp.
- **Khác biệt miền dữ liệu của model KITTI:** PointPillars được huấn luyện trên KITTI (LiDAR 64 tia, chỉ nhìn phía trước, chỉ có 3 lớp Car/Pedestrian/Cyclist), nên trên dữ liệu VF model không nhận diện được **Truck/Bus** và hay bỏ sót **Pedestrian/Cyclist**, đồng thời vẫn sinh nhiều box thừa.
- **Công thức chi phí chưa phản ánh thời gian sửa thật:** chỉ số `0.3·xóa + 1.0·thêm + 0.5·sửa` chỉ là ước lượng tương đối để so sánh cấu hình, không tương đương trực tiếp với thời gian hay giờ công gán nhãn thực tế.

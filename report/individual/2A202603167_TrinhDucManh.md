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
| Gốc | | | | |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202603167/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202603167/frames_B.txt --config d14_2A202603167/best_config.json --gt d14_2A202603167/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- ...

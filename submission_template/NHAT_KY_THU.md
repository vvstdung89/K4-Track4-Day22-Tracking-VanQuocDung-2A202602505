# Nhật ký thử nghiệm

Ghi lại các lần chạy thử trước khi chọn cấu hình nộp. Detector, kích thước ảnh và Re-ID giữ nguyên theo luật lab.

Máy: GTX 970 bị lỗi `Xid 79` (GPU rơi khỏi bus) giữa buổi, nên mọi lần chạy từ CP3 trở đi dùng `--device cpu`. Kết quả tracking không phụ thuộc thiết bị, chỉ chậm hơn.

## CP2 — Baseline

Lệnh:

```bash
python scripts/run_tracking.py --source "$LAB_DATA/video_1/img1" --seq-name video_1 \
  --tracker bytetrack --conf 0.3 --iou 0.5 --out runs/thu_nhanh --save-video --max-frames 150
```

So với nhãn của 150 frame đầu:

| | Tracker | Nhãn |
|---|---|---|
| Người mỗi frame | 4,3 (3–6) | 25,7 |
| Số danh tính | 9 ID | 35 người |
| Chiều cao hộp (trung vị) | 280 px | 105 px |

Chấm trên nhãn cắt còn 150 frame: HOTA 31,1 · DetA 13,4 · AssA 72,6 · MOTA 15,6 · IDF1 27,2 · IDSW 0.

Quan sát trên video xem thử:

- Không thấy đổi ID. Hai người phụ nữ áo tím và áo đỏ giữ một ID suốt 150 frame.
- Bỏ sót là lỗi chính. Khoảng 20 người nhỏ ở hậu cảnh (gốc cây, dãy cửa hàng, ghế đá) không có hộp. 42% người trong nhãn cao dưới 100 px, hộp nhỏ nhất tracker giữ là 92 px.
- Không thấy hộp giả trên nền.
- Kết luận: baseline thuộc kiểu C trong notebook — giữ ID tốt nhưng phát hiện thiếu. Bước tiếp theo là hạ `--conf`.

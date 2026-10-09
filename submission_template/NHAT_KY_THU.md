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

## CP3 — Thử có hệ thống (150 frame mỗi lần)

Cách làm: mỗi video chạy cả 5 tracker ở `conf 0.3 / iou 0.5` (vòng A). Tracker có vẻ hơn được quét `conf` 0.15 / 0.5 và `iou` 0.4 / 0.7, mỗi lần đổi một số (vòng B). Tracker tốt nhất của nhóm còn lại (chuyển động hoặc Re-ID) được chạy thêm ở `conf 0.15` để đối chứng.

Không có nhãn cho `video_2`–`video_5`, nên ngoài việc xem frame mẫu, nhóm đếm **"ID sinh giữa ảnh"**: số ID mới xuất hiện sau frame 10 ở xa mép ảnh. Người thật thường đi vào từ mép, nên ID mới giữa ảnh thường là track bị đứt hoặc đổi ID. Kiểm tra trên `video_1` (có nhãn): thứ hạng chỉ số này (ByteTrack 4 < BoT-SORT 9 < OC-SORT/DeepOCSORT 16 < StrongSORT 18) khớp thứ hạng IDSW thật (0 < 3 < 6–7 < 9).

### video_1 — có nhãn (chấm trên 150 frame đầu)

| Tracker | conf | iou | HOTA | DetA | AssA | MOTA | IDF1 | IDSW | FP | FN |
|---|---|---|---|---|---|---|---|---|---|---|
| bytetrack | 0.3 | 0.5 | 31.1 | 13.4 | 72.6 | 15.6 | 27.2 | 0 | 6 | 3244 |
| ocsort | 0.3 | 0.5 | 29.8 | 16.0 | 56.2 | 16.8 | 30.2 | 7 | 75 | 3123 |
| botsort | 0.3 | 0.5 | 32.3 | 16.0 | 66.0 | 17.3 | 30.7 | 3 | 70 | 3112 |
| strongsort | 0.3 | 0.5 | 31.9 | 15.7 | 65.5 | 16.5 | 29.9 | 9 | 70 | 3138 |
| deepocsort | 0.3 | 0.5 | 29.8 | 16.0 | 56.4 | 16.8 | 30.3 | 6 | 75 | 3123 |
| botsort | 0.15 | 0.5 | 33.6 | 17.5 | 65.5 | 18.4 | 32.8 | 2 | 114 | 3026 |
| botsort | 0.5 | 0.5 | 29.2 | 12.2 | 70.0 | 13.7 | 24.5 | 0 | 9 | 3314 |
| botsort | 0.3 | 0.4 | 32.1 | 15.3 | 67.8 | 16.7 | 30.2 | 0 | 52 | 3157 |
| botsort | 0.3 | 0.7 | 31.2 | 16.9 | 58.6 | 18.5 | 31.8 | 5 | 76 | 3060 |
| botsort | 0.1 | 0.5 | **33.8** | 18.1 | 64.6 | **18.8** | **34.0** | 2 | 141 | 2985 |
| bytetrack | 0.15 | 0.5 | 31.5 | 13.8 | 72.5 | 16.0 | 27.9 | 0 | 10 | 3226 |

Xác nhận trên đủ 600 frame:

| Tracker | conf | iou | HOTA | DetA | AssA | MOTA | IDF1 | IDSW |
|---|---|---|---|---|---|---|---|---|
| botsort | 0.1 | 0.5 | **29.5** | 19.6 | 45.0 | **21.0** | **30.3** | 23 |
| botsort | 0.15 | 0.5 | 29.3 | 19.2 | 45.1 | 20.7 | 29.6 | 27 |
| bytetrack | 0.15 | 0.5 | 27.3 | 15.9 | 47.1 | 18.3 | 27.0 | 13 |

Nhận xét: lỗi chính là **bỏ sót** (FN ~3000 so với FP ~100 trên 150 frame). Hạ `conf` tăng DetA đều đặn mà FP chỉ tăng nhẹ. `iou 0.7` giữ thêm hộp ở chỗ đông nhưng AssA giảm (ID nhảy khi người chồng lên nhau). Chọn **botsort, conf 0.1, iou 0.5**.

### video_2 – video_5 — không nhãn

| Video | Tracker | conf | iou | Hộp/frame | Số ID | Độ dài track TB | Track < 10 frame | ID sinh giữa ảnh |
|---|---|---|---|---|---|---|---|---|
| video_2 | bytetrack | 0.3 | 0.5 | 8.75 | 16 | 82.1 | 3 | 5 |
| video_2 | ocsort | 0.3 | 0.5 | 10.60 | 25 | 63.6 | 5 | 9 |
| video_2 | botsort | 0.3 | 0.5 | 10.29 | 23 | 67.1 | 5 | 8 |
| video_2 | strongsort | 0.3 | 0.5 | 10.49 | 24 | 65.6 | 5 | 9 |
| video_2 | deepocsort | 0.3 | 0.5 | 10.61 | 25 | 63.6 | 5 | 9 |
| video_2 | botsort | 0.15 | 0.5 | **11.47** | 21 | **81.9** | **1** | 6 |
| video_2 | botsort | 0.5 | 0.5 | 7.50 | 14 | 80.4 | 4 | 3 |
| video_2 | botsort | 0.3 | 0.4 / 0.7 | 10.29 / 10.43 | 23 / 24 | 67.1 / 65.2 | 5 / 5 | 8 / 8 |
| video_2 | bytetrack | 0.15 | 0.5 | 9.18 | 15 | 91.8 | 3 | 3 |
| video_3 | bytetrack | 0.3 | 0.5 | 3.95 | 18 | 32.9 | 7 | 9 |
| video_3 | ocsort | 0.3 | 0.5 | 5.27 | 24 | 32.9 | 9 | 14 |
| video_3 | botsort | 0.3 | 0.5 | 5.10 | 23 | 33.3 | 9 | 10 |
| video_3 | strongsort | 0.3 | 0.5 | 5.17 | 18 | 43.1 | 6 | 8 |
| video_3 | deepocsort | 0.3 | 0.5 | 5.27 | 25 | 31.6 | 10 | 13 |
| video_3 | strongsort | 0.15 | 0.5 | 7.01 | 24 | 43.8 | 8 | 11 |
| video_3 | strongsort | 0.5 | 0.5 | 3.72 | 14 | 39.9 | 5 | 6 |
| video_3 | strongsort | 0.3 | 0.4 | 5.16 | 17 | 45.5 | 5 | 7 |
| video_3 | strongsort | 0.3 | 0.7 | 5.31 | 18 | 44.2 | 5 | 8 |
| video_3 | strongsort | 0.15 | 0.4 | **6.86** | 19 | **54.2** | **4** | 8 |
| video_3 | bytetrack | 0.15 | 0.5 | 3.98 | 19 | 31.4 | 8 | 10 |
| video_4 | bytetrack | 0.3 | 0.5 | 5.39 | 15 | 53.9 | 2 | 7 |
| video_4 | ocsort | 0.3 | 0.5 | 6.56 | 21 | 46.9 | 5 | 8 |
| video_4 | botsort | 0.3 | 0.5 | 6.39 | 18 | 53.2 | 5 | 6 |
| video_4 | strongsort | 0.3 | 0.5 | 6.47 | 18 | 53.9 | 5 | 5 |
| video_4 | deepocsort | 0.3 | 0.5 | 6.59 | 18 | 54.9 | 4 | 6 |
| video_4 | deepocsort | 0.15 | 0.5 | 7.85 | 34 | 34.6 | 15 | 17 |
| video_4 | deepocsort | 0.5 | 0.5 | 5.25 | 13 | 60.5 | 2 | 6 |
| video_4 | deepocsort | 0.3 | 0.4 / 0.7 | 6.59 / 6.60 | 18 / 19 | 54.9 / 52.1 | 4 / 4 | 6 / 7 |
| video_4 | bytetrack | 0.15 | 0.5 (0.4, 0.7 như nhau) | 5.63 | **13** | **65.0** | **2** | **5** |
| video_5 | bytetrack | 0.3 | 0.5 | 4.49 | 20 | 33.6 | 6 | 12 |
| video_5 | ocsort | 0.3 | 0.5 | **6.57** | 27 | **36.5** | 7 | **12** |
| video_5 | botsort | 0.3 | 0.5 | 6.05 | 27 | 33.6 | 8 | 14 |
| video_5 | strongsort | 0.3 | 0.5 | 6.33 | 30 | 31.7 | 10 | 15 |
| video_5 | deepocsort | 0.3 | 0.5 | 6.56 | 30 | 32.8 | 9 | 13 |
| video_5 | ocsort | 0.15 | 0.5 | 10.01 | 57 | 26.4 | 24 | 40 |
| video_5 | ocsort | 0.5 | 0.5 | 3.58 | 19 | 28.3 | 7 | 12 |
| video_5 | ocsort | 0.3 | 0.4 / 0.7 | 6.57 / 6.57 | 27 / 29 | 36.5 / 34.0 | 7 / 9 | 12 / 12 |
| video_5 | botsort | 0.15 | 0.5 | 6.63 | 27 | 36.8 | 6 | 15 |

Quan sát trên frame mẫu (1, 30, 50, 80, 100, 120, 150):

- **video_2:** Cả hai tracker giữ ID ổn với người đã phát hiện (BoT-SORT giữ ID 1, 2, 3, 4, 6 suốt frame 30–150), nhưng chỉ ~10 trên 40+ người có hộp; đám đông nhỏ phía trên gần như bị bỏ sót. Hạ `conf` xuống 0.15 thêm người mà không thấy hộp giả, và track dài hơn vì ít frame bị mất phát hiện.
- **video_3:** Ở `conf 0.15` các hộp thêm là người thật (người nhỏ phía xa, người lấp ló mép trái). StrongSORT giữ ID người đàn ông áo vest và người phụ nữ áo sọc khi họ đứng sát nhau. `iou 0.4` cắt bớt hộp trùng khi người đứng chồng lên nhau, ít ID mới hơn.
- **video_4:** DeepOCSORT ở `conf 0.15` vẽ hộp lên **bóng phản chiếu** trên lan can kính và sàn bóng (ID 22, 45 — không có người), gây 34 ID. ByteTrack cùng `conf` không bị, vì nó chỉ mở track mới từ hộp điểm cao; hộp điểm thấp chỉ dùng để nối track cũ. Áo đỏ (ID 2) và áo trắng (ID 5) giữ ID suốt frame 30–150. `iou` không ảnh hưởng ở cảnh thưa này.
- **video_5:** BoT-SORT đổi ID người bên trái (7 → 15) và đẩy số ID lên 50+ vào frame 150; Re-ID trên người nhỏ, ảnh rung không đáng tin. OC-SORT ổn hơn. `conf 0.15` làm OC-SORT vỡ (57 ID, 24 track ngắn) vì hộp giả nhấp nháy khi xe rung.

### Cấu hình chọn

| Video | Tracker | conf | iou |
|---|---|---|---|
| video_1 | botsort | 0.1 | 0.5 |
| video_2 | botsort | 0.15 | 0.5 |
| video_3 | strongsort | 0.15 | 0.4 |
| video_4 | bytetrack | 0.15 | 0.5 |
| video_5 | ocsort | 0.3 | 0.5 |

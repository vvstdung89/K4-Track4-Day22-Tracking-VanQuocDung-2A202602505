# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** ………………………… **Thành viên:** …………………………

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | | | | | |
| video_2 (phố đêm, tĩnh, rất đông) | | | | | |
| video_3 (camera di động, ảnh nhỏ) | | | | | |
| video_4 (trong nhà, camera di chuyển) | | | | | |
| video_5 (trên xe bus, giao lộ đông) | | | | | |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: nop_bai_video1-pedestrian    HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            29.523    19.561    44.952    20.335    75.327    47.931    82.084    82.598    30.152    36.707    76.442    28.059
COMBINED                           29.523    19.561    44.952    20.335    75.327    47.931    82.084    82.598    30.152    36.707    76.442    28.059
CLEAR: nop_bai_video1-pedestrian   MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag
video_1                            21.005    80.163    21.129    24.062    89.135    12.903    20.968    66.129    16.232    4471      14110     545       23        8         13        41        64
COMBINED                           21.005    80.163    21.129    24.062    89.135    12.903    20.968    66.129    16.232    4471      14110     545       23        8         13        41        64
Identity: nop_bai_video1-pedestrianIDF1      IDR       IDP       IDTP      IDFN      IDFP
video_1                            30.3      19.24     71.272    3575      15006     1441
COMBINED                           30.3      19.24     71.272    3575      15006     1441
```

Cấu hình nộp: `botsort`, `conf 0.1`, `iou 0.5`, chạy đủ 600 frame.

| HOTA | DetA | AssA | MOTA | IDF1 | IDSW | FP | FN |
|---|---|---|---|---|---|---|---|
| 29.5 | 19.6 | 45.0 | 21.0 | 30.3 | 23 | 545 | 14110 |

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- Tracker đã chọn giữ ID tốt hơn, hay ít hộp giả hơn, ở điểm nào bạn nhìn thấy?
- Cảnh đó (đứng yên / chuyển động, đông / thưa, sáng / tối, trong nhà / ngoài trời) khiến tracker này hợp hơn tracker kia như thế nào?

## 4. Nếu có thêm thời gian

Một hoặc hai câu: bạn sẽ thử tiếp điều gì (Re-ID khác, quét `conf` mịn hơn, xem frame gây lỗi…).

# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** làm cá nhân (không có bạn cùng nhóm) **Thành viên:** VAN QUOC DUNG (2A202602505)

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

File nộp: `runs/nop_bai/video_1.txt` … `video_5.txt` (đủ frame). Chi tiết các lần thử: [`NHAT_KY_THU.md`](NHAT_KY_THU.md).

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.1 | 0.5 | Lỗi chính là bỏ sót người nhỏ ở hậu cảnh; người lớn ở tiền cảnh giữ ID suốt (áo tím, áo đỏ). Hạ conf thêm người mà hộp giả tăng ít (FP 141 / FN 2985 trên 150 frame đầu; đủ 600 frame: HOTA 29.5, MOTA 21.0, IDF1 30.3). | bytetrack conf 0.3: giữ ID tốt nhất (0 IDSW) nhưng bỏ sót nhiều nhất, HOTA 31.1 < 33.8. botsort iou 0.7: thêm hộp ở chỗ đông nhưng AssA giảm 66 → 59. |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.15 | 0.5 | Người đã phát hiện giữ ID ổn (ID 1, 2, 3, 4, 6 suốt frame 30–150), nhưng chỉ ~11 trên 40+ người có hộp; đám đông nhỏ phía trên gần như mất. conf 0.15 thêm người, không thấy hộp giả, track dài hơn (67 → 82 frame). | bytetrack 0.15: ít ID sinh giữa ảnh hơn (3 vs 6) nhưng ít hộp hơn 20% (9.2 vs 11.5 hộp/frame) — cảnh đông thì bỏ sót mới là lỗi lớn. botsort conf 0.5: chỉ 7.5 hộp/frame. |
| video_3 (camera di động, ảnh nhỏ) | strongsort | 0.15 | 0.4 | Người đứng sát camera, che nhau. StrongSORT giữ ID người áo vest và người áo sọc khi họ chồng lên nhau. conf 0.15 thêm người thật ở xa và người lấp ló mép trái; iou 0.4 bớt hộp trùng. Trong 150 frame thử: track dài nhất (54 frame), ít track vụn nhất (4). Nhưng trên đủ 837 frame, sau frame 150 camera di chuyển và track bị vụn: 212 ID, trung vị chỉ 8 frame — đây là điểm yếu của bài nộp. | deepocsort / ocsort conf 0.3: 24–25 ID, 13–14 ID sinh giữa ảnh. strongsort 0.15 / iou 0.5: thêm hộp nhưng 24 ID, 11 ID sinh giữa ảnh. |
| video_4 (trong nhà, camera di chuyển) | bytetrack | 0.15 | 0.5 | Áo đỏ (ID 2), áo trắng (ID 5) giữ ID suốt frame 30–150. Không có hộp trên bóng phản chiếu ở lan can kính và sàn. iou không ảnh hưởng (người ít chồng nhau). | deepocsort conf 0.15: vẽ hộp lên bóng phản chiếu (ID 22, 45 không có người), 34 ID, 15 track dưới 10 frame. |
| video_5 (trên xe bus, giao lộ đông) | ocsort | 0.3 | 0.5 | Người nhỏ, ảnh rung. Ở conf 0.3, OC-SORT có số hộp ngang các tracker Re-ID (6.6 hộp/frame) nhưng track dài nhất (36.5 frame) và ít ID sinh giữa ảnh nhất (12, ngang ByteTrack vốn ít hộp hơn hẳn). | botsort conf 0.3: đổi ID người bên trái (7 → 15), số ID lên 50+ ở frame 150. ocsort conf 0.15: vỡ thành 57 ID, 24 track dưới 10 frame. |

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

**video_1 — quảng trường, camera tĩnh, ban ngày (có số).** Với ảnh 1920 px thu về 640 px, phần lớn người ở hậu cảnh chỉ còn vài chục pixel, nên lỗi lớn nhất là bỏ sót chứ không phải đổi ID: ở conf 0.3, FN gấp ~45 lần FP. Vì vậy hạ conf (0.3 → 0.1) là thay đổi có lợi nhất: DetA tăng, HOTA và IDF1 tăng, trong khi hộp giả chỉ tăng nhẹ. BoT-SORT hơn ByteTrack ở đây vì khi giữ thêm hộp điểm thấp, Re-ID và bù chuyển động camera giúp nối lại người bị che ngắn; ByteTrack giữ ID sạch nhất (AssA cao nhất) nhưng ít người hơn nên HOTA thấp hơn. Camera tĩnh và ánh sáng tốt làm Re-ID đáng tin.

**video_2 — phố đêm, camera tĩnh trên cao, rất đông (chỉ bằng mắt).** Camera đứng yên và đèn đường đủ sáng, nên người đã có hộp giữ ID ổn với mọi tracker (BoT-SORT giữ ID 1, 2, 3, 4, 6 suốt frame 30–150). Lỗi lớn là bỏ sót: nhìn từ trên cao, người ở xa chỉ cao vài chục pixel, nên chỉ ~11 trên 40+ người có hộp. Vì vậy chọn theo hướng giữ nhiều người nhất mà không sinh ID rác: BoT-SORT ở conf 0.15 cho nhiều hộp nhất (11.5 hộp/frame) và track dài hơn so với conf 0.3 (82 so với 67 frame) vì ít frame bị mất phát hiện. ByteTrack ít ID mới hơn nhưng ít hộp hơn 20%; ở cảnh rất đông, bỏ sót người là lỗi lớn hơn.

**video_4 — trong nhà, camera tiến tới, kính phản chiếu (chỉ bằng mắt).** Sàn bóng và lan can kính tạo ra "người" phản chiếu với điểm tin cậy thấp. Khi hạ conf xuống 0.15, DeepOCSORT mở track mới cho các bóng này (ID 22, 45 trên lan can, không có người), số ID nhảy từ 18 lên 34 và có 15 track dưới 10 frame. ByteTrack cùng conf 0.15 không bị, vì nó chỉ mở track mới từ hộp điểm cao; hộp điểm thấp chỉ dùng để nối track sẵn có. Nhờ vậy ByteTrack vừa lấy thêm được hộp điểm thấp cho người thật đang bị che, vừa không sinh ID từ bóng phản chiếu: 13 ID, track trung bình 65 frame, áo đỏ và áo trắng giữ ID suốt.

**video_5 — quay từ xe bus, giao lộ đông, rung (chỉ bằng mắt).** Người trong cảnh nhỏ và ảnh rung, nên vết ngoại hình (Re-ID) trên vài chục pixel bị nhòe và dễ khớp nhầm: BoT-SORT đổi ID người bên trái (7 → 15) và số ID lên 50+ trong 150 frame. OC-SORT dựa vào chuyển động, có bước khôi phục track theo quan sát cuối nên chịu được rung tốt hơn: cùng số hộp nhưng ít ID sinh giữa ảnh hơn và nhanh gấp đôi. Ở đây không nên hạ conf: conf 0.15 làm hộp điểm thấp chập chờn theo nhịp rung, OC-SORT vỡ thành 57 ID.

## 4. Nếu có thêm thời gian

Quét conf mịn hơn quanh 0.1–0.2 cho từng video không nhãn và xem đủ video có vẽ ID thay vì frame mẫu. Với video_2 (rất đông, người nhỏ), thử thêm Re-ID khác hoặc xem các frame đám đông phía trên để biết bỏ sót do detector hay do tracker chưa xác nhận track.

# Báo cáo thực hành PointPillars — Day 13

Đây là mẫu báo cáo cá nhân dùng cho kiểm tra formative. Điền bằng kết quả thật của bạn; không đoán số liệu hoặc nhận đã chạy nếu chỉ đọc kết quả có sẵn. Chỉ đưa lên repo cá nhân nội dung được phép; dữ liệu Robotaxi và output hạn chế lưu tại nơi LC chỉ định.

## Người thực hiện và provenance

- Họ tên/MSSV: xem `TEAMMATES.md`.
- Trạng thái cá nhân: `executed-on-personal-machine` / `executed-on-room-LC-machine` / `provided-results`.
- Người thực hiện: [họ tên]; ngày/giờ; hệ máy/architecture:
- Image tag và image ID; phiên bản repo:
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp:
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp:
- Phạm vi: front-window; score threshold:
- Giả định kênh thứ tư/intensity và nguồn z_ground:

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Báo cáo tên file và đường dẫn output được phép lưu; không cần đính kèm dữ liệu hạn chế vào repo.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | | | | |
| B | 1.73 | 0.16 | | | | |
| C | 1.73 | 0.32 | | | | |

- A/B — chỉ đổi delta: A có [số thật] hộp; B có [số thật] hộp. File/vùng … khác ở … . Đây là chạy lại model trên input khác, không chỉ dịch hộp cũ; điều tôi còn chưa chắc là … .
- B/C — chỉ đổi pillar: B có [số thật] hộp; C có [số thật] hộp. File/vùng … khác ở … . Số lượng/lớp/vị trí thay đổi như sau: … . Có đủ bằng chứng để kết luận tốt hơn không? … .
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | | | | | |
| case-batch-z | | | | | |
| case-one-box-z | | | | | |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng.

## Nhận xét và quyết định cá nhân

Phần việc đã tự thực hiện: … . Một quan sát A/B/C có dẫn file hoặc hộp/vùng: … . Diễn giải phép z thuận/ngược: … . Quyết định lỗi batch và hành động: … . Điều chưa chắc: … . Nếu chỉ đọc kết quả chuẩn bị trước, ghi rõ chưa tự chạy.

## Nộp bài

- URL repo cá nhân đã nộp trên VLearn: [điền]
- Output KITTI được lưu tại nơi LC cho phép: [ghi vị trí/tên file; không commit output hạn chế]

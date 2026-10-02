# Báo cáo cá nhân — PointPillars và QC pipeline — Day 13

Điền bằng chứng thực tế của chính bạn. Không đoán số liệu hoặc ghi đã tự chạy nếu chỉ xem kết quả có sẵn. Không đưa PCD/ảnh/annotation Robotaxi hay output hạn chế vào repo; lưu output KITTI tại nơi LC cho phép để đối chiếu và chỉ ghi tên file, đường dẫn hợp lệ cùng số liệu trong báo cáo.

## 1. Người thực hiện và provenance

- Họ tên/MSSV: Nguyễn Đức Văn — 2A202602115
- Trạng thái thực hiện: `executed-on-personal-machine` (runner hoàn tất; `smoke.json` = `passed`)
- Ngày, giờ thực hiện: 2026-10-02, 11:06–11:15 UTC (theo `smoke.json`)
- Máy và architecture: host Windows; Docker Linux `amd64` / `x86_64`
- Image tag, image ID và phiên bản repo: `day13-pointpillars:lc-20261001-amd64`; `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`; revision `0831856d921609312d42c7582c366e5a311bb7b1` (working tree dirty)
- PCD/frame được cấp, nơi được phép chạy, fingerprint nếu LC cấp: KITTI Student `demo.pcd`, frame `demo`; SHA-256 `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`; dữ liệu CC BY-NC-SA 3.0, chạy từ gói Student đã cấp
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`; SHA-256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`
- Phạm vi/ROI và score threshold: front ROI; score threshold `0.3`
- Giả định kênh thứ tư/intensity và nguồn `z_ground`: reflectance gốc đã bị loại; RGB=0 là placeholder và adapter dùng kênh hằng; `z_ground=0.075 m` theo output
- Thư mục output KITTI hiện lưu ngoài repo: `D:\AI\Day-13\ket-qua-ca-nhan`; xác nhận với LC nếu cần chuyển sang vị trí đối chiếu được chỉ định

## 2. Ba lượt inference A/B/C

Cả ba lượt dùng cùng PCD/checkpoint/score/ROI và runner. Lấy `n_boxes` và `mean_z` từ `run-A/B/C/summary.csv`; không tự tính hoặc đoán. `mean_z` không phải điểm chất lượng. Đối chiếu `side-*.png` với `boxes-*.json`. Chỉ ghi tên file và đường dẫn output được phép; không cần đưa output hạn chế vào repo.

| Lượt | Delta | Pillar XY | Số hộp (`n_boxes`) | `mean_z` | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| A | 0 m | 0.16 m | 1 | 0.330 m | `run-A/boxes-demo-delta-0-voxel-0.16.json`; `side-demo-delta-0-voxel-0.16.png`; `summary.csv` | 1 `vehicles`; Side có một hộp quanh x≈13.2 m |
| B | 1.73 m | 0.16 m | 13 | 1.034 m | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`; `side-demo-delta-1.73-voxel-0.16.png`; `summary.csv` | 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian`; hộp trải từ x≈3.7 đến 55.6 m |
| C | 1.73 m | 0.32 m | 6 | 1.091 m | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`; `side-demo-delta-1.73-voxel-0.32.png`; `summary.csv` | 6 `pedestrian`; Side cho thấy bố trí/hình dạng hộp khác B |

- A/B, chỉ đổi delta: A có 1 hộp và B có 13; `mean_z` đổi từ 0.330 m lên 1.034 m. Side B có nhiều hộp hơn và mở rộng theo x so với hộp đơn của A. Đây là inference mới trên input đã dịch, không phải dịch hộp A đi 1.73 m; số lượng và vị trí đều đổi. Chênh lệch số hộp không chứng minh B chính xác hơn.
- B/C, chỉ đổi pillar: B có 13 hộp (10 xe, 1 hai bánh, 2 người đi bộ), C có 6 hộp đều là `pedestrian`; `mean_z` lần lượt là 1.034 m và 1.091 m. Side và JSON cho thấy thay đổi lớn về số lượng/class/vị trí; dữ liệu này chưa đủ để kết luận cấu hình nào tốt hơn.
- Giới hạn ROI và góc nhìn Side: front ROI không đánh giá vật ngoài vùng trước. Side chiếu x-z và chồng các đối tượng khác y, nên khó xác nhận hộp trùng nhau, miss hoặc yaw chỉ từ ảnh này; cần view khác/reference, hiện PCD demo không có camera kèm để đối chiếu.
- JSON nào chưa đủ cơ sở để import và cần kiểm gì tiếp? Không JSON A/B/C nào được import vào Robotaxi vì đây là frame KITTI demo, không phải frame/job Robotaxi. Chỉ dùng chúng để so cấu hình; muốn đánh giá độ đúng cần kiểm hình học ở nhiều view và reference được duyệt.

## 3. Ca QC có kiểm soát, không import CVAT

Các ca được tạo có chủ đích từ prediction lượt B để luyện phân biệt lỗi pipeline với lỗi từng hộp; không phải output inference mới hoặc nhãn đúng.

| Ca | Số hộp lệch z/tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Quyết định và hành động | Bằng chứng/file |
| --- | --- | --- | --- | --- | --- |
| `case-correct` | 0/13 | 0 m | Không đổi | Bản sao chuyển đổi của B; không phải ground truth | `qc-cases/case-correct.json`; `side-correct.png` |
| `case-batch-z` | 13/13 | -1.805 m mỗi hộp (`delta + z_ground`) | Không đổi | Dừng batch, kiểm lại phép inverse/frame; không sửa tay từng hộp | `qc-cases/case-batch-z.json`; `side-batch-z.png` |
| `case-one-box-z` | 1/13 | -1.805 m ở hộp đầu | Không đổi | Kiểm hộp đó bằng nhiều view; không kết luận lỗi toàn pipeline | `qc-cases/case-one-box-z.json`; `side-one-box-z.png` |

## 4. Nhận xét cá nhân

- Phần việc tự thực hiện: chạy runner A/B/C và helper QC trên bundle Student amd64; kiểm CSV, JSON, ảnh Side và `smoke.json`.
- Một quan sát A/B/C, kèm file hoặc hộp/vùng làm bằng chứng: A chỉ có một xe tại x≈13.2 m; B có 13 hộp, trong đó có hộp ở vùng xa x≈55.6 m (`run-A/side-demo-delta-0-voxel-0.16.png`, `run-B/side-demo-delta-1.73-voxel-0.16.png`). Đây là khác biệt detector, không xác nhận các hộp đều đúng.
- Giải thích phép z thuận/ngược: `z_model = z_source - z_ground - delta`; khi chuyển ngược phải cộng lại `z_ground + delta`. Với `z_ground=0.075 m`, offset ở B là 1.805 m. Inference A/B chạy lại model nên không kỳ vọng mọi hộp chỉ tịnh tiến đúng 1.73 m.
- Quyết định lỗi batch/từng hộp và hành động: lệch cùng `-1.805 m` ở 13/13 hộp thì dừng và kiểm transform; chỉ lệch 1/13 thì kiểm đối tượng đó với nhiều view. Các ca là biến đổi QC có chủ đích, không phải lỗi thật của inference B.
- Điều còn chưa chắc hoặc cần LC hỗ trợ: chỉ có một PCD KITTI/front ROI, không có camera/reference trong gói; chưa thể kết luận độ chính xác hoặc chuyển kết quả sang Robotaxi.
- Nếu chỉ phân tích `provided-results`, ghi rõ chưa trực tiếp chạy model: không áp dụng; `smoke.json` xác nhận các bước `run-A`, `run-B`, `run-C`, `qc-cases` đều `passed`.

## 5. Tự kiểm CVAT/portal cá nhân

Annotation Robotaxi được thao tác trong hệ thống được cấp. Không ghi dữ liệu nhận dạng frame hoặc nội dung annotation nhạy cảm vào repo này.

- Job nguồn đã Save trong CVAT trước khi nộp: [chưa xác minh trong portal; cập nhật trạng thái thực tế]
- Snapshot v1 đã nộp trên portal: [chưa xác minh trong portal; cập nhật trạng thái thực tế]
- Feedback QC đã nộp: [chưa xác minh trong portal; cập nhật trạng thái thực tế]
- Đã Save và nộp v2/phản hồi: [chưa xác minh trong portal; cập nhật trạng thái thực tế]
- Job còn chờ reviewer/coach hoặc bước tiếp theo: [kiểm tra portal và ghi đúng trạng thái]

CVAT/portal là nơi nộp annotation đã Save, snapshot v1, feedback và v2/phản hồi. Repo này không thay thế các bước nộp trực tiếp đó.

## Nộp bài trên VLearn

- URL repo cá nhân: [điền URL sau khi tạo repo]
- Trạng thái liên kết/nộp trên VLearn: [điền trạng thái hiển thị]
- Output KITTI hiện lưu tại `D:\AI\Day-13\ket-qua-ca-nhan`; xác nhận với LC nếu nơi đối chiếu được chỉ định khác

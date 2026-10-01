# Báo cáo thực hành PointPillars — Day 13

## Nhóm và provenance

- Mã nhóm/phòng: 18 - Quỳ Hoa Bảo Điển
- Thành viên: xem TEAMMATES.md.
- Trạng thái thực hiện: executed-by-group
- E2E smoke: passed; thời gian theo output/smoke.json là 2026-10-01, 15:21:34–15:24:17 UTC+7. Đầu mối vận hành theo phân công: xem TEAMMATES.md, Operator lượt A.
- Máy và kiến trúc: host Windows; Docker Engine 29.5.3, Linux x86_64 tương ứng amd64; Python 3.10.10.
- Image tag: day13-pointpillars:lc-20261001-amd64.
- Image ID: sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82.
- Phiên bản repo: 0831856d921609312d42c7582c366e5a311bb7b1. Manifest ghi working_tree_dirty=true.
- PCD/frame và nơi chạy: KITTI Student demo.pcd, frame_id demo, 17.238 điểm. PCD hash: 3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60. Đây là mẫu KITTI 000008 được chuyển đổi theo manifest, không phải dữ liệu Robotaxi. Host chạy: workspace Windows của người dùng; nhóm/LC xác nhận quyền và nơi chạy theo ca.
- Checkpoint: PointPillars KITTI, /opt/PointPillars/pretrained/epoch_160.pth. SHA256: 482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1.
- Phạm vi model: front-window, x=[0.0,69.12] m, y=[-39.68,39.68] m, z=[-3.0,1.0] m trong model frame; lượt chạy dùng identity pass, không bật full-scene. Score threshold: 0.3.
- Kênh thứ tư/intensity: PCD không giữ reflectance thật. Adapter đưa giá trị hằng 0.0 cho vehicles và 0.7 cho pedestrian/two-wheels; RGB=0 trong PCD là placeholder, không phải intensity đo được.
- Nguồn z_ground: script ước lượng từ PCD. Lần chạy này z_ground=0.075 m. delta=1.73 m là giả định độ cao sensor của preset KITTI, không dùng làm hiệu chỉnh chung cho sensor khác.

### E2E local

File output/smoke.json ghi status=passed. Các bước docker-load, run-A, run-B, run-C và qc-cases đều passed. Bản sao output kèm báo cáo gồm 17 file và đã được kiểm tra hash với output gốc.

## Ba lượt inference thật

| Lượt | delta (m) | Pillar XY (m) | Số hộp | mean_z (m) | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | output/run-A/boxes-demo-delta-0-voxel-0.16.json; output/run-A/side-demo-delta-0-voxel-0.16.png; output/run-A/summary.csv | CSV ghi 1 hộp. JSON gán hộp đó là vehicles. |
| B | 1.73 | 0.16 | 13 | 1.034 | output/run-B/boxes-demo-delta-1.73-voxel-0.16.json; output/run-B/side-demo-delta-1.73-voxel-0.16.png; output/run-B/summary.csv | JSON ghi 10 vehicles, 2 pedestrian và 1 two-wheels. |
| C | 1.73 | 0.32 | 6 | 1.091 | output/run-C/boxes-demo-delta-1.73-voxel-0.32.json; output/run-C/side-demo-delta-1.73-voxel-0.32.png; output/run-C/summary.csv | JSON ghi 6 pedestrian. |

### Phân tích kỹ thuật

- A/B: script đổi z của điểm trước khi áp ROI và chạy model. A có 1 hộp, B có 13 hộp, nên kết quả không thể được giải thích bằng cách dịch mọi hộp sau inference đúng 1.73 m. Đây là khác biệt của dự đoán model trên hai đầu vào/cấu hình khác nhau; không có ground truth để nói lượt nào chính xác hơn.
- B/C: giữ delta=1.73 m và đổi pillar XY từ 0.16 m sang 0.32 m. Số hộp đổi từ 13 sang 6 và phân bố class cũng đổi. Số hộp hoặc score riêng lẻ không chứng minh chất lượng vì bài thực hành không cung cấp nhãn reference đã duyệt.
- ROI và Side: model chỉ nhận vùng front-window đã ghi ở trên. Vật ngoài vùng này không được tính là miss trong phép so sánh. PNG Side là hình chiếu x-z có lấy mẫu điểm và vẽ cuboid; nó không hiển thị trục y, ảnh camera hay nhãn chuẩn. Dùng CSV/JSON để trích số liệu. Không dùng PNG này một mình để kết luận class, yaw hoặc độ đúng của cuboid.
- Cần kiểm tiếp: nếu phân tích demo, đối chiếu JSON với point cloud và kiểm điều kiện ROI. Với job Robotaxi thật, chỉ đánh giá trong CVAT/viewer được cấp, dùng các góc nhìn và ảnh camera cùng frame theo LABEL_GUIDELINE.md.
- Khả năng import: không file JSON nào của gói Student được import vào job Robotaxi. A/B/C là KITTI frame demo khác dữ liệu Robotaxi; case-* là ví dụ training-only.

Công thức trong script: z_model = z_source - z_ground - delta; z_source = z_model + delta + z_ground. Với lần này, delta + z_ground = 1.805 m.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/kích thước/yaw có đổi? | Quyết định | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0/13 | 0 m | Không | Prediction B không bị biến đổi trong ca này; điều đó không chứng nhận nhãn đúng | output/qc-cases/case-correct.json; output/qc-cases/side-correct.png |
| case-batch-z | 13/13 | -1.805 m mỗi hộp | Không; chỉ z đổi | Dừng batch và báo LC kiểm pipeline/chuyển đổi hệ tọa độ | output/qc-cases/case-batch-z.json; output/qc-cases/side-batch-z.png |
| case-one-box-z | 1/13 | -1.805 m ở hộp đầu; 12 hộp còn lại không đổi | Không; chỉ z đổi ở một hộp | Kiểm riêng hộp đó qua nhiều góc nhìn | output/qc-cases/case-one-box-z.json; output/qc-cases/side-one-box-z.png |

Manifest QC ghi z_ground=0.075 m, delta=1.73 m và height_offset=1.805 m. Helper tạo ba ca từ prediction B. Đây là biến đổi có chủ đích để luyện chẩn đoán, không phải inference độc lập, ground truth hay dữ liệu được nhập CVAT.

## Nhận xét cá nhân

Các nhận xét dưới đây theo đủ năm mục của template, dựa trên vai trò được phân công và bằng chứng trong output.

### Trần Văn Bắc — Tự nhận xét

- **Vai trò phụ trách:** Operator lượt A, Config Inspector lượt B, Geometry Inspector lượt C; đối chiếu cấu hình, manifest và trạng thái E2E.
- **Quan sát kỹ thuật có dẫn chứng:** output/smoke.json ghi passed cho docker-load, run-A, run-B, run-C và qc-cases. Ba lượt inference có lần lượt 1, 13 và 6 hộp. Điều này xác nhận pipeline hoàn tất các bước đã định nghĩa, nhưng tự nó không chứng minh các hộp dự đoán đúng.
- **Diễn giải phép biến đổi z thuận/nghịch:** Chiều thuận z_model = z_source - z_ground - delta chuyển điểm từ hệ nguồn sang model frame; chiều nghịch z_source = z_model + delta + z_ground đưa kết quả về hệ nguồn. Ở lượt này z_ground = 0.075 m và delta = 1.73 m, tổng offset là 1.805 m. Delta là giả định của preset KITTI, không thể áp dụng cho sensor Robotaxi nếu chưa đo độ cao thực tế.
- **Quyết định lỗi batch và hành động:** Trong case-batch-z, cả 13/13 hộp lệch -1.805 m trong khi các thuộc tính khác không đổi. Em sẽ dừng dùng output của batch đó, kiểm tra phép đổi hệ tọa độ và chỉ tiếp tục sau khi xác minh lại pipeline; không sửa từng hộp bằng tay.
- **Điều chưa chắc chắn:** Chưa có ground truth để kết luận cấu hình nào chính xác hơn. Em cũng chưa thể xác nhận độ cao sensor thực tế của dữ liệu Robotaxi chỉ từ preset KITTI.

### Phạm Văn Phóng — Tự nhận xét

- **Vai trò phụ trách:** Config Inspector lượt A, Geometry Inspector lượt B, Log & Report Recorder lượt C; tập trung đối chiếu A/B khi delta thay đổi.
- **Quan sát kỹ thuật có dẫn chứng:** run-A dùng delta = 0 và pillar XY = 0.16 m, có 1 hộp vehicles. run-B dùng delta = 1.73 m cùng pillar XY = 0.16 m, có 13 hộp gồm 10 vehicles, 2 pedestrian và 1 two-wheels. Số liệu ở summary.csv và JSON cho thấy đầu vào/cấu hình z khác nhau đi cùng dự đoán khác nhau; nhiều hộp hơn không đồng nghĩa chất lượng cao hơn.
- **Diễn giải phép biến đổi z thuận/nghịch:** Script trừ z_ground và delta khỏi z điểm trước khi đưa vào model; khi đổi kết quả về hệ nguồn thì cộng lại hai giá trị đó. Nếu bỏ bước nghịch, độ cao hộp vẫn ở model frame. Việc đổi z trước inference cũng có thể làm điểm vào/ra khỏi ROI, nên không thể xem đây chỉ là tịnh tiến hộp sau dự đoán.
- **Quyết định lỗi batch và hành động:** Nếu thấy toàn bộ hộp cùng lệch như case-batch-z (13/13 hộp, -1.805 m), em sẽ dừng batch, báo người phụ trách pipeline và kiểm tra z_ground, delta cùng phép đổi ngược trước khi xử lý tiếp.
- **Điều chưa chắc chắn:** Chưa có nhãn chuẩn để tách riêng ảnh hưởng của thay đổi delta khỏi ảnh hưởng của ROI lên từng dự đoán. Cần đối chiếu point cloud và vùng ROI, không kết luận chỉ từ số hộp hoặc ảnh Side.

### Nguyễn Đức Anh — Tự nhận xét

- **Vai trò phụ trách:** Geometry Inspector lượt A, Log & Report Recorder lượt B, Operator lượt C; tập trung so sánh B/C khi pillar XY thay đổi.
- **Quan sát kỹ thuật có dẫn chứng:** run-B dùng pillar XY = 0.16 m và có 13 hộp; run-C giữ delta = 1.73 m nhưng tăng pillar XY lên 0.32 m và có 6 hộp, đều được JSON của run-C gán class pedestrian. summary.csv và JSON cho thấy thay đổi kích thước pillar đi cùng thay đổi số lượng và phân bố class, nhưng chưa có ground truth để gọi một cấu hình là tốt hơn.
- **Diễn giải phép biến đổi z thuận/nghịch:** B và C đều dùng delta = 1.73 m và cùng z_ground = 0.075 m, nên công thức z_model = z_source - z_ground - delta và phép nghịch z_source = z_model + delta + z_ground dùng cùng offset 1.805 m. Tham số pillar XY thay đổi độ phân giải lưới theo mặt phẳng ngang, không thay thế phép hiệu chỉnh z.
- **Quyết định lỗi batch và hành động:** Với lỗi lệch z đồng loạt như case-batch-z, em sẽ yêu cầu dừng và kiểm tra pipeline/hệ tọa độ trước khi dùng output. Với lỗi chỉ ở một hộp như case-one-box-z, em sẽ chuyển sang kiểm tra riêng hộp đó qua nhiều góc nhìn.
- **Điều chưa chắc chắn:** Chưa thể xác định thay đổi pillar làm tăng hay giảm độ chính xác nếu không có nhãn tham chiếu và phép so sánh trên cùng dữ liệu. Cần giữ các tham số khác cố định và kiểm tra thêm trên viewer 3D.

### Trần Quốc Trọng — Tự nhận xét

- **Vai trò phụ trách:** Log & Report Recorder lượt A, Operator lượt B, Config Inspector lượt C; phụ trách đối chiếu các ca QC và tập hợp bằng chứng.
- **Quan sát kỹ thuật có dẫn chứng:** Trong bộ QC, case-correct có 0/13 hộp đổi z; case-batch-z có 13/13 hộp lệch -1.805 m; case-one-box-z có 1/13 hộp lệch -1.805 m. Manifest cho biết các ca này được tạo từ prediction B để luyện chẩn đoán, không phải inference độc lập hay ground truth.
- **Diễn giải phép biến đổi z thuận/nghịch:** Tổng offset của lượt này là z_ground + delta = 0.075 + 1.73 = 1.805 m. Chiều thuận đưa điểm vào model frame bằng phép trừ; chiều nghịch phải cộng lại offset để đưa hộp về hệ nguồn. Giá trị này phụ thuộc cấu hình và hệ tọa độ đang dùng.
- **Quyết định lỗi batch và hành động:** Khi tất cả hộp cùng lệch, em sẽ dừng batch và báo kiểm tra bước chuyển đổi tọa độ. Khi chỉ một hộp lệch, em sẽ khoanh vùng đối tượng đó để kiểm tra cục bộ. Em sẽ không đưa các JSON case QC vào job CVAT.
- **Điều chưa chắc chắn:** Ảnh Side chỉ là hình chiếu x-z, không hiển thị trục y hoặc ảnh camera; vì vậy em chưa thể xác nhận đầy đủ vị trí 3D, yaw hay class chỉ từ hình này. Cần xem viewer 3D, ảnh camera cùng frame và nhãn tham chiếu nếu có.
## LC ghi nhận riêng

> LC ghi nhận ngày 01/10/2026. **Kết luận: ĐẠT (sau khi gửi lại).**

- **Quyền dùng PCD/image và đúng ca:** Gói Student KITTI 000008 (giấy phép CC BY-NC-SA 3.0), không dùng dữ liệu Robotaxi. Input SHA-256 `3b5ea3da…` và image `sha256:e03983bd…` (amd64) khớp `smoke.json`.
- **Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:** Chạy thật. `smoke.json` passed: nạp image 129,3 s, A/B/C 10,1 / 9,8 / 7,8 s, kết quả 1/13/6. Trạng thái trong báo cáo vẫn ghi "Hoàn thành".
- **Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:** Đủ output. Không sửa JSON. Không đưa ca lỗi vào CVAT.
- **Nhận xét từng thành viên và quyết định dừng pipeline:** **Đã bổ sung** nhận xét cá nhân: đủ 4 người × 5 ý. Phép z đúng (thuận trừ, nghịch cộng 1,805 m), quyết định batch/one-box đúng, nêu được giới hạn. Văn phong 4 mục rất giống nhau: LC hỏi miệng một người bất kỳ.
- **Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:** **Đồng ý chuyển sang chỉnh/QC.**

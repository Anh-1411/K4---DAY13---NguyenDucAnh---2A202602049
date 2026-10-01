# Thành viên nhóm — Day 13

Giữ bản đã điền trong thư mục nhóm private do LC thu. Không commit họ tên hoặc MSSV lên repo public.

Mã nhóm/phòng: 18 - Quỳ Hoa Bảo Điển

Trong PRE-LABEL-REPORT.md, Thành viên 1–4 tương ứng thứ tự bốn người trong bảng dưới đây.

| Họ và tên | MSSV | Vai trò lượt A | Vai trò lượt B | Vai trò lượt C |
| --- | --- | --- | --- | --- |
| Trần Văn Bắc | 2A202602089 | Operator | Config Inspector | Geometry Inspector |
| Phạm Văn Phóng | 2A202602064 | Config Inspector | Geometry Inspector | Log & Report Recorder |
| Nguyễn Đức Anh | 2A202602049 | Geometry Inspector | Log & Report Recorder | Operator |
| Trần Quốc Trọng | 2A202602123 | Log & Report Recorder | Operator | Config Inspector |

Đây là phương án phân công công việc và vai trò theo lượt; nhóm đối chiếu với phần việc thực tế đã hoàn thành trước khi nộp.

## Phần việc từng người

| Thành viên | Phần việc phụ trách | Bằng chứng cần đối chiếu |
| --- | --- | --- |
| Trần Văn Bắc | Vận hành lượt A; đối chiếu manifest/cấu hình và trạng thái E2E; tập hợp output. | output/smoke.json; manifest; summary.csv |
| Phạm Văn Phóng | Phân tích A/B khi delta thay đổi; ghi số hộp/class và giới hạn khi không có nhãn chuẩn. | run-A và run-B: summary.csv, JSON |
| Nguyễn Đức Anh | Vận hành lượt C; phân tích B/C khi kích thước pillar XY thay đổi; nêu giới hạn khi không có nhãn chuẩn. | run-B và run-C: summary.csv, JSON |
| Trần Quốc Trọng | Vận hành lượt B; kiểm tra các ca QC, đối chiếu số hộp lệch z và tập hợp bằng chứng. | qc-cases/manifest.json và các JSON case |

Phân công này giao người phụ trách từng phần; không thay thế nhận xét cá nhân. Mỗi thành viên tự ghi quan sát sau khi xem bằng chứng được giao.

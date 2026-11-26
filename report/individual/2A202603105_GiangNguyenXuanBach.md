# Báo cáo Day 13 — Giang Nguyễn Xuân Bách, 2A202603105

## 1. Kết luận trước
Mã số học viên: 2A202603105, task `D13-2A202603105`, danh sách 10 frame: `s01f10, s01f39, s02f27, s02f46, s03f01, s03f19, s04f13, s04f35, s05f05, s05f34`. Hệ điều hành: Windows 11, trình duyệt: Google Chrome, phiên bản CVAT: v2.81.0.
Tổng số cuboid đã vẽ: 106 cuboid. Frame có số cuboid ít nhất là `s01f39` và `s02f46` (đều có 6 cuboid) do các đoạn đường này tương đối vắng vẻ, số phương tiện di chuyển trong phạm vi 50 m ít. Phiếu tự kiểm `selfcheck` ghi nhận 0/106 cuboid còn cảnh báo sau khi rà soát và tinh chỉnh. Tự tin nhất ở phần xác định class, căn chỉnh hướng mũi tên yaw dọc đầu xe và điều chỉnh đáy hộp bám sát mặt đường; phần cảm thấy còn khó khăn nhất là ước lượng chiều dài xe ở cự ly xa do mật độ tia LiDAR bị thưa.

Cách dùng CVAT:
- Đổi góc nhìn: Kéo chuột trái để xoay khung 3D phối cảnh, kéo chuột phải để dời không gian nhìn, lăn con lăn chuột để phóng to/thu nhỏ
- Vẽ và chỉnh cuboid: Chọn nhãn tương ứng, bấm nút Shape, nháy đúp chuột vào giữa cụm điểm để tạo hộp; căn chỉnh tâm, chiều dài/rộng và xoay góc yaw trên khung Top; chỉnh chiều cao và đáy chạm mặt đường trên hai khung Side và Front.
- Bật hướng và xem trên camera: Vào menu Appearance bật tùy chọn Cuboid orientation để hiện vạch đỏ chỉ hướng đầu xe; đối chiếu với 7 góc camera (cam01–cam09) hoặc chạy script `preview` để kiểm tra độ khớp của cuboid trên ảnh quang học.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f10`, `s01f39`. Ảnh chụp: `t1-s01f10-3d.PNG`, `t1-s01f10-top.PNG`, `t1-s01f39-3d.PNG`, `t1-s01f39-top.PNG`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f10 | Cạnh dưới (hướng -y); +x chỉ sang bên trái, +y chỉ về phía sau | Phẳng; thấp hơn LiDAR khoảng 1.7 m (z ≈ -1.7 m) | ~46 m (xe buýt phía trước bên phải) | Car, Truck, Bus | 2 xe con chạy phía sau (+y) và 1 xe tải ở làn đối diện (+x) | Cụm điểm phản xạ thưa ở dải phân cách phía xa |
| s01f39 | Cạnh dưới (hướng -y); +x chỉ sang bên trái, +y chỉ về phía sau | Phẳng; thấp hơn LiDAR khoảng 1.7 m | ~39 m (xe con cùng chiều phía trước) | Car | 1 xe con bám đuôi ngay sát phía sau xe thu dữ liệu (~3.5 m) | Cụm điểm cành cây tán xạ không rõ hình dạng |

3 điều **chỉ camera** cho biết:
1. Màu sơn ngoại thất xe và các dấu hiệu nhận dạng đặc thù (xe taxi, xe cứu thương, xe tuần tra).
2. Chi tiết văn bản, biển số xe, nội dung biển chỉ dẫn và tín hiệu đèn giao thông (đỏ/vàng/xanh).
3. Nhận diện người, khuôn mặt, trang phục và phân biệt rõ người đi bộ bình thường với người đang dắt xe máy.

3 điều **chỉ LiDAR** cho biết:
1. Khoảng cách vật lý 3D chính xác tuyệt đối theo mét (tọa độ x, y, z) không bị biến dạng bởi phối cảnh thấu kính.
2. Hoạt động độc lập hoàn toàn với điều kiện chiếu sáng, không bị cản trở bởi trời tối hoặc ánh sáng chói chang.
3. Cung cấp góc quét 360 độ đồng bộ xung quanh xe chỉ với một cụm dữ liệu duy nhất trong cùng một thời điểm.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f27 | Cạnh dưới (hướng -y) | Có người đi bộ đứng trên vỉa hè bên phải |
| s02f46 | Cạnh dưới (hướng -y) | Đường rất vắng, mật độ điểm nền sạch và đồng đều |
| s03f01 | Cạnh dưới (hướng -y) | Xe tải nhỏ chở hàng di chuyển song song làn bên cạnh |
| s03f19 | Cạnh dưới (hướng -y) | Mật độ phương tiện dày đặc nhất (19 cuboid), nhiều xe máy và người đi bộ |
| s04f13 | Cạnh dưới (hướng -y) | Xe buýt lớn ở làn ngược chiều tạo vệt bóng che kéo dài |
| s04f35 | Cạnh dưới (hướng -y) | Tán cây ven đường tạo nhiều điểm nhiễu lơ lửng |
| s05f05 | Cạnh dưới (hướng -y) | Xe con bám rất sát phía sau xe thu dữ liệu (~3.3 m) |
| s05f34 | Cạnh dưới (hướng -y) | Xe buýt ở rất xa , mật độ điểm chỉ còn vài vệt mỏng |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh chụp: `t2-a-car.PNG`, `t2-b-bus-truck.PNG`, `t2-c-ped-cyclist.PNG`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f10, 1 | Car | Đúng  | Đúng | Đã kiểm đủ 3 view và ảnh cam01 | Có |
| (b) xe buýt hoặc xe tải | s01f10, 7 | Truck | Đúng | Đúng | Đã kiểm đủ 3 view và ảnh camera | Có |
| (c) người đi bộ hoặc xe hai bánh | s03f19, 18 | Cyclist | Đúng | Đúng | Đã kiểm đủ 3 view và ảnh camera | Có |

Hiệu chuẩn và vật bỏ qua:
- Vòng hiệu chuẩn: được miễn để tập trung làm bài cá nhân theo MSSV
- Vật bỏ qua vì không đủ cơ sở: Tại frame `s04f35`,  xuất hiện cụm điểm mờ nhạt ở lề đường nhưng không xác định được hình dạng nên đã bỏ qua theo phân loại `LOW_VISIBILITY`.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Ảnh chụp: `t4-1-bong-che.PNG`, `t4-2-thua-o-xa.PNG`, `t4-3-phan-xa-thap.PNG`, `t4-4-diem-la.PNG`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f10 / t4-1-bong-che.PNG / task cá nhân | Tia laser LiDAR truyền thẳng bị thân xe phía trước cản lại, không chiếu tới được khu vực phía sau | Vùng rỗng không có điểm LiDAR phía sau thân xe tải tạo thành dải bóng khuyết, dù mặt đường hai bên vẫn có điểm | Bật thuộc tính occluded = 1, cuboid bao trọn phần suy luận được dựa trên kích thước xe tiêu chuẩn |
| 2. Thưa ở xa | s05f34 / t4-2-thua-o-xa.PNG / task cá nhân | Góc mở giữa các tia laser tăng dần theo khoảng cách khiến mật độ tia phản xạ giảm ở cự ly xa | Xe buýt ở cự ly xa chỉ còn ít điểm rời rạc tạo thành các vệt mỏng ngang | Nếu nhận ra >15-20% thể tích và có ảnh camera đối chiếu thì dựng hộp theo bảng 4.3; nếu dưới 15% thì bỏ qua  |
| 3. Phản xạ thấp / ít điểm bất thường | s01f10 / t4-3-phan-xa-thap.PNG / task cá nhân | Bề mặt xe sơn màu đen tối | Thân xe ít điểm hơn hẳn so với xe màu sáng ở cùng khoảng cách, giá trị cường độ thấp | Ghi nhận giả thuyết, không được co nhỏ hộp theo cụm điểm ít ỏi mà căn hộp đủ kích thước thực tế theo bảng 4.3 |
| 4. Điểm lạ | s02f35 / t4-4-diem-la.PNG / task gợi ý | Hiện tượng tán xạ đa đường qua bề mặt gương kính hoặc | Cụm điểm xuất hiện| Không gán cuboid thành vật thể, loại bỏ hoàn toàn khỏi danh sách gán nhãn |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603105-bangA.csv` và `submission/D13-2A202603105-tukiem.csv`. Mặt đường khoảng -1.7 m:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f10, 1, Car | -1.032 | 1.469 | -1.767 | 6.22 | 6.30 | 269.5 | Chạm mặt đường chuẩn (-1.77 m ≈ -1.7 m) |
| s01f10, 7, Truck | -0.628 | 1.927 | -1.591 | 24.38 | 24.39 | 89.7 | Chạm mặt đường chuẩn (-1.59 m) |
| s03f19, 18, Cyclist | -0.822 | 1.507 | -1.575 | 9.05 | 9.09 | 334.0 | Chạm mặt đường chuẩn (-1.58 m) |

Số đo cự ly, kích thước và góc quay:
- (a) Cuboid gần nhất và xa nhất:
  + Cuboid gần nhất: frame `s05f05`, idx `4`, class `Car` có `dist_m = 3.30 m` và khoảng cách không gian 3D `r = 3.40 m`.
  + Cuboid xa nhất: frame `s05f34`, idx `6`, class `Bus` có `dist_m = 57.89 m` và khoảng cách không gian 3D `r = 57.90 m`.
- (b) Kích thước l, w, h của một người đi bộ (hoặc xe hai bánh) và một xe con so với bảng 4.3:
  + Xe con (`s01f10, 1, Car`): dài 4.634 m, rộng 2.296 m, cao 1.469 m. Chiều rộng 2.296 m hơi nhỉnh hơn so với bảng 4.3 (1.80–2.10 m) do xe mở hai gương chiếu hậu và đám mây điểm bao gồm cả hai tai gương xe.
  + Xe máy (`s03f19, 18, Cyclist`): dài 1.757 m, rộng 0.699 m, cao 1.507 m. Kích thước nằm chuẩn xác trong khoảng tham chiếu của bảng 4.3 (dài 1.8–2.3 m, rộng 0.70–0.95 m, cao 1.40–1.80 m).
- (c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:
  + Xe cùng chiều phía trước (`s01f10, 1, Car`): `yaw_deg = 269.5°`, rất gần giá trị lý thuyết 270.0° (hướng di chuyển dọc theo trục -y).
  + Xe ngược chiều (`s01f10, 7, Truck`): `yaw_deg = 89.7°`, rất gần giá trị lý thuyết 90.0° (hướng di chuyển dọc theo trục +y).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f10, 1, Car | 6.22 | 2406 |
| s01f39, 2, Car | 15.61 | 795 |
| s01f10, 2, Car | 24.60 | 262 |
| s01f39, 4, Car | 38.80 | 70 |

Quy luật mật độ điểm và tự kiểm:
- Quy luật: Khi khoảng cách tăng gấp đôi thì số điểm phản xạ giảm khoảng 3 đến 4 lần (tỷ lệ nghịch với bình phương khoảng cách). Dữ liệu thực tế đo được (2406 điểm ở 6.2 m; 795 điểm ở 15.6 m; 262 điểm ở 24.6 m; 70 điểm ở 38.8 m) hoàn toàn khớp với đường cong phân bố trung vị trong slide bài giảng đối với xe con (khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m).
- Tự kiểm: Hai cuboid đã chỉnh sửa sau khi chạy `preview` và `selfcheck`:
  1. Cuboid `s01f10, 8, Car`: Ban đầu đáy hộp bị đặt hơi sâu (z đáy < -2.1 m), đã nâng đáy trên khung Side/Front về z đáy = -1.977 m để tiếp xúc đúng mặt đường dốc nhẹ.
  2. Cuboid `s01f10, 7, Truck`: Ban đầu góc yaw bị ngược hướng di chuyển do nhầm đầu/đuôi xe, đã xoay 180° trên khung Top khớp với hướng xe chạy trên camera ngược chiều.
  Sau khi chỉnh sửa và chạy lại `sync`, toàn bộ 0/106 cuboid không còn cảnh báo.
- Đổi kiểm: 

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa nhãn 3D đối với mô hình học máy:
- Nhãn 3D đo bằng mét vì trong hệ thống xe tự hành , mô hình dự đoán quỹ đạo và lập kế hoạch di chuyển  cần biết chính xác khoảng cách vật lý thực tế trong không gian 3D để tính toán thời gian va chạm và khoảng cách phanh an toàn; pixel trên ảnh 2D phụ thuộc vào góc đặt và tiêu cự camera, bị biến dạng quang học nên không thể dùng để định vị khoảng cách tuyệt đối.
- Vật thưa điểm cần quy tắc riêng vì ở cự ly xa hoặc vị trí bị che khuất, số điểm LiDAR quá ít không đủ tái hiện hình dáng hình học; nếu không có chuẩn quy định , người gán nhãn sẽ phỏng đoán tùy tiện, gây nhiễu và làm sai lệch trọng số khi huấn luyện mô hình phát hiện vật thể 3D.

(b) Sử dụng AI:
- Công cụ: Antigravity
- Câu hỏi chính: Đọc và hướng dẫn lại guideline, guide, submission... theo 1 cách tuyến tính dễ hiểu hiwn

(c) Điều còn chưa chắc:
- Tại frame `s03f19` có nhiều người đi sát nhau, khó khăn trong việc xác định

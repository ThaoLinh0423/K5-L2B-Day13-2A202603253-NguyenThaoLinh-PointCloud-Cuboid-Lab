# Báo cáo Day 13 — Nguyễn Thảo Linh, 2A202603253

## 1. Kết luận trước

- Mã số học viên (MSSV): `2A202603253`. Task CVAT: `D13-2A202603253`.
- 10 frame được giao: `s01f05`, `s01f52`, `s02f10`, `s02f36`, `s03f13`, `s03f55`, `s04f20`, `s04f39`, `s05f04`, `s05f23`.
- Môi trường: Windows 11, trình duyệt Chrome, CVAT v2.83.0.
- MSSV bạn ngồi cạnh: `2A202603252`.
- Tổng kết kết quả: 66 cuboid dạng Shape trên 10 frame (56 Car, 4 Cyclist, 3 Truck, 3 Bus), các frame ít cuboid nhất là `s03f55`, `s05f04` và `s05f23` do các đối tượng rõ ràng nằm chủ yếu ở gần 3–22 m, còn các vật xa hơn bị thưa điểm hoặc che khuất nên không vẽ đoán. Phiếu tự kiểm `selfcheck` 0/66 cuboid bị gắn cờ cảnh báo. Điểm tự tin là hướng chuyển động và phân loại class của các phương tiện ở gần (≤ 20 m); phần còn yếu nhất là căn chiều dài/rộng sát kích thước thực tế đối với xe hai bánh (`Cyclist`) và một số xe con bị che một phần.

Cách dùng CVAT 3D:
- Đổi góc nhìn: Trong khung 3D , kéo chuột trái để xoay (xoay thẳng góc từ trên xuống để quét toàn cảnh 360°), kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ và nháy đúp vào khoảng trống để đưa về góc mặc định.
- Vẽ và chỉnh cuboid: Chọn công cụ vẽ cuboid ở thanh bên trái → chọn đúng class → bấm Shape (không bấm Track) → đưa hộp tới cụm điểm và nháy đúp để đặt (`N` để vẽ tiếp, `Esc` để huỷ); chọn cuboid rồi chỉnh tâm, dài, rộng, góc `yaw` ở khung Top, còn khung Side và Front chỉ dùng để chỉnh độ cao đáy chạm mặt đường và chiều cao hộp (không xoay ở Side/Front).
- Bật mũi tên hướng và kiểm trên ảnh camera: Bật Appearance → Cuboid orientation để hiển thị vạch đỏ ngắn (`+X` của cuboid), xoay chấm xanh trong Top cho vạch đỏ chỉ về đầu xe; bấm `Ctrl+S` để lưu và chạy `python scripts/lab.py preview` để đối chiếu các hộp chiếu lên đủ 7 ảnh camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f05` và `s01f52`. Tên ảnh chụp: `t1-s01f05-3d.png`, `t1-s01f05-top.png`, `t1-s01f52-3d.png`, `t1-s01f52-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước `−y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f05` | Phía trước xe (`−y`, trùng hướng nhìn `cam01`) ở **cạnh trái** của ảnh nhìn từ trên xuống; `+x` (trục đỏ/cam, bên trái xe) chỉ xuống **cạnh dưới**, `+y` (trục xanh lá, phía sau xe) chỉ sang **cạnh phải** | Tương đối phẳng quanh xe thu dữ liệu, thấp hơn LiDAR khoảng `1,7 m` (`z_bottom` các xe gần quanh `−1,63 m` đến `−1,79 m`; làn xa bên trái hơi thấp hơn, khoảng `−2,16 m`) | `~46,8 m` (xe con `s01f05#5` ở `46,80 m`, còn 51 điểm) | Car, Truck | Có 5 vật phía sau (`y > 0`), gồm Car sát trái-sau (`5,46 m`, `cam05`), 2 Car phía sau (`16,90 m` và `26,42 m`, `cam09`), cùng 1 Car (`14,35 m`) và 1 Truck (`21,67 m`) đi ngược chiều bên trái-sau | Các cụm điểm rất thưa ngoài ngưỡng `50 m` và vùng nằm sau bóng che thân xe tải bên trái-sau không đủ hình dạng để nhận diện class |
| `s01f52` | Phía trước xe (`−y`, trùng hướng nhìn `cam01`) ở **cạnh trái** (hơi chếch lên trên) của ảnh nhìn từ trên xuống; `+x` (bên trái xe) chỉ xuống **cạnh dưới** (hơi lệch trái), `+y` (phía sau xe) chỉ sang **cạnh phải** (hơi chếch xuống dưới) | Phẳng, thấp hơn LiDAR khoảng `1,75–1,82 m` (`z_bottom` các xe đều trong khoảng `−1,750 m` đến `−1,815 m`) | `~16,3 m` đọc rõ chi tiết (xe con `s01f52#6` ở `16,29 m`, 390 điểm; các cụm xa hơn trên `30 m` thưa điểm) | Car | 1 Car bám sát ngay phía sau (`6,63 m`, `cam09`), 1 Car bên trái-sau (`9,37 m`, `cam05`) và 1 Car chạy song song bên trái (`6,03 m`, `cam06`) | Một số cụm điểm nhỏ ở rìa vỉa hè/hàng rào và các cụm điểm bị che một phần ở làn xa phía trước không tách rõ biên hình học |

So sánh thông tin giữa camera và LiDAR trên hai frame `s01f05` và `s01f52`:
- 3 điều chỉ camera cho biết: (1) Phân biệt rõ đầu xe và đuôi xe (nhờ đèn pha, đèn hậu, kính lái) cũng như nhận diện trực quan loại xe dễ nhầm (xe con, bán tải, xe tải hộp); (2) Màu sắc phương tiện, vạch kẻ làn đường, biển báo và đèn tín hiệu giao thông; (3) Ngữ cảnh chi tiết của vật thể (xe có người lái hay đỗ trống, bề mặt kính/sơn tối màu ít điểm phản xạ trên LiDAR).
- 3 điều chỉ LiDAR cho biết: (1) Khoảng cách thực theo đơn vị mét tới từng vật thể (ví dụ `5,46 m` hay `46,80 m`); (2) Kích thước hình học 3 chiều thực tế (dài, rộng, cao theo mét) và độ cao đáy so với mặt đường (`z_bottom` quanh `−1,7 m`); (3) Trường quan sát liên tục `360°` không bị khe góc mù giữa các góc máy camera và biên hình học của vùng bóng che phía sau vật cản.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f10` | Cạnh trái (`−y`) | Có 2 xe buýt lớn đi ngược chiều bên trái (`18,57 m` và `39,62 m`) tạo bóng che rộng phía sau thân xe, và 2 Cyclist bên phải (`8,34 m` và `15,41 m`) |
| `s02f36` | Cạnh trái (`−y`) | Có 1 Car đi ngược chiều rất sát bên trái (`4,27 m`) và 2 Car chạy ở tầm xa phía sau trên `31 m` (`31,19 m` và `31,44 m`) |
| `s03f13` | Cạnh trái (`−y`) | Mật độ xe cùng chiều khá dày quanh xe (Car gần nhất bên trái chỉ `3,49 m`); xe tải và xe con ở khoảng `14–16 m` phía trước có đáy thấp (`z_bottom ≈ −2,02 m` đến `−2,03 m`) do mặt đường hơi dốc xuống |
| `s03f55` | Cạnh trái (`−y`) | Có 1 xe buýt đi ngược chiều bên trái-sau (`21,52 m`, `ground_gap_m = 0,32 m`) và 1 Car khá xa phía trước (`36,70 m`, còn 78 điểm) |
| `s04f20` | Cạnh trái (`−y`) | Đông đối tượng nhất trong 10 frame (10 cuboid trong vòng `18,22 m`), gồm 9 Car ở cả phía trước, hai bên, phía sau và 1 Cyclist phía sau (`12,89 m`) |
| `s04f39` | Cạnh trái (`−y`) | Toàn bộ các vật gán đều là Car ở khoảng cách `5,16–26,62 m`, gồm cả xe cùng chiều và ngược chiều |
| `s05f04` | Cạnh trái (`−y`) | Có 1 Car sát ngay bên phải (`3,42 m`, dày tới 5.644 điểm) và 1 Truck đi sát phía sau (`9,95 m`, 2.610 điểm) |
| `s05f23` | Cạnh trái (`−y`) | Các xe phân bố đều ở tầm trung `10,14–17,58 m`, phần lớn cùng chiều, có 1 Car đi ngược chiều bên trái (`14,48 m`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Tên ảnh chụp: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f05`, `idx = 8` | `Car` | Đúng (`yaw_deg = 270,0°`, cùng chiều xe thu dữ liệu) | Có (`z_bottom = −1,760 m`, `ground_gap_m = 0,02 m`) | Có (đủ 6 mặt ở Top/Side/Front và đối chiếu `cam05`/`preview`) | Có (`4,437 × 1,790 × 1,477 m`, nằm trong ngưỡng Sedan/Hatchback `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f05`, `idx = 2` | `Truck` | Đúng (`yaw_deg = 90,0°`, ngược chiều xe thu dữ liệu) | Có (`z_bottom = −1,730 m`, `ground_gap_m = 0,11 m`) | Có (đủ Top/Side/Front và đối chiếu camera/`preview`) | Có (`5,700 × 2,280 × 2,960 m`, nằm trong ngưỡng Xe tải hộp/nhẹ `5,5–9,0 × 2,00–2,55 × 2,40–3,50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s02f10`, `idx = 7` *(lấy ở frame 3 do `s01f05`, `s01f52` không có Pedestrian/Cyclist)* | `Cyclist` | Đúng (`yaw_deg = 270,0°`, cùng chiều xe thu dữ liệu) | Có (`z_bottom = −1,620 m`, `ground_gap_m = 0,07 m`) | Có (đủ Top/Side/Front và đối chiếu camera/`preview`) | Gần đạt (`1,650 × 0,650 × 1,560 m`; dài và rộng trong ngưỡng xe đạp có người `1,6–1,9 × 0,50–0,70 m`, chiều cao `1,560 m` thấp hơn `0,04 m` so với mốc `1,60 m` nhưng vẫn thuộc ngưỡng xe máy có người `1,40–1,80 m`) |

- Vòng hiệu chuẩn (task chung `D13-HC485087`): Qua đối chiếu Bảng A của cả lớp, cuboid của tôi ban đầu lệch chủ yếu ở việc căn mép hộp hơi sát cụm điểm thấy được (dễ bị thiếu phần đuôi/thân xe bị che) và chênh nhẹ ở độ cao đáy `z_bottom` khi mới nhìn ở khung Top. Cả lớp thống nhất cách hiểu theo guideline: luôn căn vạch đỏ `+X` về đầu xe theo ảnh camera, bắt buộc kiểm tra đáy chạm mặt đường ở cả khung Side và Front, và bao trọn phần thể tích suy luận được nhưng không nới dư ra ngoài biên vật.
- Các vật bỏ qua vì không đủ cơ sở:
  - Các cụm điểm ở vùng rìa `35–50 m` và ngoài `50 m` trên các frame (ví dụ phía trước xa ở `s01f05`, `s01f52`, `s03f55`) chỉ còn vài điểm rải rác dưới ngưỡng `15–20%` thể tích, không đủ cơ sở dựng hộp (loại nghi vấn: `LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).
  - Các phương tiện nằm sâu sau bóng che của xe tải/xe buýt lớn (như vùng phía sau `s02f10#5` và `s03f55#2`) hoặc xe hai bánh dựng bên lề không rõ có người điều khiển hay không (loại nghi vấn: `LOW_VISIBILITY` / `UNCERTAIN_CLASS` — theo guideline xe hai bánh không có người thì không vẽ).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f10` / `t4-1-bong-che.png` / frame của mình (`D13-2A202603253`)  | Giả thuyết:Thân xe buýt cỡ lớn (`s02f10#5`, dài `12,390 m`, cao `2,980 m` ở cách `18,57 m`) chắn toàn bộ các tia laser truyền thẳng từ LiDAR nên phía sau nó không có tia nào chiếu tới được | Phía sau thân xe buýt xuất hiện một vùng trống hình quạt kéo dài không có điểm mặt đường và không có điểm vật thể, tương phản rõ với vùng dày điểm ở hai bên | Vùng trống là khu vực chưa quan sát được (không kết luận là đường trống); với vật bị che một phần thì bật `occluded` và dựng cuboid bao cả phần thể tích suy luận được |
| 2. Thưa ở xa | `s01f05` / `t4-2-thua-o-xa.png` / frame của mình (`D13-2A202603253`) | Giả thuyết: Do các tia LiDAR 40 tia tách rộng dần theo khoảng cách (góc phân kỳ dọc và ngang cố định), ở tầm xa `40–50 m` chỉ còn rất ít tia cắt trúng thân xe | Xe con `s01f05#5` ở `dist_m = 46,80 m` chỉ còn `51` điểm thưa thớt (chủ yếu ở vài đường quét ngang), các vật xa hơn gần như chỉ còn vài điểm lẻ khó thấy rõ góc hộp | Áp dụng quy tắc `15–20%`: vì `s01f05#5` vẫn nhận ra được hình dạng tổng thể và đối chiếu được camera nên vẫn vẽ kèm tham chiếu Bảng 4.3; các cụm thưa hơn nữa không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f05` / `t4-3-phan-xa-thap.png` / frame của mình (`D13-2A202603253`) | Giả thuyết: Bề mặt sơn tối màu, kính xe hoặc góc tới xiên làm hấp thụ/phản xạ gương tia laser (có thể kết hợp che khuất một phần), nên tín hiệu phản hồi về cảm biến yếu và mất điểm | Xe con `s01f05#1` ở `dist_m = 14,35 m` chỉ có `257` điểm — ít hơn hẳn so với các xe con khác cùng tầm `14–15 m` (như `s01f05#3` ở `15,08 m` có `615` điểm hay `s05f23#1` ở `14,48 m` có `1.115` điểm) | Không khẳng định nguyên nhân (chỉ ghi giả thuyết); phải đối chiếu kỹ ảnh camera và tham chiếu Bảng 4.3 để tránh vẽ hộp bị thu nhỏ hoặc hụt chiều cao theo vùng thiếu điểm |
| 4. Điểm lạ | `s02f36` / `t4-4-diem-la.png` / frame của mình (`D13-2A202603253`) | Giả thuyết: Nhiễu phản xạ đa đường (multipath), tia laser chạm rìa dải phân cách/bụi hoặc tín hiệu phản hồi lạc (ghost points) khiến cảm biến ghi nhận sai vị trí điểm trong không gian | Xuất hiện các điểm lẻ nổi lơ lửng giữa không khí (cặp điểm sáng tách biệt phía trên mặt đường/dải phân cách gần xe `s02f36#1` và cụm điểm rời rạc không gắn với cấu trúc vật thể nào) | Đây là điểm nhiễu sai hình học, không thuộc phương tiện hay người đi bộ nên tuyệt đối không gán thành vật thể theo guideline |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu dưới đây lấy trực tiếp từ `submission/D13-2A202603253-bangA.csv` và `submission/D13-2A202603253-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f05`, `idx = 8`, `Car` | `−1,022 m` | `1,477 m` | `−1,760 m` | `5,46 m` | `5,55 m` | `270,0°` | Chạm đường (lệch mốc `−1,7 m` chỉ `0,060 m`; `ground_gap_m = 0,02 m`) |
| `s01f05`, `idx = 4`, `Car` | `−1,130 m` | `2,050 m` | `−2,155 m` | `20,35 m` | `20,39 m` | `91,1°` | Thấp hơn mốc `−1,7 m` khoảng `0,455 m`, nhưng `ground_gap_m = 0,10 m` cho thấy mặt đường cục bộ làn trái thấp hơn nên thực tế vẫn chạm đường(không chìm) |
| `s01f05`, `idx = 1`, `Car` | `−0,900 m` | `1,218 m` | `−1,509 m` | `14,35 m` | `14,38 m` | `92,8°` | Cao hơn mốc `−1,7 m` khoảng `0,191 m` (trong biên `±0,3 m`), nhưng `ground_gap_m = 0,39 m` cho thấy đáy hơi lơ lửng nhẹ so với mặt đường cục bộ |

- (a) Cuboid gần nhất và xa nhất:
  - Gần nhất: `s05f04`, `idx = 2`, `Car` (`x = −3,379 m`, `y = −0,554 m`, `z = −0,842 m`) có `dist_m = 3,42 m` và khoảng cách 3D `r = 3,53 m`.
  - Xa nhất: `s01f05`, `idx = 5`, `Car` (`x = 13,880 m`, `y = −44,690 m`, `z = −1,040 m`) có `dist_m = 46,80 m` và khoảng cách 3D `r = 46,81 m`.
- (b) So sánh kích thước với Bảng 4.3:
  - Xe hai bánh (`Cyclist`): `s02f10`, `idx = 2`, `Cyclist` có `l = 1,270 m`, `w = 0,680 m`, `h = 1,640 m`. So với Bảng 4.3 (xe đạp có người: `1,6–1,9 × 0,50–0,70 × 1,60–1,90 m`; xe máy có người: `1,8–2,3 × 0,70–0,95 × 1,40–1,80 m`), chiều rộng và chiều cao nằm trong khoảng chuẩn, riêng chiều dài `1,270 m` nằm dưới ngưỡng chuẩn** (`1,6 m`), do góc quét ngang (`yaw_deg = 5,7°`) khiến phần bánh trước/sau ít điểm nên hộp bị co ngắn theo cụm điểm thấy được.
  - Xe con (`Car`): `s01f52`, `idx = 4`, `Car` có `l = 4,320 m`, `w = 2,150 m`, `h = 1,420 m`. So với Bảng 4.3 (Sedan/Hatchback: `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`; SUV: `4,4–5,3 × 1,80–2,10 × 1,65–2,10 m`), chiều dài và chiều cao đạt chuẩn, riêng chiều rộng `2,150 m` vượt nhẹ ngưỡng trên (`1,90 m` / `2,10 m`) do hộp bao cả phần gương chiếu hậu hai bên và một ít điểm rìa thân xe.
- (c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:
  - Xe cùng chiều phía trước: `s01f05`, `idx = 3`, `Car` (`x = 2,230 m`, `y = −14,910 m`) có `yaw_deg = 270,0°`, khớp chính xác mốc `270°` (hướng `−y`).
  - Xe ngược chiều: `s02f10`, `idx = 1`, `Car` (`x = 3,930 m`, `y = −11,580 m`) có `yaw_deg = 90,0°` (và `s01f05`, `idx = 4`, `Car` có `yaw_deg = 91,1°`), khớp rất sát mốc `90°` (hướng `+y`).

Mật độ theo khoảng cách (4 cuboid `Car` ở 4 bin `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---:|---:|
| `s01f05`, `idx = 8`, `Car` | `5,46 m` | `2.661` |
| `s01f05`, `idx = 3`, `Car` | `15,08 m` | `615` |
| `s01f05`, `idx = 6`, `Car` | `26,42 m` | `178` |
| `s01f05`, `idx = 5`, `Car` | `46,80 m` | `51` |

- Quy luật mật độ và đối chiếu slide: Từ bảng trên, khi khoảng cách xa gấp khoảng `2,8 lần` (từ `5,46 m` lên `15,08 m`) thì số điểm chỉ còn khoảng `23%` (giảm `~4,3 lần`), và khi xa gấp khoảng `3,1 lần` (từ `15,08 m` lên `46,80 m`) thì số điểm chỉ còn khoảng `8,3%` (giảm `~12,1 lần`); nói chung xa gấp `2 lần` thì số điểm còn khoảng `1/4` (tỉ lệ nghịch gần với bình phương khoảng cách). Xu hướng này khớp rất sát đường cong trung vị cho xe con ở slide bài giảng (`~1.968` điểm ở `0–10 m`, `~718` ở `10–20 m`, `~214` ở `20–30 m`, `~72` ở `30–40 m`, `~19` ở `60–80 m`).
- Tự kiểm (`preview` và `selfcheck`): Sau khi chạy `sync`, tôi đã kiểm tra ảnh chiếu 7 camera trong `work/xem/` cùng phiếu tự kiểm và sửa trực tiếp trên CVAT (rồi Save và chạy lại `sync`):
  1. Bổ sung và căn chỉnh đủ 5 cuboid cho frame cuối `s05f23` (`idx = 2, 3, 4, 5`, từ bản lưu trước chỉ có 1 cuboid `idx = 1`) bảo đảm mọi frame đều đạt tối thiểu 5 cuboid với `ground_gap_m` chỉ từ `−0,02 m` đến `0,06 m`.
  2. Chỉnh lại độ cao đáy (`Side`/`Front`) và kích thước/góc `yaw` cho các cuboid có sai lệch mép hộp khi xem trên `preview` (như nhóm xe gần ở `s01f05` và `s04f20`), đưa tổng số cảnh báo `selfcheck` về `0/66` cuboid.
  - Cảnh báo giữ nguyên: Không còn cờ nào trong `submission/D13-2A202603253-tukiem.csv` (`0/66` cuboid bị cờ). *(Lưu ý: `s01f05#1` có `ground_gap_m = 0,39 m` chưa vượt ngưỡng `0,45 m` nhưng vẫn cần theo dõi như đã ghi ở bảng `z_bottom` trên.)*
- Đổi kiểm với bạn bên cạnh: Đã đổi kiểm với bạn có MSSV `2A202603252` trên màn hình máy tính; đối chiếu 2 con số từ Bảng A (`s05f04#2`: `dist_m = 3,42 m` và `s01f05#5`: `dist_m = 46,80 m`, `points_in = 51` trong phiếu tự kiểm) — kết quả khớp 100% với file trong `submission/`.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- (a) Ý nghĩa từ góc nhìn của mô hình (model):
  - Vì sao nhãn 3D đo bằng mét chứ không phải pixel: Trên ảnh 2D, kích thước theo pixel phụ thuộc vào khoảng cách và ống kính nên không phản ánh kích thước vật lý thực; nhãn 3D theo đơn vị mét trong không gian thực giữ tính bất biến hình học và cung cấp đúng vị trí, khoảng cách, hướng chuyển động để mô hình nhận thức 3D dự đoán quỹ đạo và khoảng cách an toàn cho xe tự lái.
  - Vì sao vật thưa điểm cần quy tắc riêng: Nếu người gán nhãn đoán tùy tiện tâm, kích thước và góc quay cho những cụm điểm quá thưa (< `15–20%` thể tích), dữ liệu huấn luyện sẽ chứa nhãn nhiễu (noisy labels), làm mô hình học sai tương quan giữa mật độ điểm và hình dạng vật thể, dẫn tới giảm độ chính xác định vị hoặc tăng dự đoán giả (false positives) ở tầm xa.
- (c) Điều còn chưa chắc (không chấm):
  - Một số xe con có kích thước hơi hụt chiều dài (`s04f20#5` dài `3,350 m`, `s04f20#7` dài `3,470 m`, `s03f13#3` dài `3,510 m`) hoặc hơi rộng (`s05f04#2` và `s01f05#4` rộng `2,300 m`) so với Bảng 4.3, chưa chắc chắn hoàn toàn đó là xe cỡ nhỏ/SUV thân rộng hay do biên hộp cần nới/thu nhẹ thêm.
  - Hai cuboid `Cyclist` (`s02f10#2` dài `1,270 m` và `s04f20#1` dài `1,290 m`) ngắn hơn mốc `1,6 m` của Bảng 4.3; cần kiểm tra lại góc nhìn Top để chắc chắn không cắt mất phần bánh xe.
  - Toàn bộ `66` cuboid đang để `occluded = 0`; với một số xe nằm sau xe khác ở tầm `15–30 m` (như `s01f05#1` chỉ có `257` điểm và `ground_gap_m = 0,39 m`), tôi còn chưa chắc liệu đã cần bật `occluded = 1` và hạ đáy thêm khoảng `0,2–0,3 m` hay không.

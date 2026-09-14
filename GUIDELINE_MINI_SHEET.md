# Phiếu quy tắc gán nhãn — Ngày 2

## Thông tin

- **Họ tên:** Nguyễn Xuân Việt Anh
- **MSSV:** 2A202602102
- **Hình thức:** Cá nhân
- **Mã cặp:** SOLO

## Phạm vi gán nhãn

- Chỉ gán nhãn 4 lớp phương tiện:
  - `0 car`
  - `1 truck`
  - `2 bus`
  - `3 van`
- Mỗi phương tiện chỉ có **một bounding box**.
- Loại trừ:
  - Người
  - Xe máy
  - Xe đạp
  - Biển báo
  - Phản chiếu
- Với vật thể quá nhỏ, quá mờ hoặc không đủ căn cứ để phân loại: **không đoán**, ghi nhận lý do và xử lý theo trạng thái `needs_review`.

## Các lớp cố định

| ID | Class | Quy tắc |
|---|---|---|
| `0` | `car` | Sedan, hatchback, SUV, taxi, pickup dùng như xe chở người; không phải truck, bus, van. |
| `1` | `truck` | Có thùng/ben/cargo hoặc thiết bị chở hàng; không phải car, bus hoặc van kín một khối. |
| `2` | `bus` | Thân xe dài, phục vụ chở khách, có nhiều cửa sổ/chỗ ngồi; không phải van nhỏ, truck hoặc car. |
| `3` | `van` | Thân xe nhỏ, kín, dạng hộp để chở người hoặc hàng; không phải bus hoặc truck có khoang hàng riêng. |

**Thứ tự class cố định:** `0 car, 1 truck, 2 bus, 3 van`.

## Quy tắc bounding box

- Bounding box phải **bám sát phần nhìn thấy của vật thể**.
- Không ước lượng phần bị che khuất.
- Nếu vật thể chạm/cắt mép ảnh nhưng vẫn đủ bằng chứng thì vẫn có thể gán nhãn.
- Không đưa quá nhiều nền hoặc nhiều phương tiện khác vào cùng một box.
- Với phương tiện bị che khuất: chỉ vẽ phần nhìn thấy.
- Với phương tiện bị cắt bởi mép ảnh: chỉ vẽ phần nhìn thấy và đánh dấu `boundary=truncated`.

## Thuộc tính

Mỗi bounding box có 3 thuộc tính:

### `visibility`

- `clear`: nhìn rõ.
- `occluded`: bị che khuất một phần.
- `unclear`: khó quan sát/xác định.

### `boundary`

- `inside`: vật thể nằm trong ảnh.
- `truncated`: vật thể bị cắt bởi mép ảnh.

### `review_state`

- `confident`: quyết định chắc chắn.
- `needs_review`: cần xem lại do vật thể quá nhỏ, quá xa, bị che khuất hoặc không đủ bằng chứng để phân loại chắc chắn.

> YOLO không lưu 3 thuộc tính này; cần sử dụng export CVAT cho các thông tin thuộc tính.

## Ba tình huống cần ghi nhớ

### Tình huống A — Bus lớn ở bên phải `drive_008`

**Quan sát:** Có một xe bus lớn ở phía bên phải ảnh, thân xe dài và có nhiều cửa sổ.

**Quyết định:** Gán nhãn `bus`.

**Lý do:** Các đặc điểm về thân xe dài và nhiều cửa sổ phù hợp với lớp bus. Không gán `van` chỉ vì xe được nhìn ở góc xiên.

### Tình huống B — Xe ben/truck ở khu vực phía trên giữa `drive_008`

**Quan sát:** Xe có cabin và phần thùng hàng/ben tách biệt.

**Quyết định:** Gán nhãn `truck`.

**Lý do:** Có cabin kết hợp với khoang/thùng chở hàng riêng, phù hợp với định nghĩa `truck`.

### Tình huống C — Phương tiện nhỏ, xa hoặc bị che khuất

**Quan sát:** Một số phương tiện ở xa, rất nhỏ hoặc bị che khuất khiến việc phân biệt `car`, `van`, `bus`, `truck` không chắc chắn.

**Quyết định:** Không đoán lớp. Sử dụng `review_state=needs_review` khi không đủ bằng chứng; nếu bị che khuất thì đánh dấu `visibility=occluded`, nếu bị cắt bởi mép ảnh thì đánh dấu `boundary=truncated`.

**Lý do:** Ưu tiên tính nhất quán và khả năng kiểm tra lại thay vì cố gán lớp khi không có đủ bằng chứng.

## Checklist trước khi nộp

- [x] Đã review 4 ảnh: `drive_022`, `drive_033`, `drive_038`, `drive_008`.
- [x] Đã kiểm tra vật thể bị thiếu hoặc bị trùng.
- [x] Đã kiểm tra class và hình học bounding box.
- [x] Mỗi box đã có đủ 3 thuộc tính: `visibility`, `boundary`, `review_state`.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành 3 tình huống trước khi xem reference.
- [x] Đã kiểm tra công việc cá nhân trước khi nhận reference.
- [x] Đã ghi nhận tổng số vật thể thực tế: **96**.

## Ghi chú về số lượng

Tổng số vật thể thực tế trong 4 ảnh là **96**, vượt mục tiêu khối lượng 40–60 vật thể.

Mục tiêu 40–60 là **mục tiêu về khối lượng công việc, không phải giới hạn cắt bỏ**. Vì vậy không tự ý thêm hoặc xóa bounding box chỉ để đưa số lượng về 40–60.

## Phân bố vật thể

| Image | Tổng | car | truck | bus | van |
|---|---:|---:|---:|---:|---:|
| `drive_022` | 5 | 3 | 1 | 1 | 0 |
| `drive_033` | 31 | 28 | 1 | 2 | 0 |
| `drive_038` | 34 | 23 | 0 | 6 | 5 |
| `drive_008` | 26 | 19 | 1 | 3 | 3 |
| **Tổng** | **96** | **73** | **3** | **12** | **8** |

## Lưu ý

- Công việc được thực hiện độc lập trước khi xem reference.
- Reference chỉ được sử dụng sau khi phần gán nhãn cá nhân đã được khóa.
- Các trường hợp khác biệt giữa annotation cá nhân và reference cần được xem xét dựa trên ảnh gốc, class và hình học bounding box; không mặc định annotation khác reference là sai.

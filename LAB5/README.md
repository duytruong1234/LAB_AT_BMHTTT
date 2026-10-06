# Lab 5: Thiết lập mô hình tường lửa pfSense

## 1. Họ và tên

Nguyễn Duy Trường

## 2. MSSV

1150070048

## 3. Tên Lab

Thiết lập mô hình tường lửa pfSense.

## 4. Phiên bản môi trường thực hành

- VMware Workstation Pro 26H1u1.
- pfSense CE 2.7.2 (64-bit): RAM 2 GB, 2 vCPU, ổ đĩa 20 GB và 3 card mạng WAN, LAN, DMZ.
- Windows Server 2025 Evaluation làm Domain Controller.
- Windows Server 2025 Evaluation cài IIS làm máy chủ DMZ-Web.
- Ubuntu Server 26.04.1 làm máy LAN-Test.

## 5. Cách dựng môi trường

Em tạo các máy ảo trên VMware Workstation Pro. Card WAN của pfSense dùng VMnet0 ở chế độ Bridged, LAN dùng VMnet1 ở chế độ Host-only và DMZ dùng LAN segment dmz-net.

| Thiết bị / Interface | Địa chỉ IP | Gateway |
|---|---|---|
| pfSense LAN | 10.0.0.1/8 | Không đặt |
| pfSense DMZ | 172.16.0.1/16 | Không đặt |
| Domain Controller | 10.0.0.2/8 | 10.0.0.1 |
| LAN-Test | 10.0.0.3/8 | 10.0.0.1 |
| DMZ-Web | 172.16.0.2/16 | 172.16.0.1 |
| Card VMnet1 của máy thật | 10.0.0.100/8 | Không đặt |

Card VMnet1 của máy thật không đặt DNS. Em cấu hình NAT, các rule trên pfSense và cài IIS trên DMZ-Web để kiểm thử truy cập web.

## 6. Các tình huống thực hành

| Tình huống | Nội dung |
|---|---|
| 1 | Chặn ping nhưng vẫn cho phép DNS và truy cập web. |
| 2 | Chỉ cho máy 10.0.0.2 ra Internet, chặn máy LAN-Test. |
| 3 | Chặn DMZ truy cập LAN nhưng vẫn cho DMZ ra Internet. |
| 4 | Chuyển tiếp cổng WAN 8080 đến DMZ-Web cổng 80. |
| 5 | Bật logging và kiểm tra các gói tin bị chặn. |

## 7. Kết quả PASS/FAIL

| Tình huống | Kết quả thực tế | Đánh giá |
|---|---|---|
| 1 | Ping bị chặn, DNS và HTTPS vẫn hoạt động. | PASS |
| 2 | Máy 10.0.0.2 ping ra Internet thành công, máy 10.0.0.3 bị chặn. | PASS |
| 3 | DMZ không ping được tới LAN nhưng vẫn ping ra Internet thành công. | PASS |
| 4 | Truy cập được trang IIS qua http://192.168.1.4:8080. | PASS |
| 5 | Log ghi nhận gói ICMP từ 172.16.0.2 đến 10.0.0.2 bị chặn. | PASS |

## 8. Lỗi gặp phải và cách khắc phục

- Các mục trong Virtual Network Editor bị mờ: Em chọn Change Settings và cấp quyền quản trị để chỉnh sửa.
- Card LAN của pfSense ban đầu chọn nhầm VMnet0: Em đổi sang VMnet1 để kết nối đúng mạng LAN.
- Gõ sai tham số -Server khi kiểm tra DNS: Em sửa lại lệnh và chạy lại thành công.

# Lab 6: Xác thực Active Directory và chính sách truy cập trên pfSense

## 1. Họ và tên

Nguyễn Duy Trường

## 2. MSSV

1150070048

## 3. Tên Lab

Xác thực Active Directory và chính sách truy cập trên pfSense.

## 4. Phiên bản môi trường thực hành

- VMware Workstation Pro 26H1u1.
- pfSense CE: bổ sung phiên bản sau khi cài đặt.
- Windows Server trên máy DC01: bổ sung phiên bản thực tế.
- Windows 10 trên Client-A và Client-B.
- Ubuntu Server trên DMZ-WEB: bổ sung phiên bản thực tế.
- Nginx làm máy chủ web trên DMZ-WEB.

## 5. Cách dựng môi trường

Triển khai môi trường LAB 6 riêng trên VMware Workstation Pro. Máy pfSense-LAB6 sử dụng bốn card mạng: WAN, MGMT, LAN và DMZ.

WAN kết nối VMnet8 ở chế độ NAT, sử dụng mạng 192.168.250.0/24. MGMT kết nối VMnet2 ở chế độ Host-only, sử dụng mạng 192.168.56.0/24. LAN sử dụng LAN Segment lab6-lan; DMZ sử dụng LAN Segment lab6-dmz.

Cấu hình địa chỉ IP theo bảng:

| Thiết bị / Interface | Địa chỉ IP | Gateway | DNS |
|---|---|---|---|
| pfSense WAN | DHCP trên 192.168.250.0/24 | Nhận tự động | Theo cấu hình WAN |
| pfSense MGMT | 192.168.56.254/24 | Không đặt | Không đặt |
| Card VMnet2 trên máy thật | 192.168.56.1/24 | Không đặt | Không đặt |
| pfSense LAN | 10.6.0.1/24 | Không đặt | Không đặt |
| DC01 | 10.6.0.10/24 | 10.6.0.1 | 10.6.0.10 |
| Client-A | 10.6.0.21/24 | 10.6.0.1 | 10.6.0.10 |
| Client-B | 10.6.0.22/24 | 10.6.0.1 | 10.6.0.10 |
| pfSense DMZ | 172.16.6.1/24 | Không đặt | Không đặt |
| DMZ-WEB | 172.16.6.10/24 | 172.16.6.1 | DNS upstream phù hợp |

DC01 cung cấp dịch vụ Active Directory và DNS cho domain lab6.test. Tạo OU LAB6-Users, hai tài khoản sv_a, sv_b và tài khoản svc_pfsense_ldap dùng để kết nối LDAP.

pfSense xác thực tài khoản AD thông qua LDAP. Captive Portal được cấu hình trên LAN; giao diện quản trị được truy cập riêng qua MGMT tại https://192.168.56.254.

DMZ-WEB sử dụng Nginx và được gán thêm các IP để mô phỏng website:

| Alias | Địa chỉ IP | Mục đích |
|---|---|---|
| GENERAL_SITE | 172.16.6.10 | Website thông thường |
| JOB_SITE | 172.16.6.20 | Website việc làm mô phỏng |
| SOCIAL_SITE | 172.16.6.30 | Website mạng xã hội mô phỏng |
| RESTRICTED_SITE | 172.16.6.40 | Website có nội dung bị hạn chế mô phỏng |

Tạo các Alias CLIENT_A, CLIENT_B, POLICY_CLIENTS và WEB_PORTS. Tạo Schedule LUNCH_WINDOW từ 11:30–13:00 và WORK_HOURS từ 08:00–17:00, từ Thứ 2 đến Thứ 6. TEST_NOW được dùng để kiểm thử nhanh.

Các chính sách firewall áp dụng theo IP, cổng và thời gian. Tài khoản LDAP được sử dụng để xác thực qua Captive Portal.

## 6. Các tình huống thực hành

| Tình huống | Nội dung |
|---|---|
| 1 | Chặn Client-A truy cập JOB_SITE nhưng vẫn cho phép truy cập GENERAL_SITE. |
| 2 | Chặn cả Client-A và Client-B truy cập RESTRICTED_SITE. |
| 3 | Chỉ cho Client-B truy cập SOCIAL_SITE trong giờ nghỉ. |
| 4 | Chỉ cho phép HTTP/HTTPS trong giờ hành chính. |
| 5 | Chặn DMZ khởi tạo kết nối vào LAN nhưng vẫn cho LAN truy cập website cần thiết trong DMZ. |

## 7. Kết quả PASS/FAIL

| Nội dung | Kết quả thực tế | Đánh giá |
|---|---|---|
| Xác thực LDAP với sv_a và sv_b | | |
| Đăng nhập Captive Portal | | |
| Tình huống 1: JOB_SITE | | |
| Tình huống 2: RESTRICTED_SITE | | |
| Tình huống 3: SOCIAL_SITE theo Schedule | | |
| Tình huống 4: HTTP/HTTPS trong giờ hành chính | | |
| Tình huống 5: Cô lập DMZ khỏi LAN | | |

## 8. Lỗi gặp phải và cách khắc phục

Cập nhật sau khi thực hành.

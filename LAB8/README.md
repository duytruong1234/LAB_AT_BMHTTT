# Lab 8: Local DNS Attack - DNS Spoofing và Cache Poisoning trong mạng ảo cô lập

## 1. Họ và tên

Nguyễn Duy Trường

## 2. MSSV

1150070048

## 3. Tên Lab

Local DNS Attack - DNS Spoofing và Cache Poisoning trong mạng ảo cô lập trên môi trường pfSense.

## 4. Phiên bản môi trường thực hành

- VMware Workstation Pro 26H1u1 / Oracle VirtualBox 7.2 (hỗ trợ Internal Network / LAN Segment `dns-lab` và chế độ Promiscuous Mode).
- pfSense CE: 2.7.2-RELEASE (kế thừa từ Lab trước, bổ sung giao diện `DNSLAB` 10.9.0.1/24).
- Ubuntu Server trên DNS-SERVER (10.9.0.10): Ubuntu Server 24.04 LTS (chạy dịch vụ ISC BIND9 9.18.x làm DNS Recursive/Caching Resolver và Authoritative cho `lab9.test`).
- Ubuntu trên máy trạm VICTIM (10.9.0.100): Ubuntu 24.04 LTS Desktop/Server (sử dụng `systemd-resolved`, `netplan`, `dig`, `curl`, `getent`).
- Kali Linux trên máy ATTACKER (10.9.0.200): Kali Linux 2024.x / 2026.x (chạy Python 3.11+, thư viện `python3-scapy`, `Wireshark`, `tcpdump` và `http.server` cổng 80).
- Ubuntu Server trên APP-TEST / AUTH-DNS (172.16.6.20): Ubuntu Server 24.04 LTS (chạy trình mô phỏng Authoritative DNS `auth_dns_delayed.py` với độ trễ phản hồi 1.5s).
- Ubuntu Server trên DMZ-WEB (172.16.6.10): Ubuntu Server 24.04 LTS (chạy Nginx Web Server cung cấp trang web hợp lệ trong vùng DMZ).

## 5. Cách dựng môi trường

Triển khai môi trường LAB kế thừa hạ tầng tường lửa pfSense và vùng DMZ từ các bài thực hành trước, đồng thời bổ sung một phân đoạn mạng cô lập riêng tên `DNSLAB` (`10.9.0.0/24`) để thực hiện tấn công giả mạo DNS (DNS Pharming, Direct DNS Spoofing, DNS Cache Poisoning) mà không làm ảnh hưởng đến hệ thống `LAN-DOMAIN` (`10.6.0.0/24`) hiện có.

Mô hình gồm các phân vùng mạng chính:
- WAN: Kết nối ở chế độ NAT (192.168.250.0/24), bị chặn hoàn toàn đối với nhánh DNSLAB trong pha tấn công.
- MGMT: Kết nối ở chế độ Host-only (192.168.56.0/24), quản trị pfSense WebGUI tại https://192.168.56.254.
- DMZ: Phân vùng máy chủ (172.16.6.0/24) chứa máy chủ web hợp lệ DMZ-WEB (172.16.6.10) và máy chủ DNS có thẩm quyền mô phỏng APP-TEST / AUTH-DNS (172.16.6.20).
- DNSLAB: Phân vùng mạng nội bộ cô lập `dns-lab` (10.9.0.0/24) chứa DNS-SERVER (10.9.0.10), máy nạn nhân VICTIM (10.9.0.100) và máy tấn công ATTACKER - Kali (10.9.0.200).

Cấu hình địa chỉ IP chi tiết theo bảng:

| Thiết bị / Interface | Địa chỉ IP | Subnet Mask | Gateway | DNS | Vai trò / Ghi chú |
|---|---|---|---|---|---|
| pfSense WAN | 192.168.250.x (DHCP) | 255.255.255.0 | Nhận tự động | DNS upstream | Kết nối ngoài (cô lập với DNSLAB) |
| pfSense MGMT | 192.168.56.254 | 255.255.255.0 | Không đặt | Không đặt | Cổng quản trị pfSense WebGUI |
| Host MGMT (Máy thật) | 192.168.56.1 | 255.255.255.0 | Không đặt | Không đặt | Truy cập WebGUI từ máy vật lý |
| pfSense DMZ | 172.16.6.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway vùng máy chủ DMZ |
| pfSense DNSLAB (OPT2) | 10.9.0.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway mạng cô lập DNS-LAB |
| DNS-SERVER | 10.9.0.10 | 255.255.255.0 | 10.9.0.1 | 127.0.0.1 | BIND9 Master `lab9.test`, Forwarder & Cache |
| VICTIM | 10.9.0.100 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Máy trạm nạn nhân truy vấn DNS và HTTP |
| ATTACKER (Kali) | 10.9.0.200 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Sniff gói DNS (Scapy/Wireshark) & Fake Web (80) |
| DMZ-WEB | 172.16.6.10 | 255.255.255.0 | 172.16.6.1 | Không đặt | Web Server hợp lệ đích của tên miền |
| APP-TEST / AUTH-DNS | 172.16.6.20 | 255.255.255.0 | 172.16.6.1 | Không đặt | Chạy `auth_dns_delayed.py` (UDP 53, delay 1.5s) |

Nguyên lý vận hành luồng DNS, cô lập mạng và cơ chế tấn công trên môi trường Lab:
1. Chính sách cô lập trên pfSense (DNSLAB Ruleset): Tường lửa pfSense chỉ mở 2 luồng hợp lệ từ nhánh DNSLAB: (Rule 1) Cho phép duy nhất DNS-SERVER (10.9.0.10) truy vấn UDP/TCP 53 tới AUTH-DNS (172.16.6.20); (Rule 2) Cho phép mạng 10.9.0.0/24 kết nối TCP 80 tới DMZ-WEB (172.16.6.10). Các Rule 3, 4, 5 chặn và ghi log (Block + Log) toàn bộ kết nối sang LAN-DOMAIN (10.6.0.0/24), MGMT (192.168.56.0/24) và Any (chặn ra Internet/WAN).
2. Luồng phân giải DNS hợp lệ (Baseline): Máy DNS-SERVER (10.9.0.10) chạy BIND9 nắm quyền authoritative cho zone nội bộ `lab9.test` (`www.lab9.test -> 172.16.6.10`) và cấu hình chuyển tiếp (`type forward`) đối với zone `external.lab9.test` về 172.16.6.20. Khi VICTIM hỏi `www.external.lab9.test`, DNS-SERVER hỏi tiếp 172.16.6.20 và nhận câu trả lời hợp lệ 172.16.6.10 sau 1.5 giây.
3. Cơ chế bắt gói và chạy đua phản hồi (DNS Race Condition): Card mạng `dns-lab` của máy Kali (10.9.0.200) được bật chế độ `Promiscuous Mode = Allow VMs` để bắt được các gói tin UDP unicast đi qua switch ảo. Ngay khi phát hiện gói truy vấn DNS cho `www.external.lab9.test`, script `dns_spoof_lab9.py` (dùng Scapy) trích xuất chính xác Transaction ID và cổng nguồn UDP sport của gói truy vấn, sau đó gửi trả ngay lập tức gói phản hồi giả mạo chứa bản ghi `A = 10.9.0.200` (trước khi gói hợp lệ từ 172.16.6.20 kịp phản hồi sau 1.5s).

## 6. Các tình huống thực hành

| Tình huống | Tên kịch bản | Nội dung thực hiện |
|---|---|---|
| 1 | Cô lập mạng DNSLAB & Kiểm tra Baseline | Cấu hình interface DNSLAB (10.9.0.1/24) và 5 rule cô lập trên pfSense; cấu hình BIND9 trên DNS-SERVER (forward/reverse zone và forwarder); khởi chạy `auth_dns_delayed.py` trên APP-TEST và xác nhận `www.external.lab9.test` phân giải đúng về 172.16.6.10. |
| 2 | Task 1 - Thay đổi tệp `/etc/hosts` cục bộ (Local Hosts Redirection) | Trên máy VICTIM, chèn dòng `10.9.0.200 www.lab9.test` vào `/etc/hosts`; kiểm tra sự khác biệt giữa công cụ phân giải qua thư viện hệ thống (`getent`, `curl` bị chuyển hướng về trang web giả 10.9.0.200) và công cụ truy vấn DNS trực tiếp (`dig @10.9.0.10` vẫn trả về 172.16.6.10). |
| 3 | Task 2 - Giả mạo DNS trực tiếp tới Victim (Direct DNS Spoofing) | Xóa cache DNS; chạy `dns_spoof_lab9.py --mode victim` trên Kali. Khi VICTIM gửi truy vấn `www.external.lab9.test` tới 10.9.0.10, Kali giả mạo IP nguồn 10.9.0.10 trả lời trực tiếp cho VICTIM địa chỉ 10.9.0.200 thắng cuộc đua thời gian trước gói trả lời thật. |
| 4 | Task 3 - Đầu độc bộ nhớ đệm DNS Resolver (DNS Cache Poisoning) | Xóa cache BIND9; chạy `dns_spoof_lab9.py --mode resolver` trên Kali. Khi DNS-SERVER chuyển tiếp truy vấn từ 10.9.0.10 tới 172.16.6.20, Kali giả mạo IP nguồn 172.16.6.20 trả lời `A = 10.9.0.200` (TTL=120s) cho DNS-SERVER. Kiểm tra tệp `named_dump.db` xác nhận cache của BIND9 đã bị nhiễm bản ghi giả. |
| 5 | Khôi phục môi trường & Phòng thủ (Restore & Hardening) | Dừng script tấn công, thực hiện `sudo rndc flush` trên DNS-SERVER và `sudo resolvectl flush-caches` trên VICTIM, xác nhận tên miền phân giải lại về 172.16.6.10. Bật lại cơ chế bảo vệ DNS COOKIE trên BIND9 và trả Promiscuous Mode của Kali về `Deny`. |

## 7. Kết quả PASS/FAIL

| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
|---|---|---|
| Cấu hình pfSense DNSLAB (10.9.0.1/24) & 5 Firewall Rules | Interface hoạt động tốt, cho phép đúng DNS (port 53) tới 172.16.6.20 và HTTP (port 80) tới 172.16.6.10, chặn hoàn toàn truy cập sang LAN-DOMAIN, MGMT và WAN. |  |
| Cấu hình BIND9 & Kiểm tra cú pháp trên DNS-SERVER | Lệnh `named-checkconf` và `named-checkzone` cho cả 2 zone `lab9.test` và `9.10.in-addr.arpa` đều báo OK; dịch vụ `bind9` lắng nghe tại 10.9.0.10:53. |  |
| Kiểm tra Baseline trước tấn công | Lệnh `dig @10.9.0.10 www.external.lab9.test` trả về đúng 172.16.6.10; lệnh `curl http://www.external.lab9.test` hiển thị nội dung web hợp lệ của DMZ-WEB. |  |
| Task 1: Chuyển hướng bằng `/etc/hosts` trên Victim | Lệnh `getent hosts www.lab9.test` và `curl http://www.lab9.test` trỏ về 10.9.0.200 (trang giả lập trên Kali), trong khi `dig @10.9.0.10 www.lab9.test` vẫn trả về 172.16.6.10. |  |
| Task 2: Direct DNS Spoofing (`--mode victim`) | Wireshark ghi nhận gói Forged Response (10.9.0.10 -> 10.9.0.100, khớp Transaction ID và UDP port) đến trước gói thật; VICTIM nhận IP 10.9.0.200 và hiển thị trang web mô phỏng của Kali. |  |
| Task 3: DNS Cache Poisoning (`--mode resolver`) | Kali gửi gói giả mạo nguồn 172.16.6.20 -> 10.9.0.10; kiểm tra dump cache `/var/cache/bind/named_dump.db` bằng `rndc dumpdb -cache` ghi nhận bản ghi `www.external.lab9.test` trỏ về 10.9.0.200. |  |
| Khôi phục & Hardening sau thực hành | Sau khi flush cache trên BIND9 và Victim, truy vấn `www.external.lab9.test` trở về 172.16.6.10; tắt Promiscuous Mode trên Kali về `Deny` và bật lại DNS COOKIE mặc định. |  |

## 8. Lỗi gặp phải và cách khắc phục

# Lab 8: Local DNS Attack - DNS Spoofing và Cache Poisoning trong mạng ảo cô lập

## 1. Họ và tên

Nguyễn Duy Trường

## 2. MSSV

1150070048

## 3. Tên Lab

Local DNS Attack - DNS Spoofing và Cache Poisoning trong mạng ảo cô lập trên môi trường pfSense.

## 4. Phiên bản môi trường thực hành

- VMware Workstation Pro (tạo LAN Segment `dns-lab` cô lập và bật Promiscuous Mode).
- pfSense CE 2.7.2-RELEASE (kế thừa từ bài trước, thêm cổng `DNSLAB` 10.9.0.1/24).
- Máy DNS-SERVER (10.9.0.10): Ubuntu Server chạy dịch vụ BIND9 (quản lý zone `lab9.test` và chuyển tiếp `external.lab9.test`).
- Máy VICTIM (10.9.0.100): Máy trạm nạn nhân (Windows 10 / Ubuntu Client) trỏ DNS duy nhất về 10.9.0.10.
- Máy ATTACKER (10.9.0.200): Kali Linux 2026.2 chạy Python Scapy (`dns_spoof_lab9.py`), Wireshark và Web Server giả ở cổng 80.
- Cụm máy DMZ: DMZ-WEB (172.16.6.10) chạy trang web thật và APP-TEST / AUTH-DNS (172.16.6.20) chạy script `auth_dns_delayed.py` (trả lời DNS chậm 1.5 giây).

## 5. Cách dựng môi trường

Bài lab được dựng tiếp nối từ hệ thống pfSense của các bài trước. Để thực hành tấn công DNS an toàn mà không làm hỏng mạng LAN cũ (10.6.0.0/24), mình tạo thêm một nhánh mạng riêng biệt tên là `DNSLAB` (10.9.0.0/24) thông qua LAN Segment `dns-lab` trên VMware.

Các nhánh mạng trong bài gồm:
- WAN (192.168.250.0/24 - NAT): Đường ra Internet, bị chặn đối với nhánh DNSLAB trong lúc tấn công.
- MGMT (192.168.56.0/24 - Host-only): Dùng để vào trang quản trị pfSense WebGUI (https://192.168.56.254).
- DMZ (172.16.6.0/24): Đặt Web Server hợp lệ (172.16.6.10) và DNS Server có thẩm quyền mô phỏng (172.16.6.20).
- DNSLAB (10.9.0.0/24 - LAN Segment `dns-lab`): Gồm 3 máy DNS-SERVER (10.9.0.10), VICTIM (10.9.0.100) và Kali ATTACKER (10.9.0.200) nối chung một switch ảo.

Bảng phân chia IP:

| Thiết bị / Interface | Địa chỉ IP | Subnet Mask | Gateway | DNS | Vai trò / Ghi chú |
|---|---|---|---|---|---|
| pfSense WAN | 192.168.250.x (DHCP) | 255.255.255.0 | Tự động | Tự động | Đường ra ngoài (chặn với DNSLAB) |
| pfSense MGMT | 192.168.56.254 | 255.255.255.0 | Không đặt | Không đặt | Cổng vào WebGUI pfSense |
| pfSense DMZ | 172.16.6.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway nhánh DMZ |
| pfSense DNSLAB | 10.9.0.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway nhánh cô lập DNSLAB |
| DNS-SERVER | 10.9.0.10 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Chạy BIND9 phân giải và lưu cache |
| VICTIM | 10.9.0.100 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Máy nạn nhân truy vấn DNS và duyệt web |
| ATTACKER (Kali) | 10.9.0.200 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Nghe lén gói DNS và dựng web giả (cổng 80) |
| DMZ-WEB | 172.16.6.10 | 255.255.255.0 | 172.16.6.1 | Không đặt | Trang web thật của bài lab |
| AUTH-DNS | 172.16.6.20 | 255.255.255.0 | 172.16.6.1 | Không đặt | DNS gốc trả lời chậm 1.5s (`auth_dns_delayed.py`) |

Nguyên lý hoạt động trong bài:
1. Cô lập bằng pfSense: Trên tab `DNSLAB`, mình tạo 5 rule: chỉ mở cho DNS-SERVER (10.9.0.10) hỏi DNS tới 172.16.6.20 (cổng 53) và cho các máy trong mạng 10.9.0.0/24 vào web 172.16.6.10 (cổng 80). Toàn bộ truy cập sang LAN cũ (10.6.0.0/24), MGMT (192.168.56.0/24) và ra ngoài Internet đều bị chặn và ghi log.
2. Khi hoạt động bình thường (Baseline): VICTIM hỏi `www.external.lab9.test` -> DNS-SERVER (10.9.0.10) hỏi tiếp sang AUTH-DNS (172.16.6.20) -> sau 1.5 giây sẽ nhận được IP thật là `172.16.6.10`.
3. Khi Kali tấn công: Nhờ bật Promiscuous Mode trên card `eth0`, Kali nhìn thấy ngay gói hỏi DNS đi qua mạng. Script `dns_spoof_lab9.py` lập tức lấy đúng mã `Transaction ID` và cổng nguồn UDP của gói hỏi để bắn trả gói DNS giả (trỏ về IP `10.9.0.200` của Kali) chỉ trong vài ms. Vì gói giả tới trước gói thật (bị trễ 1.5s), máy nhận sẽ tin gói giả và bỏ gói thật đến sau.

## 6. Các tình huống thực hành

| Tình huống | Tên kịch bản | Nội dung thực hiện |
|---|---|---|
| 1 | Dựng mạng & Kiểm tra Baseline | Tạo cổng `DNSLAB` và 5 rule trên pfSense; cấu hình BIND9 trên DNS-SERVER; bật `auth_dns_delayed.py` ở 172.16.6.20 và kiểm tra trên VICTIM thấy `www.external.lab9.test` phân giải đúng về `172.16.6.10` (vào được web thật DMZ-WEB). |
| 2 | Task 1 - Sửa file `/etc/hosts` trên Victim | Chèn dòng `10.9.0.200 www.lab9.test` vào `/etc/hosts` của VICTIM. Kiểm tra bằng `getent` và `curl` thì bị dẫn về web giả của Kali (`10.9.0.200`), còn dùng `dig @10.9.0.10` thì vẫn ra IP thật `172.16.6.10` vì `dig` hỏi thẳng DNS Server chứ không đọc file `hosts`. Sau đó khôi phục lại file `hosts` gốc. |
| 3 | Task 2 - Giả mạo DNS trực tiếp tới Victim (`--mode victim`) | Xóa sạch cache DNS; bật Wireshark và chạy `dns_spoof_lab9.py --mode victim --iface eth0` trên Kali. Khi VICTIM hỏi tên miền, Kali giả danh `10.9.0.10` trả lời thẳng cho VICTIM địa chỉ `10.9.0.200` (chỉ mất 12.8ms), làm `curl` mở ra trang web giả. |
| 4 | Task 3 - Đầu độc cache của DNS Server (`--mode resolver`) | Xóa cache BIND9; chạy `dns_spoof_lab9.py --mode resolver --iface eth0` trên Kali. Khi DNS-SERVER (10.9.0.10) hỏi lên 172.16.6.20, Kali giả danh `172.16.6.20` trả lời IP `10.9.0.200` về cho DNS-SERVER. Xuất file cache (`rndc dumpdb -cache`) và kiểm tra `/var/cache/bind/named_dump.db` thấy bản ghi giả `10.9.0.200` đã lưu trong cache của BIND9. |
| 5 | Khôi phục & Phòng thủ sau khi test | Tắt tool spoof trên Kali, xóa cache trên cả DNS-SERVER (`rndc flush`) và VICTIM (`resolvectl flush-caches`), kiểm tra lại thấy tên miền đã về lại IP thật `172.16.6.10`. Cuối cùng tắt chế độ nghe lén trên Kali (`ip link set eth0 promisc off`). |

## 7. Kết quả PASS/FAIL

| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
|---|---|---|
| Cấu hình pfSense DNSLAB & 5 Firewall Rules | Nhánh DNSLAB thông mạng nội bộ, chỉ đi được cổng 53 tới 172.16.6.20 và cổng 80 tới 172.16.6.10, chặn hoàn toàn sang LAN cũ, MGMT và WAN. | **PASS** |
| Cấu hình BIND9 trên DNS-SERVER | Lệnh `named-checkconf` và `named-checkzone` đều báo OK; dịch vụ BIND9 chạy ổn định ở `10.9.0.10:53`. | **PASS** |
| Kiểm tra Baseline ban đầu | Truy vấn `www.external.lab9.test` ra đúng IP thật `172.16.6.10`, lệnh `curl` tải đúng giao diện trang DMZ-WEB. | **PASS** |
| Task 1: Chuyển hướng bằng file `/etc/hosts` | `getent` và `curl` bị chuyển hướng về `10.9.0.200` (trang web giả), trong khi `dig @10.9.0.10` vẫn trả về `172.16.6.10`. Khôi phục lại về như cũ thành công. | **PASS** |
| Task 2: Direct DNS Spoofing (`--mode victim`) | VICTIM nhận IP giả `10.9.0.200` và mở ra web giả của Kali; trên Wireshark bắt được rõ gói trả lời giả mạo đến trước gói thật. | **PASS** |
| Task 3: DNS Cache Poisoning (`--mode resolver`) | Dump cache của BIND9 trong `/var/cache/bind/named_dump.db` xuất hiện dòng `www.external.lab9.test. 115 IN A 10.9.0.200`. | **PASS** |
| Khôi phục môi trường & Tắt Promiscuous Mode | Sau khi xóa cache, tên miền phân giải lại về `172.16.6.10`; kiểm tra `ip -d link show eth0` trên Kali đã về `promiscuity 0`. | **PASS** |

## 8. Lỗi gặp phải và cách khắc phục

- **Lỗi 1: Không cài được gói `bind9` và `dnsutils` khi máy DNS-SERVER đã đưa vào mạng `dns-lab`.**
  - *Nguyên nhân:* Do rule trên pfSense đã chặn nhánh `DNSLAB` đi ra Internet nên lệnh `apt install` không tải được gói.
  - *Cách khắc phục:* Tạm chuyển card mạng máy Ubuntu sang chế độ NAT để cài xong `bind9` và `dnsutils`, sau đó chỉnh lại về LAN Segment `dns-lab` và đặt IP tĩnh `10.9.0.10/24`.
- **Lỗi 2: BIND9 báo lỗi không phân giải được miền con `www.external.lab9.test` dù đã cấu hình forward.**
  - *Nguyên nhân:* Vì DNS-SERVER đang giữ quyền quản lý (master) cho miền cha `lab9.test` nên nó tự coi miền con `external.lab9.test` không tồn tại, đồng thời cơ chế `dnssec-validation` mặc định cố kết nối ra Internet làm truy vấn bị treo.
  - *Cách khắc phục:* Khai báo thêm dòng ủy quyền NS cho miền con `external` trong file `/etc/bind/db.lab9.test` và tắt kiểm tra DNSSEC (`dnssec-validation no; edns no;`) trong `named.conf.options`.
- **Lỗi 3: Máy Kali tự động bị mất IP tĩnh `10.9.0.200` sau khoảng 1 phút.**
  - *Nguyên nhân:* Dịch vụ `NetworkManager` trên Kali tự động tìm DHCP Server trên card `eth0`, khi không thấy DHCP trong mạng cô lập thì nó tự xóa IP vừa gán.
  - *Cách khắc phục:* Tắt `NetworkManager` bằng lệnh `sudo systemctl stop NetworkManager` rồi mới đặt IP tĩnh `10.9.0.200/24`.
- **Lỗi 4: Chạy script trên Kali báo lỗi không tìm thấy card mạng hoặc sai tham số mode.**
  - *Nguyên nhân:* Trên VMware tên card mạng của Kali là `eth0` (khác với `eth1` trong tài liệu mẫu trên VirtualBox), và lúc gõ lệnh bị thiếu chữ `r` ở `--mode resolver`.
  - *Cách khắc phục:* Kiểm tra lại tên card bằng `ip -br addr` và chạy đúng lệnh `sudo python3 dns_spoof_lab9.py --mode resolver --iface eth0`.
- **Lỗi 5: Lệnh `rndc dumpdb -cache` không tạo ra file `named_dump.db` hoặc khi `curl` web giả bị lỗi font chữ.**
  - *Nguyên nhân:* Thư mục `/var/cache/bind` thiếu quyền ghi cho user `bind`, còn file HTML web giả có dấu tiếng Việt nên khi `curl` trên terminal bị hiển thị lỗi ký tự.
  - *Cách khắc phục:* Cấp lại quyền `sudo chown bind:bind /var/cache/bind`, và sửa nội dung file `index.html` trên Kali sang tiếng Việt không dấu để hiển thị rõ ràng.

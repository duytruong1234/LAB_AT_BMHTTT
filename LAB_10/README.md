# Lab 10: Remote DNS Cache Poisoning trong môi trường pfSense cô lập

## 1. Họ và tên
Nguyễn Duy Trường

## 2. MSSV
1150070048

## 3. Tên Lab
Remote DNS Cache Poisoning trong môi trường pfSense cô lập.

## 4. Phiên bản môi trường thực hành
- VMware Workstation Pro (tạo 2 LAN Segment cô lập dns-lab và attack-net, tắt Promiscuous Mode trên Kali).
- pfSense CE 2.7.2-RELEASE (kế thừa bài trước, giữ cổng DNSLAB 10.9.0.1/24, DMZ 172.16.6.1/24 và thêm cổng ATTACKNET 10.10.0.1/24).
- Máy DNS-SERVER (10.9.0.10): Ubuntu Server 24.04 chạy BIND9 và resolver mô phỏng vuln_resolver_lab10.py (cố định cổng nguồn UDP 33333 khi tấn công).
- Máy VICTIM (10.9.0.100): Máy trạm Windows 10 trỏ DNS duy nhất về 10.9.0.10.
- Máy ATTACKER (10.10.0.200): Kali Linux đặt ở nhánh attack-net, chạy kaminsky_remote_lab10.py, evil_dns_lab10.py (cổng 53), tcpdump và Web Server giả ở cổng 80.
- Cụm máy DMZ: DMZ-WEB (172.16.6.10) chạy trang web thật và AUTH-DNS/RELAY (172.16.6.20) chạy auth_dns_relay_lab10.py (trả lời chậm 3 giây với tên miền ngẫu nhiên và mở cổng UDP 5300 làm race relay).

## 5. Cách dựng môi trường
Bài lab được dựng tiếp nối từ hệ thống DNS-LAB của bài trước nhưng tách máy tấn công Kali sang một nhánh mạng riêng biệt tên là ATTACKNET (10.10.0.0/24) qua LAN Segment attack-net trên VMware để Kali không thể nghe lén gói tin của DNS-SERVER.

Các nhánh mạng trong bài gồm:
- WAN (192.168.250.0/24 - NAT): Đường ra Internet, bị chặn đối với DNSLAB và ATTACKNET trong lúc tấn công.
- MGMT (192.168.56.0/24 - Host-only): Dùng để vào trang quản trị pfSense WebGUI (https://192.168.56.254).
- DMZ (172.16.6.0/24): Đặt Web Server hợp lệ (172.16.6.10) và DNS Server có thẩm quyền mô phỏng kiêm race relay (172.16.6.20).
- DNSLAB (10.9.0.0/24 - LAN Segment dns-lab): Chỉ gồm 2 máy DNS-SERVER (10.9.0.10) và VICTIM (10.9.0.100).
- ATTACKNET (10.10.0.0/24 - LAN Segment attack-net): Nhánh mạng riêng chứa máy Kali ATTACKER (10.10.0.200), tắt Promiscuous Mode.

Bảng phân chia IP:

| Thiết bị / Interface | Địa chỉ IP | Subnet Mask | Gateway | DNS | Vai trò / Ghi chú |
|---|---|---|---|---|---|
| pfSense WAN | 192.168.250.x (DHCP) | 255.255.255.0 | Tự động | Tự động | Đường ra ngoài (chặn với DNSLAB và ATTACKNET) |
| pfSense MGMT | 192.168.56.254 | 255.255.255.0 | Không đặt | Không đặt | Cổng vào WebGUI pfSense |
| pfSense DMZ | 172.16.6.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway nhánh DMZ |
| pfSense DNSLAB | 10.9.0.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway nhánh DNSLAB |
| pfSense ATTACKNET | 10.10.0.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway nhánh ATTACKNET (mới thêm ở Lab 10) |
| DNS-SERVER | 10.9.0.10 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Chạy BIND9 và vuln_resolver_lab10.py (port 33333) |
| VICTIM | 10.9.0.100 | 255.255.255.0 | 10.9.0.1 | 10.9.0.10 | Máy nạn nhân truy vấn DNS và duyệt web |
| ATTACKER (Kali) | 10.10.0.200 | 255.255.255.0 | 10.10.0.1 | 10.9.0.10 | Đoán TXID từ xa, chạy Evil DNS (53) và Fake Web (80) |
| DMZ-WEB | 172.16.6.10 | 255.255.255.0 | 172.16.6.1 | Không đặt | Trang web thật của bài lab |
| AUTH-DNS / RELAY | 172.16.6.20 | 255.255.255.0 | 172.16.6.1 | Không đặt | DNS gốc trả lời chậm 3s + Race Relay cổng UDP 5300 |

Nguyên lý hoạt động trong bài:
1. Cô lập bằng pfSense: Trên tab DNSLAB chỉ mở cho VICTIM hỏi DNS tới 10.9.0.10, mở cho DNS-SERVER hỏi DNS tới 172.16.6.20 và 10.10.0.200 (cổng 53), mở cổng 80 tới 172.16.6.10 và 10.10.0.200. Trên tab ATTACKNET chỉ mở cho Kali (10.10.0.200) gửi query mồi tới 10.9.0.10 (cổng 53) và gửi lô đoán TXID tới 172.16.6.20 (cổng 5300). Toàn bộ truy cập sang LAN cũ, MGMT và WAN đều bị chặn.
2. Khi hoạt động bình thường (Baseline): VICTIM hỏi www.example.lab10.test -> DNS-SERVER (10.9.0.10) hỏi tiếp sang AUTH-DNS (172.16.6.20) -> nhận được IP thật 172.16.6.10 và vào được web thật DMZ-WEB.
3. Khi Kali tấn công từ xa: Do khác nhánh mạng nên Kali không nghe lén được Transaction ID. Khi DNS-SERVER chạy vuln_resolver_lab10.py (cố định cổng nguồn 33333), Kali gửi truy vấn mồi bằng một tên ngẫu nhiên chưa có trong cache để ép DNS-SERVER hỏi lên AUTH-DNS (bị giữ trễ 3 giây), đồng thời bắn 16 lô đoán toàn bộ 65.536 giá trị TXID qua cổng 5300. Khi trúng TXID, gói giả chứa bản ghi ủy quyền NS (ns.evil.example.lab10.test) kèm Glue A (10.10.0.200) về tới cổng 33333 trước gói thật, làm toàn bộ tên miền example.lab10.test bị chuyển hướng sang hỏi Evil DNS trên Kali.

## 6. Các tình huống thực hành

| Tình huống | Tên kịch bản | Nội dung thực hiện |
|---|---|---|
| 1 | Tách mạng ATTACKNET & Kiểm tra Baseline | Tạo cổng ATTACKNET (10.10.0.1/24) và các rule trên pfSense; chuyển Kali sang attack-net (10.10.0.200/24); bật BIND9 và auth_dns_relay_lab10.py rồi kiểm tra trên VICTIM thấy www.example.lab10.test phân giải đúng về 172.16.6.10. |
| 2 | Chứng minh Kali không sniff được DNSLAB | Chạy tcpdump -ni any udp port 53 trên Kali trong lúc VICTIM truy vấn DNS. Kết quả báo 0 packets captured, chứng minh Kali ở mạng riêng không thể nghe lén được gói tin của DNS-SERVER. |
| 3 | Task 1 - Tấn công Remote Cache Poisoning (Kaminsky) | Dừng BIND9, chạy vuln_resolver_lab10.py (cổng 33333) trên 10.9.0.10. Trên Kali bật evil_dns_lab10.py, web giả và chạy kaminsky_remote_lab10.py. Quan sát log [PENDING], [HIT] trên AUTH-DNS, log [CACHE] trên Resolver và thông báo SUCCESS trên Kali. |
| 4 | Task 2 - Bắt gói phân tích & Kiểm tra trên Victim | Bắt gói tcpdump cổng 33333 trên DNS-SERVER thấy gói hỏi lên 172.16.6.20, gói trả lời giả 0/1/1 và gói hỏi sau khi bị poison đi thẳng sang 10.10.0.200:53. Trên VICTIM truy vấn lại ra IP 10.10.0.200 và mở ra trang LAB 10 - Fake Site. |
| 5 | Khôi phục BIND9 & Kiểm chứng cổng nguồn ngẫu nhiên | Dừng resolver mô phỏng, bật lại BIND9 và xóa cache (rndc flush). Bắt gói tcpdump khi VICTIM truy vấn thấy www.example.lab10.test đã về lại 172.16.6.10 và cổng nguồn UDP của BIND9 thay đổi ngẫu nhiên liên tục (41100, 40086, 38604...) chứ không còn cố định ở 33333. |

## 7. Kết quả PASS/FAIL

| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
|---|---|---|
| Cấu hình pfSense ATTACKNET & Firewall Rules | Nhánh ATTACKNET thông mạng với Kali (10.10.0.200), chặn hoàn toàn sang LAN cũ, MGMT và WAN. | PASS |
| Kiểm tra Baseline ban đầu | Truy vấn www.example.lab10.test ra đúng IP thật 172.16.6.10, tải đúng giao diện trang DMZ-WEB. | PASS |
| Xác nhận Kali không sniff được DNSLAB | Chạy tcpdump trên Kali khi VICTIM truy vấn thu được 0 packets captured. | PASS |
| Task 1: Remote Cache Poisoning / Kaminsky | Log AUTH-DNS báo [PENDING] và [HIT]; Resolver báo [CACHE] delegation poisoned; Kali báo SUCCESS (10.10.0.200). | PASS |
| Task 2: Bắt gói cổng 33333 & Kiểm tra trên Victim | Bắt được đủ gói hỏi/đáp giả trên cổng 33333; máy VICTIM nhận IP 10.10.0.200 và mở ra trang Fake Site. | PASS |
| Khôi phục BIND9 & Source-Port Randomization | Tên miền về lại 172.16.6.10; tcpdump thấy BIND9 đổi cổng nguồn UDP ngẫu nhiên ở mỗi truy vấn. | PASS |

## 8. Lỗi gặp phải và cách khắc phục

- Lỗi 1: Máy Kali (10.10.0.200) không ping được Gateway 10.10.0.1 dù đã tạo rule trên tab ATTACKNET.
  - Nguyên nhân: Lúc tạo rule trên pfSense chọn nhầm Protocol là IGMP thay vì ICMP.
  - Cách khắc phục: Đổi lại Protocol từ IGMP sang ICMP trên tab ATTACKNET rồi bấm Apply Changes.
- Lỗi 2: Máy Windows 10 (VICTIM) truy vấn Resolve-DnsName bị timeout ở bước Baseline.
  - Nguyên nhân: Máy Windows 10 còn dính route tĩnh cũ từ bài trước và khi chạy gộp dịch vụ trên cùng máy Ubuntu thì gói nội bộ dùng IP nguồn 172.16.6.10 bị script lọc bỏ.
  - Cách khắc phục: Xóa route cũ (route delete 10.9.0.10), thêm query-source address 10.9.0.10 vào named.conf.options và bỏ dòng lọc IP nguồn nội bộ trong auth_dns_relay_lab10.py.
- Lỗi 3: Bật lại web giả trên Kali báo lỗi Address already in use.
  - Nguyên nhân: Cổng 80 trên Kali đã có tiến trình http.server chạy ngầm từ trước giữ sẵn.
  - Cách khắc phục: Giữ nguyên tiến trình web cổng 80 đang chạy hoặc tắt tiến trình cũ trước khi bật lại.
- Lỗi 4: Trên VICTIM dùng curl.exe thì ra trang Fake Site nhưng mở trình duyệt Edge lại báo ERR_TIMED_OUT.
  - Nguyên nhân: Trình duyệt Edge trên Windows 10 vẫn còn bật Proxy cũ (10.6.0.30:8080) từ bài Lab 7.
  - Cách khắc phục: Tắt Proxy trong cài đặt mạng của Windows 10 rồi tải lại trang trên Edge là hiện ngay Fake Site.
- Lỗi 5: Lúc bật lại BIND9 ở bước cuối thì dịch vụ báo lỗi status=1/FAILURE.
  - Nguyên nhân: Chạy lệnh sed thêm dòng query-source address 10.9.0.10 vào named.conf.options bị lặp 2 lần làm trùng cấu hình.
  - Cách khắc phục: Xóa dòng query-source bị lặp trong named.conf.options rồi khởi động lại bind9 là chạy bình thường.

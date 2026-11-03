Created README.md
Created README.md

Mình đã viết xong và lưu sẵn file [README.md](file:///d:/An%20to%C3%A0n%20v%C3%A0%20b%E1%BA%A3o%20m%E1%BA%ADt%20h%E1%BB%87%20th%E1%BB%91ng%20th%C3%B4ng%20tin/%C4%90%E1%BB%81/LAB10/README.md) (và tại [LAB10/README.md](file:///d:/An%20to%C3%A0n%20v%C3%A0%20b%E1%BA%A3o%20m%E1%BA%ADt%20h%E1%BB%87%20th%E1%BB%91ng%20th%C3%B4ng%20tin/%C4%90%E1%BB%81/LAB10/LAB10/README.md)) theo đúng 100% bố cục 8 mục và giọng văn tự nhiên của bài Lab 8. Dưới đây là toàn bộ nội dung để bạn tiện copy:

---

# Lab 10: Remote DNS Cache Poisoning trong môi trường pfSense cô lập

### 1. Họ và tên
Nguyễn Duy Trường

### 2. MSSV
1150070048

### 3. Tên Lab
Remote DNS Cache Poisoning trong môi trường pfSense cô lập.

### 4. Phiên bản môi trường thực hành
* **VMware Workstation Pro** (tạo 2 LAN Segment cô lập `dns-lab` và `attack-net`, tắt Promiscuous Mode trên Kali).
* **pfSense CE 2.7.2-RELEASE** (kế thừa từ bài trước, giữ cổng `DNSLAB 10.9.0.1/24`, `DMZ 172.16.6.1/24` và thêm cổng mới `ATTACKNET 10.10.0.1/24`).
* **Máy DNS-SERVER (`10.9.0.10`)**: Ubuntu Server 24.04 LTS chạy dịch vụ `BIND 9.20.24` (ở pha Baseline/Hardening) và resolver mô phỏng `vuln_resolver_lab10.py` (cố định cổng nguồn UDP `33333` ở pha tấn công).
* **Máy VICTIM (`10.9.0.100`)**: Máy trạm nạn nhân (Windows 10 Pro) trỏ DNS duy nhất về `10.9.0.10`.
* **Máy ATTACKER (`10.10.0.200`)**: Kali Linux đặt riêng ở nhánh `attack-net` (`Promiscuous Mode = Deny`), chạy `kaminsky_remote_lab10.py`, `evil_dns_lab10.py` (cổng 53), `tcpdump` và Fake Web Server ở cổng 80.
* **Cụm máy DMZ**: `DMZ-WEB` (`172.16.6.10`) chạy trang web thật và `APP-TEST / AUTH-DNS` (`172.16.6.20`) chạy script `auth_dns_relay_lab10.py` (trả lời DNS hợp lệ chậm `3.0 giây` đối với tên miền ngẫu nhiên và mở cổng `UDP 5300` làm race relay).

### 5. Cách dựng môi trường
Bài lab được dựng tiếp nối từ hệ thống `DNS-LAB` của bài trước nhưng nâng cấp lên kịch bản tấn công từ xa (**Remote DNS Cache Poisoning / Kaminsky**). Thay vì để Kali nằm chung mạng `dns-lab` và nghe lén như bài trước, mình tạo thêm một nhánh mạng riêng biệt tên là `ATTACKNET` (`10.10.0.0/24`) thông qua LAN Segment `attack-net` trên VMware và chuyển máy Kali sang nhánh này.

**Các nhánh mạng trong bài gồm:**
* **WAN (`192.168.250.0/24` - NAT)**: Đường ra Internet, bị chặn đối với cả nhánh `DNSLAB` và `ATTACKNET` trong lúc thực hành.
* **MGMT (`192.168.56.0/24` - Host-only)**: Dùng để vào trang quản trị pfSense WebGUI (`https://192.168.56.254`).
* **DMZ (`172.16.6.0/24`)**: Đặt Web Server hợp lệ (`172.16.6.10`) và DNS Server có thẩm quyền kiêm bộ chuyển tiếp đua TXID (`172.16.6.20`).
* **DNSLAB (`10.9.0.0/24` - LAN Segment `dns-lab`)**: Chỉ còn 2 máy là `DNS-SERVER` (`10.9.0.10`) và `VICTIM` (`10.9.0.100`).
* **ATTACKNET (`10.10.0.0/24` - LAN Segment `attack-net`)**: Nhánh mạng riêng chứa máy `ATTACKER Kali` (`10.10.0.200`), đặt `Promiscuous Mode = Deny` để không thể nghe lén lưu lượng của `DNSLAB`.

**Bảng phân chia IP:**

| Thiết bị / Interface | Địa chỉ IP | Subnet Mask | Gateway | DNS | Vai trò / Ghi chú |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **pfSense WAN** | `192.168.250.x` (DHCP) | `255.255.255.0` | Tự động | Tự động | Đường ra ngoài (chặn với `DNSLAB` & `ATTACKNET`) |
| **pfSense MGMT** | `192.168.56.254` | `255.255.255.0` | Không đặt | Không đặt | Cổng vào WebGUI pfSense |
| **pfSense DMZ** | `172.16.6.1` | `255.255.255.0` | Không đặt | Không đặt | Gateway nhánh DMZ |
| **pfSense DNSLAB** | `10.9.0.1` | `255.255.255.0` | Không đặt | Không đặt | Gateway nhánh `DNSLAB` |
| **pfSense ATTACKNET** | `10.10.0.1` | `255.255.255.0` | Không đặt | Không đặt | Gateway nhánh `ATTACKNET` (mới thêm ở Lab 10) |
| **DNS-SERVER** | `10.9.0.10` | `255.255.255.0` | `10.9.0.1` | `10.9.0.10` | Chạy `BIND9` và `vuln_resolver_lab10.py` (port `33333`) |
| **VICTIM** | `10.9.0.100` | `255.255.255.0` | `10.9.0.1` | `10.9.0.10` | Máy nạn nhân Windows 10 truy vấn DNS và mở web |
| **ATTACKER (Kali)** | `10.10.0.200` | `255.255.255.0` | `10.10.0.1` | `10.9.0.10` | Bắn gói đoán TXID từ xa, chạy Evil DNS (53) & Fake Web (80) |
| **DMZ-WEB** | `172.16.6.10` | `255.255.255.0` | `172.16.6.1` | Không đặt | Trang web thật của bài lab (`www.example.lab10.test`) |
| **AUTH-DNS / RELAY** | `172.16.6.20` | `255.255.255.0` | `172.16.6.1` | Không đặt | DNS gốc trả lời chậm `3.0s` + Race Relay cổng `UDP 5300` |

**Nguyên lý hoạt động trong bài:**
* **Cô lập bằng pfSense**:
  * Trên tab `DNSLAB`: Mở cho `VICTIM` (`10.9.0.100`) hỏi DNS tới `10.9.0.10` (cổng 53), mở cho `DNS-SERVER` (`10.9.0.10`) hỏi DNS tới `172.16.6.20` (DNS thật) và `10.10.0.200` (Evil DNS sau khi bị poison), mở cho mạng `10.9.0.0/24` vào web cổng 80 của `172.16.6.10` và `10.10.0.200`. Chặn và ghi log toàn bộ kết nối sang `LAN` (`10.6.0.0/24`), `MGMT` (`192.168.56.0/24`) và `WAN`.
  * Trên tab `ATTACKNET`: Chỉ mở cho `Kali` (`10.10.0.200`) gửi truy vấn mồi tới `10.9.0.10` (cổng 53) và gửi batch đoán TXID tới `172.16.6.20` (cổng `UDP 5300`). Chặn toàn bộ truy cập sang `LAN`, `MGMT` và `WAN`.
* **Khi hoạt động bình thường (Baseline)**: `VICTIM` hỏi `www.example.lab10.test` $\rightarrow$ `DNS-SERVER` (`10.9.0.10`) chạy `BIND9` chuyển tiếp sang `AUTH-DNS` (`172.16.6.20`) $\rightarrow$ nhận về IP thật `172.16.6.10` và mở ra trang web hợp lệ `DMZ-WEB`.
* **Khi Kali tấn công từ xa (Kaminsky Simulation)**: Vì Kali nằm ở `ATTACKNET` nên không thể sniff được `Transaction ID` (chứng minh bằng `tcpdump` thu được `0 packets captured`). Ở pha tấn công, `DNS-SERVER` chạy `vuln_resolver_lab10.py` cố định cổng nguồn UDP là `33333`. Kali gửi truy vấn mồi bằng một tên miền con ngẫu nhiên chưa có trong cache (như `rvjgh0wqw34.example.lab10.test`) để ép Resolver phải hỏi `AUTH-DNS` (`172.16.6.20`) và chờ `3.0 giây`. Ngay trong cửa sổ `3.0 giây` đó, Kali gửi 16 batch đoán toàn bộ $2^{16} = 65.536$ giá trị `TXID` qua cổng `5300`. Khi trúng đúng `TXID` thật, gói trả lời giả mạo chứa bản ghi ủy quyền `NS = ns.evil.example.lab10.test` kèm `Glue A = 10.10.0.200` sẽ về tới cổng `33333` của Resolver trước gói thật, làm toàn bộ vùng `example.lab10.test` bị chuyển hướng sang hỏi Evil DNS trên Kali.

### 6. Các tình huống thực hành

| Tình huống | Tên kịch bản | Nội dung thực hiện |
| :---: | :--- | :--- |
| **1** | **Tách mạng ATTACKNET & Cấu hình pfSense** | Thêm card mạng `attack-net` (`10.10.0.1/24`) trên pfSense; chuyển Kali sang `attack-net` với IP `10.10.0.200/24`, tắt Promiscuous Mode; cấu hình bộ rule cô lập trên cả 2 tab `DNSLAB` và `ATTACKNET`. |
| **2** | **Chuẩn bị DNS-SERVER, AUTH-DNS & Kiểm tra Baseline** | Cấu hình `BIND9` trên `10.9.0.10` chỉ cho phép `10.9.0.0/24` đệ quy và forward `example.lab10.test` về `172.16.6.20`; bật `auth_dns_relay_lab10.py` trên `172.16.6.20`. Kiểm tra từ `VICTIM` (`10.9.0.100`) thấy `www.example.lab10.test` trả về đúng `172.16.6.10` và mở được web thật `DMZ-WEB`. |
| **3** | **Chứng minh tính "Remote" (Không sniff được)** | Chạy `sudo tcpdump -ni any udp port 53` trên Kali trong lúc `VICTIM` truy vấn DNS. Kết quả Kali bắt được `0 packets captured`, chứng minh Kali hoàn toàn mù thông tin, không thấy được gói tin giữa `DNS-SERVER` và `AUTH-DNS`. |
| **4** | **Task 1 - Remote DNS Cache Poisoning (Kaminsky)** | Dừng `BIND9`, bật `vuln_resolver_lab10.py` (cố định source port `33333`) trên `10.9.0.10`. Trên Kali bật `evil_dns_lab10.py` (cổng 53), Fake Web (cổng 80) và chạy `kaminsky_remote_lab10.py`. Quan sát log `[PENDING]`, `[HIT]` (`txid=57473`) trên `AUTH-DNS`, log `[CACHE] delegation poisoned` trên Resolver và thông báo `SUCCESS` trên Kali. |
| **5** | **Task 2 - Bắt gói phân tích & Kiểm chứng trên Victim** | Dùng `tcpdump` trên `DNS-SERVER` bắt gói qua cổng `33333` thấy rõ gói hỏi `10.9.0.10:33333 -> 172.16.6.20:53`, gói trả lời giả `0/1/1` (`NS` + `Glue A`) và gói hỏi sau khi bị poison đi thẳng tới `10.10.0.200:53`. Kiểm tra trên `VICTIM` thấy `www.example.lab10.test` trả về `10.10.0.200` và trình duyệt mở ra trang **LAB 10 - Fake Site**. |
| **6** | **Khôi phục BIND9 & Kiểm chứng Source-Port Randomization** | Dừng `vuln_resolver_lab10.py`, bật lại `BIND9`, xóa cache (`rndc flush`). Bắt gói `tcpdump` trên `DNS-SERVER` khi `VICTIM` truy vấn `www`, `a1`, `a2.example.lab10.test` thấy `www` về lại IP thật `172.16.6.10` và cổng nguồn UDP của `BIND9` thay đổi ngẫu nhiên liên tục (`41100`, `40086`, `38604`, `34069`, `58780`...) chứ không còn cố định ở `33333`. |

### 7. Kết quả PASS/FAIL

| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
| :--- | :--- | :---: |
| **Cấu hình pfSense ATTACKNET & Firewall Rules cô lập** | Nhánh `ATTACKNET` (`10.10.0.1/24`) thông mạng với Kali (`10.10.0.200`), ping `0% loss`; chặn hoàn toàn truy cập sang `LAN`, `MGMT` và `WAN`. | **PASS** |
| **Kiểm tra Baseline (`BIND9` + `AUTH-DNS`)** | `named-checkconf` hợp lệ; `VICTIM` phân giải `www.example.lab10.test` ra đúng `172.16.6.10` và tải được trang `DMZ-WEB` hợp lệ. | **PASS** |
| **Xác nhận Kali không sniff được `DNSLAB`** | Chạy `tcpdump -ni any udp port 53` trên Kali khi `VICTIM` truy vấn thu được `0 packets captured`. | **PASS** |
| **Task 1: Remote Cache Poisoning / Kaminsky Simulation** | Log `AUTH-DNS` báo `[PENDING]` & `[HIT] txid=57473`; Resolver báo `[CACHE] delegation poisoned`; Kali báo `SUCCESS` (`www.example.lab10.test A = 10.10.0.200`). | **PASS** |
| **Task 2: Minh chứng bắt gói (`Packet Capture`) trên cổng `33333`** | Bắt được đầy đủ 4 bước: truy vấn từ `10.9.0.10.33333 -> 172.16.6.20.53`, gói giả mạo `0/1/1` trả về, và truy vấn sau poison đổi hướng sang `10.10.0.200.53`. | **PASS** |
| **Kiểm tra `VICTIM` sau khi Cache bị Poison** | `Resolve-DnsName` trên Windows 10 trả về `10.10.0.200`; cả `curl.exe` và trình duyệt Microsoft Edge đều hiển thị trang **LAB 10 - Fake Site**. | **PASS** |
| **Khôi phục `BIND9` & Kiểm chứng Source-Port Randomization** | Tên miền `www.example.lab10.test` về lại `172.16.6.10`; `tcpdump` chứng minh `BIND9` dùng UDP Source Port ngẫu nhiên (`41100`, `40086`, `38604`...) ở mỗi truy vấn. | **PASS** |

### 8. Lỗi gặp phải và cách khắc phục

* **Lỗi 1: Máy Kali (`10.10.0.200`) không ping được Gateway `10.10.0.1` của pfSense dù đã tạo rule trên tab `ATTACKNET`.**
  * **Nguyên nhân**: Lúc tạo rule cho phép ping trên pfSense, mục *Protocol* vô tình chọn nhầm thành `IGMP` thay vì `ICMP`, và ban đầu card `eth0` của Kali bị mất IP tĩnh khi khởi động lại.
  * **Cách khắc phục**: Vào pfSense sửa lại *Protocol* của rule đầu tiên trên tab `ATTACKNET` từ `IGMP` sang `ICMP`, bấm *Apply Changes* và gán lại IP `10.10.0.200/24` kèm default gateway `10.10.0.1` trên Kali.
* **Lỗi 2: Máy Windows 10 (`VICTIM`) truy vấn `Resolve-DnsName www.example.lab10.test` bị treo timeout ở bước Baseline.**
  * **Nguyên nhân**: Trên máy Windows 10 vẫn còn lưu một tuyến tĩnh (static route) cũ từ bài Lab trước trỏ sai đường đi tới `10.9.0.10`, đồng thời khi chạy gộp cả `DNS-SERVER` (`10.9.0.10`) và `AUTH-DNS` (`172.16.6.20`) trên cùng một máy Ubuntu thì nhân Linux tự chọn IP nguồn `172.16.6.10` khi gửi gói nội bộ làm `auth_dns_relay_lab10.py` bỏ qua gói tin.
  * **Cách khắc phục**: Xóa route cũ trên Windows 10 (`route delete 10.9.0.10`), thêm cấu hình `query-source address 10.9.0.10;` vào `named.conf.options` và bỏ bộ lọc chặn IP nguồn nội bộ trong `auth_dns_relay_lab10.py`.
* **Lỗi 3: Chạy `kaminsky_remote_lab10.py` lần đầu báo `NOT YET` và trên Kali báo lỗi `Address already in use` khi bật lại web giả.**
  * **Nguyên nhân**: Cổng 80 trên Kali đã được tiến trình `http.server` chạy ngầm trước đó giữ sẵn nên bật thêm lần nữa sẽ báo trùng cổng; còn gói UDP cổng `5300` đi từ Kali sang `172.16.6.20` cần được chuyển tiếp đúng tới tiến trình `auth_dns_relay_lab10.py`.
  * **Cách khắc phục**: Giữ nguyên tiến trình web cổng 80 đang chạy sẵn, cập nhật lại bộ lọc địa chỉ trên `auth_dns_relay_lab10.py` rồi chạy lại `python3 ~/LAB10/kaminsky_remote_lab10.py` là báo `SUCCESS` ngay.
* **Lỗi 4: Trên `VICTIM`, lệnh `curl.exe http://www.example.lab10.test` đã ra trang Fake Site nhưng mở trên trình duyệt Microsoft Edge lại báo `ERR_TIMED_OUT`.**
  * **Nguyên nhân**: Trình duyệt Microsoft Edge trên Windows 10 vẫn còn bật cấu hình Proxy cũ (`10.6.0.30:8080`) từ bài Lab 7 trong Registry, trong khi `curl.exe` kết nối trực tiếp không qua proxy.
  * **Cách khắc phục**: Tắt Proxy hệ thống bằng PowerShell (`Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0`) rồi bấm Refresh trên Edge là hiện ngay trang Fake Site.
* **Lỗi 5: Lúc khôi phục `BIND9` ở bước cuối, dịch vụ `named.service` báo lỗi `status=1/FAILURE` không khởi động được.**
  * **Nguyên nhân**: Do chạy lệnh `sed` chèn dòng `query-source address 10.9.0.10;` vào file `/etc/bind/named.conf.options` bị lặp 2 lần (một lần ở đầu bài và một lần ở cuối bài), làm `BIND9` báo lỗi cú pháp trùng lặp cấu hình.
  * **Cách khắc phục**: Dùng lệnh `sudo sed -i '/query-source/d' /etc/bind/named.conf.options` để xóa dòng bị lặp, chỉ chèn lại đúng 1 dòng duy nhất rồi khởi động lại `bind9` là dịch vụ lên xanh (`active running`) bình thường.

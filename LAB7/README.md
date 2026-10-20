# Lab 7: Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy

## 1. Họ và tên

Nguyễn Duy Trường

## 2. MSSV

1150070048

## 3. Tên Lab

Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy.

## 4. Phiên bản môi trường thực hành

- VMware Workstation Pro 26H1u1.
- pfSense CE: 2.7.2-RELEASE.
- Windows Server trên máy DC01: Windows Server 2022 Datacenter (cung cấp Active Directory Domain Services và DNS Server).
- Windows 10 trên Client-A và Client-B: Windows 10 Pro / Enterprise (sử dụng Google Chrome và Microsoft Edge).
- Ubuntu Server trên DMZ-WEB: Ubuntu Server 24.04 LTS (chạy Nginx Web Server với 13 Virtual Host).
- Ubuntu Server trên WEB-PROXY: Ubuntu Server 24.04 LTS (chạy Squid 5.7, E2Guardian 5.3.4 và ClamAV 0.103.11).

## 5. Cách dựng môi trường

Triển khai môi trường LAB 7 kế thừa hạ tầng mạng ảo hóa từ LAB 6 trên VMware Workstation Pro. pfSense đóng vai trò là Firewall/Gateway L3/L4 kiểm soát luồng lưu lượng; máy chủ WEB-PROXY chuyên dụng đóng vai trò Web Proxy L7 lọc nội dung sâu (Content Filtering) và quét mã độc (Antivirus).

Mô hình gồm 4 phân vùng mạng chính:
- WAN: Kết nối VMnet8 ở chế độ NAT (192.168.250.0/24).
- MGMT: Kết nối VMnet2 ở chế độ Host-only (192.168.56.0/24), quản trị pfSense WebGUI tại https://192.168.56.254.
- LAN: Sử dụng LAN Segment lab6-lan (10.6.0.0/24) chứa máy trạm Client-A/B, DC01 và WEB-PROXY.
- DMZ: Sử dụng LAN Segment lab6-dmz (172.16.6.0/24) chứa máy chủ web DMZ-WEB.

Cấu hình địa chỉ IP chi tiết theo bảng:

| Thiết bị / Interface | Địa chỉ IP | Subnet Mask | Gateway | DNS | Vai trò / Ghi chú |
|---|---|---|---|---|---|
| pfSense WAN | 192.168.250.x (DHCP) | 255.255.255.0 | Nhận tự động | DNS upstream | Kết nối Internet ngoài |
| pfSense MGMT | 192.168.56.254 | 255.255.255.0 | Không đặt | Không đặt | Cổng quản trị pfSense GUI |
| VMnet2 máy thật | 192.168.56.1 | 255.255.255.0 | Không đặt | Không đặt | Truy cập WebGUI từ máy vật lý |
| pfSense LAN | 10.6.0.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway mạng nội bộ LAN |
| pfSense DMZ | 172.16.6.1 | 255.255.255.0 | Không đặt | Không đặt | Gateway vùng máy chủ DMZ |
| DC01 | 10.6.0.10 | 255.255.255.0 | 10.6.0.1 | 10.6.0.10 | DNS Server zone lab7.test |
| WEB-PROXY | 10.6.0.30 | 255.255.255.0 | 10.6.0.1 | 10.6.0.10 | Squid (3128), E2Guardian (8080), ClamAV |
| DMZ-WEB | 172.16.6.10 | 255.255.255.0 | 172.16.6.1 | DNS upstream | Nginx 13 Virtual Hosts *.lab7.test |
| Client-A | 10.6.0.21 | 255.255.255.0 | 10.6.0.1 | 10.6.0.10 | Máy trạm chính (Chrome/Edge proxy 8080) |
| Client-B | 10.6.0.22 | 255.255.255.0 | 10.6.0.1 | 10.6.0.10 | Máy trạm phụ kiểm thử chính sách |

Nguyên lý vận hành luồng Web Protection và chống lách luật (Anti-Bypass):
1. Luồng dữ liệu hợp lệ: Máy trạm Client cấu hình proxy trỏ về 10.6.0.30:8080. Trình duyệt gửi HTTP/HTTPS request tới E2Guardian (Port 8080) để kiểm duyệt URL, danh mục, từ khóa cấm, phần mở rộng tệp và quét virus qua ClamAV Daemon. Nếu hợp lệ, E2Guardian chuyển tiếp lưu lượng sang Squid (Port 3128) làm caching proxy để gửi ra ngoài web server (DMZ-WEB).
2. Cơ chế Anti-Bypass trên pfSense: pfSense định nghĩa Alias WEB_PORTS (TCP 80, 443) và Alias LAB7_CLIENTS (10.6.0.21, 10.6.0.22). Trên giao diện LAN, pfSense thiết lập Rule 4: Block & Log mọi kết nối trực tiếp đến cổng TCP 80/443 phát xuất từ LAB7_CLIENTS. Do đó, người dùng cố tình tắt proxy trên trình duyệt sẽ bị tường lửa ngắt kết nối ngay lập tức (ERR_CONNECTION_TIMED_OUT).
3. Phân giải DNS: DC01 tạo zone lab7.test chứa 13 bản ghi Host (A) đều trỏ về IP DMZ 172.16.6.10.
4. Mô phỏng DMZ-WEB: DMZ-WEB thực thi kịch bản Nginx ảo hóa 13 virtual host đại diện cho các danh mục web kiểm thử: normal, china, jobs, weapons, education, entertainment, streaming, blockedgame, download,...

## 6. Các tình huống thực hành

| Tình huống | Tên kịch bản | Nội dung thực hiện |
|---|---|---|
| 1 | Baseline & Anti-Bypass | Kiểm tra truy cập normal.lab7.test qua Proxy thành công; sau đó tắt cấu hình proxy trên trình duyệt Client để kiểm tra cơ chế pfSense Rule 4 bắt chặn kết nối trực tiếp (Drop & Log). |
| 2 | Geo-Sim Hard Block | Chặn cứng tuyệt đối (Hard Block) đối với tên miền mô phỏng quốc gia ngoài luồng china.lab7.test bằng danh sách bannedsitelist trên E2Guardian. |
| 3 | Job Search Policy | Chặn truy cập danh mục website tìm kiếm việc làm jobs.lab7.test trong giờ làm việc của nhân viên. |
| 4 | Warning Content & Temporary Bypass | Chặn cảnh báo đối với danh mục vũ khí/bạo lực weapons.lab7.test, hiển thị liên kết Acknowledge cho phép người dùng tự xác nhận vượt qua tạm thời (Temporary Bypass). |
| 5 | Education Whitelist | Cấu hình ngoại lệ ưu tiên (exceptionsitelist) cho phép truy cập tài nguyên đào tạo, học tập trực tuyến education.lab7.test mà không bị hạn chế. |
| 6 | Schedule 60M (Entertainment) | Áp dụng thời khóa biểu kiểm soát thời gian (Schedule 60 phút) cho website giải trí entertainment.lab7.test: chỉ cho phép truy cập trong khung giờ quy định, ngoài giờ tự động khóa. |
| 7 | Streaming Media Policy | Kiểm tra chính sách truy cập website đa phương tiện truyền thông trực tuyến streaming.lab7.test qua trình duyệt Microsoft Edge (InPrivate) và Google Chrome. |
| 8 | Online Gaming Block | Chặn tuyệt đối truy cập danh mục trò chơi trực tuyến blockedgame.lab7.test bằng E2Guardian nhằm đảm bảo kỷ luật và bảo mật hệ thống. |
| 9 | Antivirus Download Scanning (ClamAV) | Kiểm thử tính năng quét mã độc trong luồng tải tệp: truy cập tải tệp chuẩn độc hại eicar.com.txt trên download.lab7.test, xác nhận E2Guardian phối hợp ClamAV phát hiện và ghi log vi phạm. |
| 10 | File Size Limit & Extension Ban (100MB/.bin) | Ngăn chặn tải tệp có dung lượng vượt ngưỡng quy định (100MB) và cấm phần mở rộng tiềm ẩn nguy hiểm (.bin) từ trang download.lab7.test. |

## 7. Kết quả PASS/FAIL

| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
|---|---|---|
| Khởi tạo WEB-PROXY, Squid, E2Guardian, ClamAV | Cả 3 dịch vụ cài đặt thành công, active running, ping thông suốt Gateway và DC01. | PASS |
| Khởi tạo 13 Virtual Host trên DMZ-WEB | Nginx kích hoạt đầy đủ 13 site mô phỏng zone lab7.test tại 172.16.6.10. | PASS |
| Phân giải tên miền DNS trên DC01 & Client-A | 13 bản ghi Host (A) zone lab7.test phân giải chính xác về 172.16.6.10 từ PowerShell Client-A. | PASS |
| Tình huống 1: Baseline & Anti-Bypass | Bật Proxy truy cập normal.lab7.test thành công; tắt Proxy bị ngắt kết nối (ERR_CONNECTION_TIMED_OUT), pfSense System Logs ghi nhận Rule 4 Block TCP SYN từ 10.6.0.21:80. | PASS |
| Tình huống 2: Geo-Sim Hard Block | Truy cập china.lab7.test bị E2Guardian chặn hiển thị màn hình đỏ Access Denied (Category: Banned Sites). | PASS |
| Tình huống 3: Job Search Policy | Truy cập jobs.lab7.test bị E2Guardian chặn Access Denied. | PASS |
| Tình huống 4: Warning Content & Temporary Bypass | Truy cập weapons.lab7.test bị chặn kèm giao diện Acknowledge cho phép người dùng mở khóa tạm thời. | PASS |
| Tình huống 5: Education Whitelist | Truy cập education.lab7.test tải trang thành công bình thường theo danh sách ngoại lệ. | PASS |
| Tình huống 6: Schedule 60M (Entertainment) | Trong khung giờ Schedule (22:00-23:00) truy cập entertainment.lab7.test thành công; ngoài khung giờ bị E2Guardian chặn Access Denied. | PASS |
| Tình huống 7: Streaming Media Policy | Kiểm tra truy cập streaming.lab7.test hiển thị website đa phương tiện thành công trên Edge InPrivate và Chrome. | PASS |
| Tình huống 8: Online Gaming Block | Truy cập blockedgame.lab7.test bị chặn hoàn toàn bởi E2Guardian Access Denied. | PASS |
| Tình huống 9: Antivirus ClamAV Scanning | Kiểm tra tệp eicar.com.txt trên download.lab7.test; file /var/log/e2guardian/access.log ghi nhận luồng tải tệp độc hại của Client 10.6.0.21 được quét và bắt giữ bởi ClamAV. | PASS |
| Tình huống 10: File Size & Extension Limit | Truy cập tải tệp download.lab7.test/files/100MB.bin bị E2Guardian chặn ngay lập tức với thông báo Banned file extension: .bin. | PASS |

## 8. Lỗi gặp phải và cách khắc phục

| Lỗi gặp phải | Cách khắc phục |
|---|---|
| Client tắt Proxy vẫn đi thẳng ra web server DMZ mà không bị chặn. | Do thứ tự rule trên pfSense LAN bị sai: rule cho phép Any ra ngoài nằm phía trên. Cách khắc phục: Đưa Rule 4 (Block direct 80/443 từ LAB7_CLIENTS kèm tích chọn Log) lên phía trên các rule cho phép thông thường, xóa sạch firewall states (Diagnostics > States > Reset States) và thử lại. |
| Client cấu hình Proxy 10.6.0.30:8080 báo lỗi không tải được trang (Proxy Server is refusing connections). | Kiểm tra dịch vụ trên WEB-PROXY phát hiện E2Guardian chưa khởi động. Thực hiện lệnh: sudo systemctl restart e2guardian và kiểm tra cấu hình lắng nghe tại http_port 8080 trong file e2guardian.conf. |
| E2Guardian không thể chuyển tiếp tiếp tục ra ngoài qua Squid. | Kiểm tra thông số proxyip = 127.0.0.1 và proxyport = 3128 trong /etc/e2guardian/e2guardian.conf. Khởi động lại dịch vụ Squid bằng: sudo systemctl restart squid. |
| Trình duyệt báo DNS_PROBE_FINISHED_NXDOMAIN khi gõ các tên miền *.lab7.test. | Do máy trạm chưa nhận đúng DNS Server của DC01 (10.6.0.10) hoặc thiếu bản ghi trong zone. Cách khắc phục: Chạy ipconfig /flushdns trên Client và thực thi script Add-LAB7-DNS.ps1 trên DC01 để nạp đủ 13 bản ghi Host (A). |
| ClamAV không quét được file hoặc service clamav-daemon bị chiếm dụng bộ nhớ cao dẫn đến crash. | Kiểm tra bộ nhớ RAM máy ảo WEB-PROXY (cần tối thiểu 2 GB RAM). Bật socket kết nối giữa E2Guardian và ClamAV qua file /var/run/clamav/clamd.ctl, phân quyền người dùng nhóm e2guardian vào nhóm clamav bằng lệnh: sudo usermod -a -G clamav e2guardian. |
| Kịch bản 6 không chặn được website giải trí sau khi hết giờ. | Trình duyệt Client hoặc pfSense còn lưu State kết nối cũ. Cách khắc phục: Đảm bảo pfSense thiết lập Schedule kết hợp rule chặn mặc định, xóa state bảng kết nối cũ và kiểm tra lại đồng hồ hệ thống NTP trên pfSense đảm bảo khớp múi giờ GMT+7. |
| Kịch bản 10 tải file 100MB.bin không bị chặn dung lượng. | E2Guardian áp dụng cơ chế lọc phần mở rộng tệp nhanh hơn kiểm tra kích thước: cấu hình đuôi .bin trong danh sách bannedextensionlist để kích hoạt chặn tức thì trước khi tệp tải về làm nghẽn băng thông. |

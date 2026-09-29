# LAB 4: Khảo sát và đánh giá bề mặt mạng bằng Nmap

## 1. Họ và tên
Nguyễn Duy Trường

## 2. MSSV
1150070048

## 3. Tên Lab
Khảo sát và đánh giá bề mặt mạng bằng Nmap

## 4. Phiên bản môi trường thực hành
VMware Workstation Pro 26H1u1, Kali Linux Rolling 2026.2, Metasploitable 2, Windows 10 64-bit, Nmap 7.99/7.991 và Npcap 1.88.

## 5. Cách dựng môi trường
Em tạo các máy ảo bằng VMware Workstation Pro. Kali Linux dùng làm máy quét, Metasploitable 2 và Windows 10 dùng làm máy mục tiêu. Các máy được kết nối bằng mạng Host-Only VMnet1 để thực hành trong mạng nội bộ.

## 6. Các tình huống đã thực hiện
- Kiểm tra kết nối giữa các máy.
- Quét tìm các host đang hoạt động.
- Thực hiện TCP Connect Scan và SYN Scan.
- Thực hiện FIN, Xmas, NULL và ACK Scan.
- Quét các cổng UDP phổ biến.
- Phát hiện dịch vụ và phiên bản bằng -sV.
- Nhận diện hệ điều hành bằng -O và quét tổng hợp bằng -A.
- Sử dụng NSE Script kiểm tra dịch vụ SMB.
- Lưu kết quả quét dưới dạng TXT, XML, Grepable và HTML.
- Thực hiện quét trước và sau khi hardening trên Windows 10.

## 7. Kết quả PASS/FAIL
PASS: Các máy kết nối được với nhau, các kiểu quét Nmap thực hiện thành công và kết quả được lưu ra file. Sau khi hardening Windows 10, cổng 8080 chuyển từ trạng thái open sang filtered.

## 8. Lỗi gặp phải và cách khắc phục
Lỗi: Kali Linux ban đầu không nhận được địa chỉ IPv4 khi sử dụng mạng Host-Only.

Khắc phục: Em bật DHCP cho VMnet1 trong VMware Virtual Network Editor và kiểm tra lại Network Adapter. Sau đó Kali nhận được IP 172.16.16.128/24 và kết nối Host-Only bình thường.

Lỗi: File XML ban đầu không đúng định dạng nên không thể chuyển sang HTML.

Khắc phục: Em xuất lại kết quả bằng tùy chọn `-oX`, sau đó dùng `xsltproc` để chuyển file XML sang HTML thành công.

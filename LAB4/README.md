# LAB 4: Khảo sát và thực hành Nmap

## 1. Họ và tên
Nguyễn Duy Trường

## 2. MSSV
1150070048

## 3. Tên Lab
Khảo sát và thực hành Nmap

## 4. Phiên bản môi trường thực hành
VMware Workstation Pro 26H1u1, Kali Linux Rolling 2026.2, Metasploitable 2, Windows 10/11 và Nmap.

## 5. Cách dựng môi trường
Em tạo các máy ảo bằng VMware Workstation Pro. Kali Linux dùng làm máy quét, Metasploitable 2 làm máy mục tiêu và Windows dùng để kiểm tra, đối chiếu. Các máy được kết nối bằng mạng Host-Only để thực hành trong mạng nội bộ.

## 6. Các tình huống đã thực hiện
- Kiểm tra kết nối giữa các máy.
- Quét tìm host đang hoạt động.
- Quét cổng TCP và UDP.
- Thực hiện SYN, FIN, Xmas, NULL và ACK Scan.
- Phát hiện dịch vụ và phiên bản.
- Nhận diện hệ điều hành.
- Sử dụng NSE Script kiểm tra dịch vụ SMB.
- Lưu kết quả quét ra file.
- Thực hiện quét trước và sau khi hardening.

## 7. Kết quả PASS/FAIL
PASS: Các máy kết nối được với nhau và thực hiện được các nội dung quét Nmap theo yêu cầu.

## 8. Lỗi gặp phải và cách khắc phục
Lỗi: Kali Linux ban đầu không nhận được địa chỉ IPv4 khi dùng mạng Host-Only.

Khắc phục: Em bật DHCP cho VMnet1 trong VMware Virtual Network Editor và kiểm tra lại Network Adapter. Sau đó Kali nhận được IP 172.16.16.128/24 và kết nối Host-Only bình thường.

# Lab 3: Mã hóa khóa công khai

## 1. Họ và tên

Nguyễn Duy Trường

## 2. MSSV

1150070048

## 3. Tên Lab

Mã hóa khóa công khai: RSA, ECC, hệ mã lai RSA + AES, chữ ký số ECDSA và lưu lịch sử thao tác trên Microsoft SQL Server.

## 4. Phiên bản môi trường thực hành

- Hệ điều hành: Windows 11 64-bit.
- Ngôn ngữ và công cụ: Java OpenJDK 21.0.13 LTS, Apache Maven 3.6.3, Visual Studio Code.
- Thư viện sử dụng: Gói bảo mật chuẩn Java (java.security, javax.crypto) và Microsoft JDBC Driver (mssql-jdbc 12.8.1.jre11).
- Hệ quản trị cơ sở dữ liệu: Microsoft SQL Server 2025 (17.00.1000) và SQL Server Management Studio (SSMS).

## 5. Cách dựng môi trường

Triển khai dự án Maven giao diện Java Swing kết hợp cơ sở dữ liệu Microsoft SQL Server để thực hiện mã hóa khóa công khai, hệ mã lai, ký số xác thực và lưu lịch sử thao tác. Cấu hình chi tiết hệ thống theo bảng:

| Thành phần | Giá trị thiết lập | Vai trò |
| :--- | :--- | :--- |
| Dự án Java | LAB3_MaHoaKhoaCongKhai (package week_03) | Chứa cấu hình pom.xml, các lớp thuật toán mật mã, chương trình Console và 7 giao diện Swing |
| Giao thức kết nối | TCP/IP cổng 14333 | Cổng kết nối cố định giữa ứng dụng Java (JDBC) và SQL Server |
| Cơ sở dữ liệu | LAB3_MaHoaKhoaCongKhai | Lưu trữ bảng dbo.History ghi nhận nhật ký mã hóa, giải mã, ký số và xác minh |
| Tài khoản kết nối | lab3_user | Tài khoản phân quyền tối thiểu, chỉ cấp quyền SELECT và INSERT trên bảng dbo.History |

## 6. Kịch bản thực hiện

| Tình huống | Tên kịch bản | Nội dung thực hiện |
| :---: | :--- | :--- |
| 1 | Bài 3.1 – Mật mã RSA (Console) | Sinh cặp khóa RSA 2048 bit (181 ms) và 3072 bit (628 ms) với đệm OAEP-SHA256, mã hóa và giải mã bản rõ HELLO RSA, kiểm tra giới hạn 190 byte và 318 byte. |
| 2 | Bài 3.2 – Mật mã RSA (Swing) | Xây dựng giao diện frm_RSA mã hóa và giải mã thông điệp, thử mã hóa hai lần liên tiếp cùng bản rõ HELLO RSA để kiểm chứng tính ngẫu nhiên của OAEP. |
| 3 | Bài 3.3 – Mật mã đường cong Elliptic (ECC) | Mã hóa theo sơ đồ ECIES trên đường cong secp256r1 (ECDH + HKDF-SHA256 + AES-256-GCM) với bản rõ HELLO ECC, lưu nạp tệp khóa thử nghiệm và kiểm tra bắt lỗi sai khóa riêng. |
| 4 | Bài 3.4 – Hệ mã lai RSA + AES | Sử dụng AES-256-GCM mã hóa văn bản dài trên 1.000 ký tự và RSA-2048 OAEP bọc khóa phiên AES 32 byte, kiểm thử sửa 1 ký tự bản mã để phát hiện can thiệp dữ liệu. |
| 5 | Bài 3.5 – Chữ ký số ECDSA | Tạo chữ ký số bằng khóa riêng và xác minh bằng khóa công khai (SHA256withECDSA) với thông điệp THONG DIEP A (VALID) và sửa thành THONG DIEP B hoặc đổi khóa (INVALID). |
| 6 | Tích hợp Menu chính và SQL Server | Xây dựng cửa sổ điều hướng frm_Menu, kết nối SQL Server trên frm_KetNoiSQL, tra cứu bảng lịch sử trên frm_LichSu và đối chiếu thống kê truy vấn trên SSMS. |

## 7. Kết quả PASS/FAIL

| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
| :--- | :--- | :---: |
| Kiểm thử RSA Console (2048/3072 bit) và RSA Swing | Giải mã đúng HELLO RSA ở cả hai cỡ khóa (tạo khóa 181 ms và 628 ms), hai lần mã hóa cùng bản rõ sinh ra hai bản mã Base64 khác nhau. | PASS |
| Kiểm thử ECC (secp256r1) và lưu nạp khóa | Giải mã đúng HELLO ECC, khôi phục đúng cặp khóa thử nghiệm từ tệp và báo lỗi AEADBadTagException khi dùng khóa riêng khác. | PASS |
| Kiểm thử hệ mã lai RSA + AES với văn bản dài | Mã hóa và giải mã toàn vẹn văn bản dài trên 1.000 ký tự, phát hiện và từ chối giải mã ngay khi sửa 1 ký tự bản mã. | PASS |
| Kiểm thử chữ ký số ECDSA | Ký và xác minh THONG DIEP A trả về VALID; sửa thông điệp thành THONG DIEP B hoặc dùng khóa không khớp trả về INVALID. | PASS |
| Kiểm thử Menu chính và nhật ký SQL Server | Mở đúng tất cả các cửa sổ con không tắt chương trình chính, kết nối thành công cổng 14333 và hiển thị khớp 12 bản ghi lịch sử giữa frm_LichSu và SSMS. | PASS |

## 8. Lỗi gặp phải và cách khắc phục

| Lỗi gặp phải | Cách khắc phục |
| :--- | :--- |
| Lệnh mvn không nhận diện trực tiếp trên cửa sổ Terminal của VS Code và báo lỗi phiên bản trình biên dịch mặc định | Sử dụng bộ Maven Wrapper đi kèm tiện ích Maven for Java trên VS Code kết hợp truyền tham số biên dịch chuẩn Java 17 để build thành công toàn bộ 16 tệp nguồn. |
| Đổi cặp khóa mới trên form ECC dẫn đến không giải mã được bản mã cũ (báo lỗi Tag mismatch) | Sử dụng chức năng Save Keys để lưu cặp khóa trước khi đổi và nạp lại đúng tệp khóa tương ứng bằng nút Load Keys khi cần giải mã. |

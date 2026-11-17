# Lab 2: Mã hóa đối xứng hiện đại
## 1. Họ và tên
Nguyễn Duy Trường
## 2. MSSV
1150070048
## 3. Tên Lab
Mã hóa đối xứng hiện đại, xác thực người dùng lưu lịch sử SQL Server và ứng dụng mã hóa bằng Python.
## 4. Phiên bản môi trường thực hành
- Hệ điều hành: Windows 11 64-bit.
- Ngôn ngữ và công cụ: Java JDK 21, Maven, Visual Studio Code, Python 3.14.
- Hệ quản trị cơ sở dữ liệu: Microsoft SQL Server 2025 và SQL Server Management Studio.
## 5. Cách dựng môi trường
Triển khai dự án Maven giao diện Java Swing kết hợp cơ sở dữ liệu Microsoft SQL Server và ứng dụng Python Tkinter để thực hiện mã hóa, giải mã, xác thực tài khoản và lưu lịch sử thao tác.
Cấu hình chi tiết hệ thống theo bảng:
| Thành phần | Giá trị thiết lập | Vai trò |
|---|---|---|
| Dự án Java | LAB2_MaHoaHienDai | Chứa cấu hình Maven, mã nguồn các thuật toán và giao diện đồ họa |
| Ứng dụng Python | LAB2_Python | Chứa chương trình giao diện Tkinter mã hóa và tự kiểm thử AES-CBC |
| Giao thức kết nối | TCP/IP cổng 14333 | Cổng kết nối cố định giữa ứng dụng Java và SQL Server |
| Cơ sở dữ liệu | LAB2_MaHoaHienDai | Lưu trữ bảng người dùng và bảng lịch sử mã hóa, giải mã |
| Tài khoản kết nối | lab2_user | Chỉ cấp quyền đọc và ghi dữ liệu trên cơ sở dữ liệu bài thực hành |
Hướng dẫn chạy chương trình:
- Bước 1: Mở SSMS bằng quyền quản trị, chạy các tệp SQL tạo cơ sở dữ liệu LAB2_MaHoaHienDai và tài khoản lab2_user trên cổng 14333.
- Bước 2: Mở dự án LAB2_MaHoaHienDai, chạy cửa sổ chính frm_Menu, bấm Kết nối SQL Server nhập mật khẩu rồi chọn từng bài thực hành.
- Bước 3: Cài thư viện cryptography cho Python và chạy tệp aes_cbc_gui.py trong thư mục LAB2_Python.
## 6. Các tình huống thực hành
| Tình huống | Tên kịch bản | Nội dung thực hiện |
|---|---|---|
| 1 | Bài 2.1 – Mật mã DES | Mã hóa và giải mã văn bản tiếng Việt chế độ CBC kèm IV ngẫu nhiên 8 byte với khóa 12345678, lưu mở tệp và kiểm tra bắt lỗi sai khóa. |
| 2 | Bài 2.2 – Mật mã TripleDES | Mã hóa và giải mã với khóa chuẩn 24 byte, kiểm tra bắt lỗi khi nhập khóa không đủ 24 ký tự. |
| 3 | Bài 2.3 – Mật mã AES và SQL Server | Đăng ký, đăng nhập tài khoản băm bằng PBKDF2 kèm salt, cấp phiên Ticket, mã hóa AES-256-GCM và kiểm tra chống sửa đổi bản mã. |
| 4 | Bài 2.4 – Mật mã dòng RC4 | Sinh dòng khóa KSA và PRGA XOR dữ liệu, kiểm thử bản rõ Plaintext với khóa Key thu được bản mã HEX BBF316E8D940AF0AD3. |
| 5 | Bài 2.5 – Mật mã Twofish | Sử dụng thư viện BouncyCastle mã hóa chế độ CBC với IV 32 ký tự HEX, lưu và mở tệp khôi phục lại đúng IV và bản mã. |
| 6 | Bài 2.6 – AES-CTR bằng Java | Sinh khóa 256 bit và Nonce 96 bit ngẫu nhiên để mã hóa chế độ bộ đếm không cần đệm dữ liệu. |
| 7 | Bài 2.6 – AES-CBC bằng Python | Xây dựng giao diện Tkinter mã hóa văn bản dài tiếng Việt và kiểm thử tự động bằng tham số self-test. |
| 8 | Tích hợp Menu chính | Xây dựng cửa sổ điều hướng trung tâm frm_Menu để mở toàn bộ các bài và xem lịch sử thao tác trên frm_LichSu. |
## 7. Kết quả PASS/FAIL
| Nội dung kiểm thử | Kết quả thực tế | Đánh giá |
|---|---|---|
| Kiểm thử DES, TripleDES, RC4 và Twofish | Mã hóa, giải mã đúng dữ liệu chuẩn, khôi phục đúng IV từ tệp và bắt lỗi khóa sai chính xác. | PASS |
| Kiểm thử AES-GCM kết hợp đăng ký, đăng nhập SQL Server | Đăng nhập cấp Ticket thành công, lưu đầy đủ bản ghi người dùng, lịch sử và phát hiện bản mã bị sửa. | PASS |
| Kiểm thử hai bài nâng cao AES-CTR Java và AES-CBC Python | Giải mã đúng văn bản dài tiếng Việt, chế độ tự kiểm thử Python báo SELF-TEST OK. | PASS |
| Kiểm thử cửa sổ Menu chính và xem lịch sử | Mở đúng tất cả các cửa sổ con không làm tắt chương trình chính, hiển thị khớp 22 bản ghi từ SQL Server. | PASS |
## 8. Lỗi gặp phải và cách khắc phục
| Lỗi gặp phải | Cách khắc phục |
|---|---|
| Tệp cấu hình pom.xml đặt nhầm vào thư mục con làm báo lỗi sai đường dẫn package. | Di chuyển tệp pom.xml ra đúng thư mục gốc của dự án và tải lại cấu hình Java trong trình soạn thảo. |
| Chạy kịch bản khởi tạo cơ sở dữ liệu báo lỗi không đủ quyền trên master. | Chuyển kết nối trong SSMS sang quyền quản trị Windows Authentication trước khi chạy kịch bản tạo CSDL. |

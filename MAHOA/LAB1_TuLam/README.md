Thực hành: Lab 1: Mã hóa đối xứng cổ điển
1. Họ và tên

Nguyễn Duy Trường

2. MSSV

1150070048

3. Tên Lab

Mã hóa đối xứng cổ điển, chương trình tổng hợp lưu lịch sử SQL Server và phân tích mật mã cổ điển.

4. Phiên bản môi trường thực hành
Hệ điều hành: Windows 11 64-bit.
Ngôn ngữ và công cụ: Java JDK 17+, Maven, Visual Studio Code, Apache NetBeans.
Hệ quản trị cơ sở dữ liệu: Microsoft SQL Server 2025 và SQL Server Management Studio.
5. Cách dựng môi trường

Triển khai dự án Maven kết hợp giao diện đồ họa Java Swing và cơ sở dữ liệu Microsoft SQL Server để thực hiện mã hóa, giải mã, lưu lịch sử và phân tích mật mã cổ điển.

Cấu hình chi tiết hệ thống theo bảng:

Thành phần	Giá trị thiết lập	Vai trò
Dự án mã nguồn	LAB1_TuLam	Chứa cấu hình Maven, mã nguồn các thuật toán và giao diện đồ họa
Giao thức kết nối	TCP/IP cổng 14333	Cổng kết nối cố định giữa ứng dụng Java và SQL Server
Cơ sở dữ liệu	LAB1_MaHoaCoDien	Lưu trữ bảng lịch sử mã hóa và giải mã
Tài khoản kết nối	lab1_user	Chỉ cấp quyền đọc, thêm và xóa dữ liệu trên bảng lịch sử
6. Các tình huống thực hành
Tình huống	Tên kịch bản	Nội dung thực hiện
1	Bài 1.1 – Mật mã Caesar	Mã hóa bản rõ HELLO với khóa 3 thu được KHOOR, giải mã ngược lại, lưu mở tệp văn bản và kiểm tra bắt lỗi khóa chữ abc.
2	Bài 1.2 – Mật mã Vigenère	Mã hóa bản rõ HELLO với khóa KEY thu được RIJVS, giải mã ngược lại và kiểm tra bắt lỗi khóa chứa chữ số K3Y.
3	Bài 1.3 – Mật mã Rail Fence	Mã hóa bản rõ HELLO WORLD với số hàng 3 thu được HOREL OLLWD, giải mã ngược lại và kiểm tra bắt lỗi khi nhập số hàng bằng 1.
4	Bài 1.4 – Mật mã Playfair	Tạo ma trận khóa 5x5 với từ khóa KEYWORD, mã hóa HELLOWORLD thu được GYIZSCOKCFBU, giải mã thu được HELXLOWORLDX.
5	Bài 1.5 – Mật mã Hoán vị cột	Mã hóa bản rõ BAOMAT với khóa thứ tự cột 2,3,1 thu được AAOTBM, giải mã ngược lại và kiểm tra bắt lỗi khóa trùng cột 1,1,2.
6	Bài 1.6 – Mật mã Autokey	Mã hóa bản rõ HELLO với khóa mồi KEY thu được RIJSS, giải mã ngược lại và kiểm tra bắt lỗi khi nhập khóa số 123.
7	Bài nâng cao 01 – Chương trình tổng hợp và SQL Server	Tích hợp cả 6 thuật toán trên một giao diện chung, kết nối SQL Server cổng 14333 và tự động lưu lịch sử thao tác vào cơ sở dữ liệu.
8	Bài nâng cao 02 – Phân tích mật mã cổ điển	Thực hiện 3 thí nghiệm thám mã: vét cạn 26 khóa Caesar, thống kê tần suất chữ cái tiếng Anh và suy khóa Vigenère khi biết trước bản rõ.
9	Tích hợp Menu chính	Xây dựng cửa sổ điều hướng trung tâm để mở toàn bộ các bài thực hành.
7. Kết quả PASS/FAIL
Nội dung kiểm thử	Kết quả thực tế	Đánh giá
Kiểm thử 6 bài mã hóa cơ bản	Mã hóa, giải mã, lưu mở tệp văn bản và bắt lỗi nhập sai khóa đúng theo bộ dữ liệu chuẩn.	PASS
Kiểm thử chương trình tổng hợp và lưu lịch sử SQL Server	Kết nối thành công cổng 14333, lưu đầy đủ 6 bản ghi lịch sử vào cơ sở dữ liệu và hiển thị khớp với SSMS.	PASS
Kiểm thử 3 thí nghiệm phân tích mật mã cổ điển	Vét cạn tìm đúng khóa k = 3, thống kê tần suất đúng chữ H và W chiếm 21.7%, suy đúng khóa lặp KEYKE.	PASS
Kiểm thử cửa sổ điều hướng Menu chính	Mở đúng tất cả các cửa sổ chức năng, đóng cửa sổ con không làm tắt chương trình chính.	PASS
8. Lỗi gặp phải và cách khắc phục
Lỗi gặp phải	Cách khắc phục
Tạo tệp mã nguồn mới bị nằm lệch ngoài thư mục package làm báo lỗi biên dịch.	Di chuyển tệp về đúng thư mục package và tắt chế độ gộp thư mục rỗng trong cài đặt trình soạn thảo.
Ứng dụng báo lỗi không kết nối được tới SQL Server qua cổng 14333.	Bật giao thức TCP/IP trong SQL Server Configuration Manager, thiết lập cổng tĩnh 14333 tại mục IPAll và khởi động lại dịch vụ SQL Server.

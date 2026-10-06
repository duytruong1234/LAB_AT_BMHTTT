Lab 5: Thiết lập mô hình tường lửa pfSense

1. Họ và tên

Nguyễn Duy Trường

2. MSSV

1150070048

3. Tên Lab

Thiết lập mô hình tường lửa pfSense.

4. Phiên bản môi trường thực hành

VMware Workstation Pro 26H1u1, pfSense CE 2.7.2 (64-bit), RAM 2 GB, 2 vCPU, ổ đĩa 20 GB và 3 card mạng WAN, LAN, DMZ.

5. Cách dựng môi trường

Em tạo máy ảo pfSense bằng VMware Workstation Pro. Card WAN dùng VMnet0 ở chế độ Bridged, LAN dùng VMnet1 ở chế độ Host-only và DMZ dùng LAN segment dmz-net. Máy thật dùng IP 10.0.0.100/8 để quản trị pfSense, không đặt gateway và DNS.

6. Các tình huống thực hành

Bật và tắt rule cho LAN ra Internet.

Chặn ping nhưng vẫn cho phép DNS và truy cập Web.

Chỉ cho một máy trong LAN ra Internet.

Chặn DMZ truy cập LAN nhưng vẫn cho DMZ ra Internet.

Chuyển tiếp cổng từ WAN vào máy Web trong DMZ.

Bật logging và kiểm tra các gói tin bị chặn.

7. Kết quả PASS/FAIL

Chưa kiểm thử. Em sẽ bổ sung kết quả thực tế sau khi hoàn thành cấu hình và thực hiện từng tình huống.

8. Lỗi gặp phải và cách khắc phục

Lỗi: Các mục trong Virtual Network Editor bị mờ, không chỉnh sửa được.

Khắc phục: Em bấm Change Settings và đồng ý cấp quyền quản trị để chỉnh cấu hình mạng.

Lỗi cấu hình: Card mạng thứ hai của pfSense ban đầu chọn VMnet0.

Khắc phục: Em đổi card thứ hai sang VMnet1 để kết nối đúng mạng LAN.

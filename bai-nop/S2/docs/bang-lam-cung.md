# Bảng làm cứng, lab S2

Tên: *(điền)* · Mã số sinh viên: *(điền)* · Ngày nộp: *(điền)*

Mỗi dòng là một thay đổi bạn đã làm, không phải một thay đổi bạn định làm. Cần ít
nhất bốn dòng, và trong đó ít nhất một dòng dẫn về kỹ thuật T1548.001 của ATT&CK
v19.2 và ít nhất một dòng dẫn về một mã trong CWE 4.20.

Ba cột nặng nhẹ khác nhau. Cột thứ nhất dễ nhất, vì nó chỉ đòi bạn đọc tài liệu.
Cột thứ hai đòi bạn nghĩ như người phải trực hệ thống ấy sáu tháng nữa. Cột thứ ba
đòi bạn nghĩ như người tấn công vẫn còn đường vào sau khi bạn đã vá, và nó là cột
hay bị bỏ trống nhất.

| Thay đổi đã làm | Chặn được điều gì, dẫn ATT&CK v19.2 hoặc CWE 4.20 | Làm hỏng điều gì, tính bằng thao tác thủ công mỗi tuần | Thứ vẫn còn qua được sau khi đã sửa |
|---|---|---|---|
| Chạy dịch vụ bằng tài khoản không đặc quyền `sv` (uid 10001) thay vì root trong compose | Chặn việc thực thi với đặc quyền thừa, CWE-250 (Execution with Unnecessary Privileges); mọi lỗ hổng giờ chạy trong quyền của sv chứ không phải root | Không tốn thao tác thủ công hằng tuần; thỉnh thoảng (khoảng 0,5 giờ/tháng) phải đọc log để tự xử lý vì không còn quyền root sửa tức thì | Nếu kẻ tấn công chiếm được một lỗ hổng RCE trong dịch vụ, chúng vẫn nằm trong không gian của sv: có thể đọc dữ liệu sv đọc được và ghi nhật ký, nhưng phải có thêm một lỗ hổng khác để leo tiếp lên root |
| Bỏ toàn bộ năng lực mặc định bằng `cap_drop: ALL` rồi chỉ cấp lại `NET_BIND_SERVICE` | Chặn các năng lực nguy hiểm như cap_sys_admin, cap_dac_override, cap_net_raw mà tiến trình root mặc định mang, khiến chúng không còn hiệu lực khi bị chiếm | Không có thao tác thủ công hằng tuần nào phát sinh | Attacker bị chiếm dịch vụ chỉ còn quyền của sv và năng lực mở cổng dưới 1024: có thể mở thêm socket để duy trì, nhưng không đọc được tệp root và không leo quyền |
| Đặt `no-new-privileges: true` cho toàn bộ cây tiến trình con | Chặn đường leo quyền qua tệp setuid, kỹ thuật T1548.001 của ATT&CK v19.2 (Abuse Elevation Control Mechanism / setuid) | Không phát sinh thao tác tuần, nhưng nếu sau này có một tiến trình con cần setuid hợp pháp thì nó cũng bị chặn luôn, phải xử lý riêng | Cửa leo quyền qua setuid đã đóng, nhưng nếu ứng dụng còn lỗ hổng cho phép thực thi lệnh thì attacker vẫn chạy được lệnh dưới quyền sv; chúng chỉ không đổi được sang root qua setuid |
| Đặt hệ tệp gốc `read_only` và chỉ mở đúng một lối ghi là volume `/var/log/css-s02` | Chặn việc ghi đè tệp nhị phân, tệp cấu hình hay cài cắm duy trì vào hệ tệp gốc, liên quan CWE-732 về gán quyền sai tài nguyên trọng yếu | Mỗi lần cập nhật hoặc vá phải dựng lại ảnh thay vì sửa trong container, quy ra khoảng 0,5 giờ mỗi tháng cho người vận hành | Vẫn còn đúng một chỗ ghi là `nhat-ky:/var/log/css-s02`; attacker chiếm được sv có thể ghi rác vào nhật ký để đánh lừa giám sát, và read_only ở tầng container không cản được nội dung ghi vào volume |
| Bỏ bit setuid của `/usr/local/bin/doc-bimat` (4755 -> 0755) và thu quyền tệp cấu hình từ 666 xuống 640 trong Dockerfile | Chặn cả nhánh nâng quyền bằng tệp setuid (T1548.001) và chặn mọi tài khoản ghi đè cấu hình, dẫn CWE-732 (Incorrect Permission Assignment for Critical Resource) | Không phát sinh thao tác hằng tuần | Kẻ chiếm được sv không còn có sẵn một tệp setuid để nâng quyền; tuy nhiên cấu hình 640 chỉ chặn người ngoài nhóm, nên người thuộc nhóm root trong container vẫn đọc được nó, và tệp bí mật 600 vẫn chỉ root đọc được |

## Một câu về thứ bạn quyết định không làm

Tôi cân nhắc đổi hẳn cổng dịch vụ sang cổng không đặc quyền (ví dụ 8080) để khỏi phải cấp lại năng lực NET_BIND_SERVICE nào cả. Tôi bỏ phương án này vì đổi cổng buộc phải cập nhật tệp cấu hình, ánh xạ cổng trong compose và thông báo cho người dùng thay đổi luồng truy cập; phần rủi ro còn lại của một năng lực NET_BIND_SERVICE là rất nhỏ so với chi phí vận hành đó, nên theo nguyên lý thứ bảy tôi giữ cổng 80 và chấp nhận đúng một năng lực này.

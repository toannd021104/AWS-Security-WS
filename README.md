

# AWS Security Workshop

Website này tổng hợp các lab thực hành chi tiết về các dịch vụ bảo mật AWS, giúp bạn hiểu và vận dụng các giải pháp bảo vệ hệ thống trên nền tảng AWS.

## Nội dung các lab

1. **Security Hub**
	- Tổng hợp và quản lý các cảnh báo bảo mật từ nhiều dịch vụ AWS.
	- Hướng dẫn cách xem, lọc, tạo insight và xử lý findings.

2. **GuardDuty**
	- Phát hiện mối đe dọa tự động dựa trên phân tích hành vi và dữ liệu log.
	- Thực hành kích hoạt, kiểm tra findings, tạo danh sách đe dọa tùy chỉnh.

3. **Inspector**
	- Quét lỗ hổng bảo mật trên EC2, container và các tài nguyên cloud.
	- Hướng dẫn cấu hình, chạy scan, xem kết quả và xử lý findings.

4. **Detective**
	- Phân tích, điều tra sự kiện bảo mật, truy vết nguyên nhân sự cố.
	- Thực hành tạo graph, phân tích mối liên hệ giữa các sự kiện.

5. **Tích hợp đối tác & tổng hợp nhiều tài khoản**
	- Hướng dẫn kết nối các giải pháp bảo mật của đối tác vào Security Hub.
	- Thực hành tổng hợp findings từ nhiều tài khoản và vùng AWS.

## Mục tiêu
- Giúp người học hiểu rõ cách vận hành, cấu hình và khai thác các dịch vụ bảo mật AWS.
- Nâng cao kỹ năng thực hành bảo mật cloud qua các ví dụ thực tế.
- Tối ưu hóa quy trình phát hiện, phân tích và xử lý sự cố bảo mật.

## Hướng dẫn sử dụng
- Xem chi tiết từng lab trong thư mục `content/`
- Chạy thử website bằng lệnh: `hugo server -D` và truy cập http://localhost:1313
- Mỗi lần push code, website sẽ tự động build và deploy lên GitHub Pages qua GitHub Actions.

---
Tác giả: Nguyen Duc Toan

# 1. Giới thiệu về Firewalld
# 2. Các khái niệm cơ bản trong Firewalld
# 3. Cài đặt Firewalld
# 4. Cấu hình Firewalld

=======================================================================================

# 1. Giới thiệu về Firewalld
## 1.1 Khái niệm
Firewalld là giải pháp tường lửa mạnh mẽ, toàn diện được cài đặt mặc định trên CentOS/RHEL 7, nhằm thay thế Iptablés với những khác biệt cơ bản:
- Firewalld sử dụng "zones" và "services" thay vì "chain" và "rules" trong Iptables
- Firewalld quản lý các quy tắc được thiết lập tự động có tác dụng ngay lập tức mà không làm mất đi các kết nối và session hiện có 

# 2. Các khái niệm cơ bản trong Firewalld
## 2.1 Zone
Trong Firewalld, zone là một nhóm các quy tắc nhằm chỉ ra những luồng dữ liệu được cho phép, dựa trên mức độ tin tưởng của điểm xuất phát luồng dữ liệu đó trên hệ thống mạng. 
Các zone được xác định theo mức độ tin cậy, theo thứ tự từ ít tin cậy nhất đến đáng tin cậy nhất
- drop: Ít tin cậy nhất, toàn bộ các kết nối đến bị từ chối mà không phản hồi, chỉ cho phép duy nhất kết nối đi ra
- block: Tương tự drop nhưng các kết nối đến bị từ chối và phản hồi bằng tin nhắn từ icmp-host-prohibited (hoặc icmp6-adm-prohibited)
- public: Đại diện cho mạng công cộng, không đáng tin cậy nhưng vẫn cho phép các kết nối đến trên cơ sở chọn từng trường hợp cụ thể
- external: Hệ thống mạng bên ngoài trong trường hợp sử dụng tường lửa làm gateway, được cấu hình NAT để giữ bảo mật nội bộ mà vẫn có thể truy cập 
- internal: Sử dụng cho phần bên trong của gateway. Các máy tính/dịch vụ thuộc zone này thì khá đáng tin cậy
- dmz: Sử dụng cho các máy tính/service trong khu vực DMZ(Demilitarized), các máy tính bị cô lập sẽ không có quyền truy cập vào phần còn lại của hệ thống mạng, chỉ cho phép một số kết nối nhất định 
- work: Sử dụng trong công việc, tin tưởng hầu hết các máy tính và thêm một vài service được hoạt động 
- trusted: Đáng tin cậy nhất, tin tưởng toàn bộ thiết bị trong hệ thống
## 2.2 Quy tắc Runtime/Permanent
- Runtime(default): Có tác dụng ngay lập tức, mất hiệu lực khi reboot hệ thống
- Permanent : Không áp dụng cho hệ thống đang chạy, cần reload mới có hiệu lực, tác dụng vĩnh viễn cả khi reboot hệ thống 
Ví dụ: Thêm dịch vụ http với zone = public
```sh
firewall-cmd --zone=public --add-service=http
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```
Việc reload/restart sẽ hủy bỏ các thiết lập Runtime đồng thời áp dụng thiết lập Permanent mà không hề phá vỡ các kết nối và session hiện tại

# 3. Cài đặt Firewalld
Firewalld được cài đặt mặc định trên CentOS 7
- Nếu chưa có
```sh
yum install -y firewalld
```
- Khởi động firewalld 
```sh
systemctl start firewalld
```
- Kiểm tra tình trạng hoạt động 
```sh
systemctl status firewalld 
```
- Thiết lập Firewalld khởi động cùng hệ thống
```sh
systemctl enable firewall 
```
- Kiểm tra lại
```sh
systemctl is-enabled firewalld
```
- Khởi động lại firewalld
```sh
systemctl restart firewalld
firewall-cmd --reload
```
- Dừng firewalld
```sh
systemctl stop firewalld
```
- Vô hiệu hóa firewalld
```sh
systemctl disable firewalld
```
# 4. Cấu hình Firewalld
## 4.1 Thiết lập các zone trong hệ thống
- Liệt kê các zone trong hệ thống
```sh
firewall-cmd --get-zones
```
- Kiểm tra zone mặc định 
```sh
firewall-cmd --get-default-zone
```
- Kiểm tra zone active
```sh
firewall-cmd --get-active-zones
```
- Thay đổi zone mặc định, ví dụ thành work
```sh
firewall-cmd --set-default-zone=work
```
## 4.2 Các lệnh liệt kê
- Liệt kê toàn bộ các quy tắc của zones
```sh
firewall-cmd --list-all-zones
```
- Liệt kê danh sách service/port được cho phép trong zone
```sh
firewall-cmd --zone=public --list-services
firewall-cmd --zone=public --list-ports
```
## 4.3 Thiết lập cho Service
- Xác định service trên hệ thống
```sh
firewall-cmd --get-services
```
- Thiết lập cho phép service trên hệ Firewalld, sử dụng -add-service:
```sh
firewall-cmd --zone=public --add-service=http
firewall-cmd --zone=public --add-service=http --permanent
```
- Kiểm tra lại 
```sh
firewall-cmd --zone=public --list-services
```
- Vô hiệu hóa services trên firewalld, sử dụng -remove-service:
```sh
firewall-cmd --zone=public --remove-service=http
firewall-cmd --zone=public --remove-service=http --permanent
```
## 4.4 Thiết lập cho Port
- Mở port với tham số -add-port
```sh
firewall-cmd --zone=public --add-port=80/tcp
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
- Mở 1 dải port
```sh
firewall-cmd --zone=public --add-port=80-100/tcp
firewall-cmd --zone=public --add-port=80-100/tcp --permanent
```
- Kiểm tra lại
```sh
firewall-cmd --zone=pulic --list-ports
```
- Đóng port với tham số -remove-port
```sh
firewall-cmd --zone=public --remove-port=80/tcp
firewall-cmd --zone=public --remove-port=80/tcp -permanent

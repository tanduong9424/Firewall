#  I. Tổng quan đề tài
## 1. Giới thiệu mô hình
Đề tài thực hiện việc tìm hiểu cách cài đặt, triển khai Firewall bằng pfSense, cách đặt luật, cài đặt DMZ cũng như kết hợp hệ thống cảnh báo phát hiện xâm nhập IDS/IPS trên nền tảng ảo hóa VMWare. Đề tài được xây dựng theo mô hình bao gồm:
* Firewall pfSense dùng 3 card mạng (NAT, VMnet2, VMnet3).
* DMZ dùng dùng hệ điều hành Windows 10 với 1 card mạng (VMnet3).
* Web Server dùng Windows Server 2012 với một card mạng (VMnet2).
* Máy Client dùng hệ điều hành Windows 10 với card mạng (NAT).

![Mô hình mạng](img/topology.png)
<img src="img/topology.png" />
## 2. Công việc sẽ thực hiện
Công việc sẽ thực hiện bao gồm:
* Triển khai một Website cơ bản để demo cho chức năng của Web Server.
* Triển khai DMZ.
* Tiến hành Port Forward để NAT lớp mạng NAT vào lớp mạng LAN.
* Tiến hành tìm hiểu các rule cơ bản trên firewall.
* Cài đặt hệ thống IDS/IPS.
# II. Tiến hành
## 1. Triển khai Web Server Local
* Trên Windows Server 2012 truy cập vào Server Manager, chọn Manage -> Add roles and featuers
* Tiếp tục thực hiện các bước đến khi chọn gói cài đặt, ta chọn gói Web Server (ISS) để cài.
![](img/b1.png)

* Khi đã cài hoàn tất, ta vào Tool và vào ISS vừa mới cài xong. Bên cột bên trái ta tiến hành tạo một web site mới với tên, đường dẫn đến file index.html, địa chỉ IP và port được điền đầy đủ, có thể tham khảo hình dưới.   
![](img/b2.png)

* Ở đề tài này, website được thiết kế sẵn, bạn có thể tham khảo các website mẫu khác để thực hiện demo.
![](img/b3.png)

* Sau khi xong các bước cài đặt website ta tiến hành chạy thử bằng cách chọn Browse Website bên cột phải.
![](img/b4.png)

 * Hoặc đơn giản hơn là dùng trình duyệt và truy cập `https://IP_SERVER:PORT` như ở lab đã cấu hình thì truy cập `https://192.168.10.10:8081` ta được kết quả tương tự như hình.
 ![](img/b5.png)

## 2. Triển khai DMZ
## 3. Tiến hành Port Forward để NAT lớp mạng NAT vào lớp mạng LAN.
## 4. Tiến hành tìm hiểu các rule cơ bản trên firewall.
## 5. Cài đặt hệ thống IDS/IPS.
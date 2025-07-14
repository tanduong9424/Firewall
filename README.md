# Tìm hiểu Firewall, IDS/IPS trên VMWare
#  I. Tổng quan đề tài
## 1. Giới thiệu mô hình
Đề tài thực hiện việc tìm hiểu cách cài đặt, triển khai Firewall bằng pfSense, cách đặt luật, cài đặt DMZ sử dụng Load Balancing HAProxy cũng như kết hợp hệ thống cảnh báo phát hiện xâm nhập IDS/IPS trên nền tảng ảo hóa VMWare. Đề tài được xây dựng theo mô hình bao gồm:
* Firewall pfSense dùng 3 card mạng (NAT, VMnet2, VMnet3).
* DMZ dùng 2 máy ảo hệ điều hành CentOS 7 với 1 card mạng mỗi máy (VMnet3).
* Web Server dùng Windows Server 2012 với một card mạng (VMnet2).
* Máy Client dùng hệ điều hành Windows 10 với card mạng (NAT) có thể dùng máy vật lý để tối ưu không gian lưu trữ.

![Mô hình mạng](img/topology.png)

Tải và cài đặt file img pfSense tại `https://atxfiles.netgate.com/mirror/downloads/`

## 2. Công việc sẽ thực hiện
Công việc sẽ thực hiện bao gồm:
* Triển khai một Website cơ bản để demo cho chức năng của Web Server để chứng tỏ rằng mô hình này Web Server có thể được dùng để triển khai làm server nội bộ hoặc để quản lý DMZ.
* Tiến hành thử nghiệm chức năng Port Forward để NAT lớp mạng NAT vào lớp mạng LAN.
* Tắt tính năng Port Forward, dựng 2 WebServer trên 2 máy ảo CentOS7.
* Triển khai DMZ kết hợp Load Balancing bằng HAProxy với 2 node chạy CentOS 7 với chức năng chịu tải cho Website.
* Cài đặt hệ thống IDS/IPS.
* 
# II. Tiến hành thực hiện
## 1. Port Forwarding
### a) Triển khai Web Server Local trên Windows Server
* Trên Windows Server 2012 truy cập vào Server Manager, chọn **Manage** -> **Add roles and featuers**
* Tiếp tục thực hiện các bước đến khi chọn gói cài đặt, ta chọn gói Web Server (ISS) để cài.
![Bước 1](img/b1.png)

* Khi đã cài hoàn tất, ta vào Tool và vào ISS vừa mới cài xong. Bên cột bên trái ta tiến hành tạo một web site mới với tên, đường dẫn đến file index.html, địa chỉ IP và port được điền đầy đủ, có thể tham khảo hình dưới.   
![Bước 2](img/b2.png)

* Ở đề tài này, website được thiết kế sẵn, bạn có thể tham khảo các website mẫu khác để thực hiện demo.
![Bước 3](img/b3.png)

* Sau khi xong các bước cài đặt website ta tiến hành chạy thử bằng cách chọn Browse Website bên cột phải.
![Bước 4](img/b4.png)

 * Hoặc đơn giản hơn là dùng trình duyệt và truy cập `https://IP_SERVER:PORT` như ở lab đã cấu hình thì truy cập `https://192.168.10.10:8081` ta được kết quả tương tự như hình.
 ![Demo Website](img/b5.png)

### b) Tiến hành Port Forward để NAT lớp mạng NAT vào lớp mạng LAN.
#### b.1 Cấu hình
 Truy cập vào **Firewall** -> **NAT** và tiến hành Add theo các cấu hình dưới:
```
    interface WAN
    Address Family IPv4
    Protocol TCP/UDP
    Source Any
    Source port range Any
    Destination port range HTTP to HTTP
    Redirect target IP 
        Type Address or Alias 192.168.10.10
    Redirect target port
        Other 8080
```
 Ta sẽ được kết quả là một Rule mới trong NAT như hình:
 ![Rule NAT Forward](img/NATForward.png)

 Tiếp theo truy cập vào **Interfaces** -> **WAN** và bỏ chọn mục `Block private networks and loopback address` như hình:
 ![Reserved Networks](img/ReservedNet.png)
* `Block private networks and loopback address` : 
IP Private là những IP được cấp trong mạng LAN ví dụ 192.168.x.x/24, 10.x.x.x/8, 172.16.x.x/16. Nhưng việc những IP xuất hiện trên mạng internet dưới dạng IP Public là một điều bất thường nên Rule này sẽ chặn nhưng traffic đến card mạng WAN với source address là những IP private kể trên. 
* Việc **bỏ chọn** `Block private networks and loopback address` là để thuận tiện cho việc demo bởi lớp mạng WAN là 192.168.29.0/24 được cấp bằng DHCP của card mạng NAT trên VMWare. Tuy nhiên trong thực tế nên áp dụng Rule này.
* `Block bogon networks` : IP bogon là những địa chỉ IP không hợp lệ, có thể là những dãy IP không được IANA (Internet Assigned Numbers Authority) cấp phát hoặc những dãy được dùng với mục đích cụ thể không phải để định danh như Multicast,TEST-NET-1, TEST-NET-2, TEST-NET-3,...
* Việc **chọn** `Block bogon networks` hoàn toàn không ảnh hưởng đến kết quả demo và hoàn toàn phù hợp với thực tế.


Bước tiếp theo ta truy cập vào **Firewall** -> **RULES** chọn card **WAN** và tiến hành add rule theo các cấu hình dưới:
```
    Rule 1: Allow inbound Web Traffic to Server

    Action Pass
    Interface WAN
    Address Family IPv4
    Protocol TCP/UDP
    Source Any
    Source Port Range any
    Destination WAN address
    Destination Port Range
        other 8080 to 8080
```
```
    Rule 2: Bất kỳ traffic nào (TCP/UDP) đến cổng WAN 
    ở port 80, hãy chuyển nó vào 192.168.10.10 ở port 
    8080.

    Action Pass
    Interface WAN
    Address Family IPv4
    Protocol TCP/UDP
    Source Any
    Source Port Range any
    Destination 
        Address or Alias 192.168.10.10
    Destination Port Range
        other 8080 to 8080
```
Sau khi cấu hình xong ở Rule sẽ như hình:
![Rule table](img/RulePortForward.png)
#### b.2 Kiểm tra cấu hình
Sau khi cấu hình NAT port Forward rồi thì Client từ ngoài Internet dùng card mạng NAT có thể truy cập đến Website đặt trong mạng LAN như hình dưới IP `192.168.29.131` là IP card mạng NAT trên Firewall.
![test Port Forward](img/webNat.png)
Vì chưa cấu hình DMZ nên Firewall sẽ redirect sang trực tiếp WebServer nội bộ.
![Chưa có DMZ](img/NATChuaCoDMZ.png)

✍️ Như vậy mục này ta đã kiểm thử được cách truy cập từ mạng Internet (WAN) vào trong Web Server (LAN) bằng Port Forwading.

## 3. Triển khai DMZ kết hợp Load Balancing
### a. Cấu hình Web Server trên CentOS bằng Apache
Vì CentOS 7 đã ngừng hỗ trợ nên việc update các package cũng như cài các dịch vụ mới sẽ gặp rắc rối. Bạn đọc có thể tham khảo cách dưới được tổng hợp theo nguồn Internet, phục vụ cho việc demo.

[Cách cấu hình CentOS để update và cài đặt gói mới](https://github.com/tanduong9424/trick/blob/main/README.md)

Sau khi update hoàn tất, tiến hành cài đặt apache theo câu lệnh ``yum install httpd -y``

Thực hiện cấu hình lại file ``nano /etc/httpd/conf/httpd.conf``, thêm vào cuối file câu:
```
IncludeOptional conf.d/*.conf
NameVirtualHost *:80
```

Tiếp theo ta tạo mới file example.conf để cấu hình ``nano /etc/httpd/conf.d/example.conf`` theo mẫu dưới:
```
<VirtualHost *:80>
	ServerAdmin webmaster@htd.edu.vn
	DocumentRoot /home/htd/Desktop/web/WebBanAoDaBanh/index
	ServerName htd1.vn
	ServerAlias fit.htd1vn
	<Directory /home/htd/Desktop/web/WebBanAoDaBanh/index>
	        Options Indexes FollowSymLinks
	        AllowOverride All
	        Require all granted
	 </Directory>
</VirtualHost>
```
**Chú ý:**
* ServerAdmin là email, thông tin của chủ Website.
* DocumentRoot là đường dẫn tới file **index.html** thường sẽ là ``www/root/html``
* ServerName là tên miền (nếu có).
* ServerAlias là Alias cho tên miền.

Sau khi điều chỉnh hoàn tất ta tiến hành gán quyền cho và restart Apache.
```
sudo chown -R htd:htd /home/htd/Desktop/web/WebBanAoDaBanh
sudo chmod -R 755 /home/htd/Desktop/web/WebBanAoDaBanh
chmod +x /home/htd/Desktop/web/WebBanAoDaBanh/index/index.html

chmod o+x /home
chmod o+x /home/htd
chmod o+x /home/htd/Desktop
chmod o+x /home/htd/Desktop/web
chmod o+x /home/htd/Desktop/web/WebBanAoDaBanh

systemctl enable httpd
systemctl start httpd
systemctl restart httpd
```
### b. Cài đặt, cấu hình Load Balancing
Truy cập vào giao diện quản lý pfSense, truy cập **System**->**Package Manager**->**Available Packages**-> Tìm kiếm gói **HAProxy** và cài đặt.

Khi cài xong truy cập **Services**->**HAProxy** để tiến hành cấu hình.

Ở phần **Settings** ta tích vào để bật HAPRoxy như hình và kéo xuống cuối cùng để **SAVE** :
![bật HAProxy](img/EnableHAProxy.png)

Bên tab **BackEnd**, ta tạo mới một Pool, đây sẽ là nơi chứ các node cho Load Balancing, như trong mô hình đã đề cập, ta tạo 2 node ``192.168.20.10:80`` và `` 192.168.20.20:80`` như hình dưới:

![bật HAProxy](img/Backend1.png)

Đồng thời ta có thể chọn thuật toán Load Balancing ở tab ``Loadbalancing options``, ở đây để đơn giản ta có thể chọn Round Robin.
![thuật toán](img/thuattoanLB.png)

Bên tab **FrontEnd**, ta tạo mới một Frontend, đây là nơi mà HAProxy có thể nhận luồng traffic rồi từ đó phân chia request về các node. Theo mô hình chúng ta đã đề cập thì Fontend ở đây là card mạng WAN trên Firewall.

Cấu hình cụ thể:
![FrontEnd](img/Frontend1.png)

Chú ý ở mục này ta sẽ chọn Pool đã tạo ở bước vừa tạo bên trên, còn lại các lựa chọn khác có thể để như mặc định.
![FrontEnd](img/Frontend2.png)

### c. Cấu hình Rule để thiết lập DMZ
Với mô hình đã đề cập, có thể hình dung các luồng traffic như sau:
![Luồng traffic](img/flowAccessDMZ.png)
* Windows Server trong mạng LAN có thể dùng để chứa các dịch vụ nội bộ, cũng có thể dùng để quản lý DMZ nên, traffic đi từ LAN đến DMZ được pass.
![Rule LAN](img/ruleLan.png)

* DMZ dùng để chứa các dịch vụ công cộng, cho user từ internet truy cập và sử dụng, nên nếu DMZ có bị hacker tấn công thì luồng traffic không được đến trong lớp mạng LAN nên buộc phải block traffic từ DMZ đến LAN.

* DMZ vẫn có thể truy cập ngược ra internet để có thể respond request của user khi sử dụng dịch vụ, nên traffic từ DMZ đến WAN được pass. Ở rule cho DMZ truy cập internet thì ``!LAN subnets`` nghĩa là đối tượng được áp dụng là toàn bộ lớp mạng **ngoại trừ** lớp mạng **LAN**
![Rule DMZ](img/ruleDMZ.png)

* Client từ mạng internet truy cập vào Website sẽ được điều hướng vào card mạng WAN, vào đến mạng WAN, HAProxy sẽ thực hiện điều tiết Load Balancing, chuyển request đến các node trong pool, việc điều tiết request HAProxy sẽ thực hiện, không cần cấu hình bằng rule, việc cần cấu hình là điều hướng request vào card mạng WAN.
![Rule WAN](img/ruleWAN.png)

### d. Kiểm tra kết quả
Sau khi cài đặt xong ta sẽ tiến hành truy cập vào WebSite từ bên ngoài mạng WAN vào bằng cách truy cập vào IP card mạng WAN trên Firewall, trong mô hình này ta sẽ truy cập vào ``192.168.29.131:80``. 

Đồng thời để phân biệt 2 node, ta sẽ để header website ở node 1 màu đỏ, header website node thứ 2 sẽ có màu xanh nước. Và kết quả ta được như dưới hình:
![node1](img/webNode1.png)
![node2](img/webNode2.png)

✍️Kết quả thử nghiệm cho thấy các Rule và Load balancing đã được cấu hình chính xác, đúng như kết quả mong đợi.
## 4. Cài đặt hệ thống IDS/IPS.

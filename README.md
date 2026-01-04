# Triển khai và Giám sát Hệ thống Mạng với Splunk (SIEM)

## 1. Tổng Quan Đề Tài
Đồ án tập trung xây dựng một giải pháp **SIEM (Security Information and Event Management)** sử dụng **Splunk Enterprise**. Hệ thống có khả năng thu thập log tập trung từ nhiều thành phần mạng khác nhau (Firewall, Server, Container), phân tích dữ liệu và đưa ra cảnh báo thời gian thực về các mối đe dọa bảo mật.

**Mục tiêu chính:**
* Giám sát toàn diện hệ thống mạng doanh nghiệp giả lập.
* Phát hiện sớm các hành vi bất thường (truy cập trái phép, tấn công DDoS, lỗi hệ thống).
* Tự động hóa quy trình cảnh báo tới quản trị viên.

## 2. Kiến Trúc Hệ Thống (Network Topology)
Hệ thống mạng được chia thành 3 vùng bảo mật chính:
1.  **WAN (Internet):** Nơi mô phỏng các kết nối từ bên ngoài và Attacker.
2.  **DMZ (Demilitarized Zone):** Chứa Web Server (chạy trên Docker).
3.  **LAN (Mạng nội bộ):** Chứa Domain Controller (Windows Server), Splunk Server và Client PC.

Tất cả được bảo vệ và kiểm soát bởi **Sophos Firewall**.

![Sơ đồ kiến trúc hệ thống](topology.png)

## 3. Công Nghệ Sử Dụng (Tech Stack)
* **Core SIEM:** Splunk Enterprise (Indexer & Search Head).
* **Firewall:** Sophos XG Firewall.
* **Server:** Windows Server 2019 (Domain Controller), Ubuntu Linux.
* **Container:** Docker (Web Server Deployment).
* **Log Forwarders:** * Splunk Universal Forwarder.
    * Syslog (UDP 514).
    * HTTP Event Collector (HEC).

## 4. Chi Tiết Triển Khai

### 4.1. Thu thập Log từ Sophos Firewall
* **Cơ chế:** Sử dụng giao thức **Syslog** đẩy log qua cổng **UDP 514**.
* **Cấu hình:** Trên Sophos XG cấu hình Log Settings trỏ về IP của Splunk Server.
* **Phân tích:** Sử dụng *Splunk Add-on for Sophos Next-Gen Firewall* để chuẩn hóa dữ liệu (parsing fields).

### 4.2. Giám sát Windows Domain Controller (DC)
* **Cơ chế:** Cài đặt **Splunk Universal Forwarder** trên máy DC.
* **Dữ liệu:** Thu thập Security Log, System Log, và Application Log.
* **Use Case:** Phát hiện hành vi xóa tài khoản người dùng (User Account Deletion) hoặc đăng nhập thất bại liên tục.

### 4.3. Giám sát Docker & Web Server
* **Cơ chế:** Sử dụng **HTTP Event Collector (HEC)** qua cổng **8088**.
* **Triển khai:** Cấu hình Docker Log Driver để gửi log container trực tiếp về Splunk mà không cần cài agent lên từng container.
    ```
    # Ví dụ lệnh chạy container gửi log về Splunk
    docker run --log-driver=splunk \
      --log-opt splunk-url=[https://10.0.0.10:8088](https://10.0.0.10:8088) \
      --log-opt splunk-token=<HEC_TOKEN> \
      --log-opt splunk-insecureskipverify=true \
      my-web-server
    ```

### 4.4. Hệ thống Cảnh báo (Alerting)
* **Kênh thông báo:** Email (SMTP Gmail).
* **Cơ chế:** Splunk tự động gửi email cho quản trị viên ngay lập tức khi phát hiện sự kiện thỏa mãn điều kiện (VD: Có user bị xóa khỏi AD, hoặc phát hiện traffic từ IP đen).

## 5. Kết Quả & Dashboard
Dưới đây là hình ảnh Dashboard giám sát trực quan trên Splunk:

![Dashboard giám sát Splunk](dashboard.png)

## 6. Hướng Dẫn Cài Đặt Cơ Bản
1.  **Dựng Lab:** Sử dụng VMware/EVE-NG để dựng các máy ảo (Sophos, Windows DC, Ubuntu Splunk).
2.  **Cài đặt Splunk:** Cài Splunk Enterprise lên máy Ubuntu, mở port 8000 (Web), 9997 (Forwarder), 8088 (HEC), 514 (Syslog).
3.  **Cài đặt Agent:**
    * Cài Universal Forwarder lên Windows DC -> Trỏ về IP Splunk:9997.
    * Cấu hình Syslog trên Sophos -> Trỏ về IP Splunk:514.
4.  **Cài đặt Add-on:** Vào Splunkbase tải các Add-on tương ứng (Windows, Sophos, Docker) để Splunk hiểu định dạng log.

---
*Dự án được thực hiện bởi Nhóm 10 - UIT.*
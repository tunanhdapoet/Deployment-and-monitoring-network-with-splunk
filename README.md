# 🛡️ Triển Khai và Giám Sát Hệ Thống Mạng với Splunk (SIEM)

![Splunk](https://img.shields.io/badge/SIEM-Splunk_Enterprise-000000?logo=splunk&logoColor=white)
![Sophos](https://img.shields.io/badge/Firewall-Sophos_XG-174898?logo=sophos&logoColor=white)
![Docker](https://img.shields.io/badge/Container-Docker-2496ED?logo=docker&logoColor=white)
![Windows Server](https://img.shields.io/badge/Server-Windows_Server_2019-0078D6?logo=windows&logoColor=white)

> **Đồ án môn học: Đánh giá hiệu năng hệ thống mạng máy tính (NT531.P21)** > **Giảng viên hướng dẫn:** ThS. Đặng Lê Bảo Chương

Dự án xây dựng giải pháp **SIEM (Security Information and Event Management)** tập trung sử dụng **Splunk Enterprise**. Hệ thống thực hiện thu thập log từ đa nguồn (Firewall, AD Server, Docker Containers), chuẩn hóa dữ liệu và tự động phát hiện các hành vi bất thường trong thời gian thực.

## 🏗️ 1. Kiến Trúc Hệ Thống (Network Topology)

Hệ thống được thiết kế theo mô hình 3 vùng bảo mật (3-tier security zones), được bảo vệ bởi **Sophos XG Firewall**:

1.  **WAN Zone:** Mô phỏng Internet và các máy tấn công (Attacker).
2.  **DMZ Zone:** Chứa Web Server chạy trên nền tảng Docker Container.
3.  **LAN Zone:** Mạng nội bộ chứa Domain Controller (Windows Server), Splunk Server (Ubuntu) và Client.

![Network Topology](https://github.com/user-attachments/assets/3c7da685-eb32-4843-ad0d-4c9ce9f7dffa)

---

## 🛠️ 2. Công Nghệ & Giải Pháp (Tech Stack)

### Core SIEM
* **Splunk Enterprise:** Đóng vai trò Indexer và Search Head để lưu trữ và phân tích log.

### Log Sources & Forwarders
| Nguồn Log | Cơ chế thu thập | Cổng (Port) | Mô tả |
|:---|:---|:---|:---|
| **Sophos Firewall** | Syslog | UDP 514 | Đẩy log Traffic, IPS, System events về Splunk. |
| **Windows AD** | Universal Forwarder | TCP 9997 | Thu thập Security, System, Application logs qua Agent. |
| **Docker Container** | HTTP Event Collector (HEC) | TCP 8088 | Sử dụng Docker Log Driver gửi log trực tiếp qua HTTP. |

---

## ⚙️ 3. Chi Tiết Triển Khai (Implementation Details)

### 3.1. Giám sát Sophos Firewall
* **Cấu hình:** Trên Sophos XG, thiết lập **Log Settings** để gửi logs về địa chỉ IP của Splunk Server qua giao thức Syslog.
* **Parsing:** Cài đặt **Splunk Add-on for Sophos Next-Gen Firewall** trên Splunk để tự động trích xuất các trường thông tin (Src_IP, Dst_IP, Action, Rule_ID...).

### 3.2. Giám sát Windows Domain Controller
* **Agent:** Cài đặt **Splunk Universal Forwarder** trên Windows Server 2019.
* **Input:** Cấu hình file `inputs.conf` để theo dõi các Event Log quan trọng.
* **Use Case:** Giám sát hành vi tạo/xóa user trái phép, đăng nhập thất bại (Brute-force detection).

### 3.3. Giám sát Web Server (Docker)
* **Phương pháp:** Không cài agent vào container, sử dụng **Splunk Logging Driver** của Docker.
* **Lệnh triển khai mẫu:**
    ```bash
    docker run --log-driver=splunk \
      --log-opt splunk-url=https://<SPLUNK_IP>:8088 \
      --log-opt splunk-token=<HEC_TOKEN> \
      --log-opt splunk-insecureskipverify=true \
      -d my-web-server
    ```

### 3.4. Hệ Thống Cảnh Báo (Alerting)
* **Trigger:** Thiết lập các **Correlation Searches** (ví dụ: Phát hiện User bị xóa khỏi Active Directory).
* **Action:** Tự động gửi Email cảnh báo tới quản trị viên thông qua SMTP Gmail Server ngay khi sự kiện xảy ra.

---

## 📊 4. Dashboard & Kết Quả

Giao diện giám sát tập trung (Centralized Dashboard) hiển thị lưu lượng mạng, trạng thái các node và các cảnh báo bảo mật.

![Splunk Dashboard](https://github.com/user-attachments/assets/588a1af7-365c-4a37-bdfd-84432aa685ed)

---

## 🚀 5. Hướng Dẫn Cài Đặt Nhanh (Quick Start)

1.  **Chuẩn bị môi trường Lab:** Dựng các máy ảo (VMware/EVE-NG) gồm Sophos Firewall, Windows Server, Ubuntu (cho Splunk).
2.  **Cài đặt Splunk Enterprise:**
    * Tải và cài đặt Splunk trên Ubuntu.
    * Mở các port cần thiết trên Firewall OS: `8000` (Web), `9997` (Forwarder), `8088` (HEC), `514` (Syslog).
3.  **Cấu hình Data Inputs:**
    * Vào **Settings > Data Inputs** để kích hoạt UDP 514 và HTTP Event Collector (tạo Token).
4.  **Cài đặt Add-ons:**
    * Tải từ Splunkbase: *Splunk Add-on for Windows*, *Splunk Add-on for Sophos*, *Splunk Add-on for Docker*.
5.  **Kết nối Client:**
    * Cấu hình log forwarding trên các máy con (Firewall, AD, Docker host) trỏ về Splunk Server.

# Thiết lập Lab — Metasploitable 2 Pentest

## 1. Môi trường thực hành

| Thành phần | Cấu hình |
|---|---|
| Phần mềm ảo hóa | VMware Workstation |
| Máy kiểm thử (Attacker) | Kali Linux 2025.3 |
| Máy mục tiêu (Target) | Metasploitable 2 (Ubuntu 8.04) |
| RAM Kali Linux | 2 GB |
| RAM Metasploitable 2 | 1 GB |
| CPU Kali Linux | 4 vCPU |
| CPU Metasploitable 2 | 1 vCPU |
| Chế độ mạng | VMware LAN Segment |
| Tên mạng | PENTEST-LAB |

## 2. Cấu hình mạng

Hai máy ảo được kết nối vào cùng một mạng LAN Segment trên VMware, cách ly với mạng bên ngoài.

| Máy ảo | Địa chỉ IP | Vai trò |
|---|---|---|
| Kali Linux | 192.168.77.10/24 | Máy kiểm thử |
| Metasploitable 2 | 192.168.77.20/24 | Máy mục tiêu |

- **Subnet:** `192.168.77.0/24`
- **Subnet Mask:** `255.255.255.0`
- **Card mạng:** `eth0`
- **Default Gateway:** Không cấu hình

## 3. Kiểm tra kết nối

Sau khi cấu hình địa chỉ IP, tiến hành kiểm tra kết nối từ Kali Linux đến Metasploitable 2.

**Lệnh thực hiện:**

`ping -c 5 192.168.77.20`

**Kết quả:**

- Trạng thái: Kết nối thành công
- Số gói gửi: 5
- Số gói nhận: 5
- Tỷ lệ mất gói: 0%

**Nhận xét:**

Kali Linux đã kết nối thành công đến Metasploitable 2 thông qua mạng LAN Segment.

**File kết quả:** `setup/connectivity.txt`
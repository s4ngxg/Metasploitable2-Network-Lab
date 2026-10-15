## 1. Reconnaissance

### 1.1. Host Discovery

**Mục đích:** Tìm các máy đang hoạt động trong mạng lab.

**Lệnh:**

`sudo nmap -sn -PR 192.168.77.0/24`

- `-sn`: Chỉ tìm host, không quét port.
- `-PR`: Sử dụng ARP để phát hiện host.

**Kết quả:**

| IP | Thiết bị |
|---|---|
| 192.168.77.10 | Kali Linux |
| 192.168.77.20 | Metasploitable 2 |

**Nhận xét:** Phát hiện 2 host đang hoạt động. Xác định `192.168.77.20` là máy mục tiêu.

**Output:** [01-host-discovery.nmap](scans/01-host-discovery.nmap)

---

### 1.2. TCP Port Scanning

**Mục đích:** Tìm các cổng TCP đang mở trên máy mục tiêu.

**Lệnh:**

`sudo nmap -sS -p- -T3 --reason -n 192.168.77.20`

- `-sS`: Quét TCP SYN.
- `-p-`: Quét toàn bộ 65.535 port.
- `-T3`: Tốc độ quét mặc định.
- `--reason`: Hiển thị lý do xác định trạng thái port.
- `-n`: Không phân giải DNS.

**Kết quả:**

- Open: **30 ports**
- Closed: **65.505 ports**
- Thời gian: **7,14 giây**

Một số port đáng chú ý:

| Port | Service |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 80 | HTTP |
| 139, 445 | SMB / NetBIOS |
| 3306 | MySQL |
| 3632 | distccd |
| 5432 | PostgreSQL |
| 5900 | VNC |

**Nhận xét:** Máy mục tiêu mở nhiều dịch vụ mạng. Cần xác định phiên bản dịch vụ để tiếp tục phân tích lỗ hổng.

**Output:** [02-tcp-ports.nmap](scans/02-tcp-ports.nmap)

---

## 2. Service Enumeration
 
**Mục đích:** Sau khi tìm được các port đang mở, tiến hành xác định dịch vụ và phiên bản phần mềm trên các port TCP đã phát hiện.


**Lệnh:** `sudo nmap -sV -sC -n -p <PORTS> 192.168.77.20`

- `-sV`: Xác định phiên bản dịch vụ.
- `-sC`: Chạy các script mặc định.
- `-n`: Không phân giải DNS.

**Kết quả:**

Nmap phát hiện và nhận diện dịch vụ trên 30 cổng TCP đang mở.

| Port | Service | Version / Thông tin |
|---|---|---|
| 21 | FTP | vsftpd 2.3.4 |
| 22 | SSH | OpenSSH 4.7p1 |
| 23 | Telnet | Linux telnetd |
| 25 | SMTP | Postfix smtpd |
| 53 | DNS | ISC BIND 9.4.2 |
| 80 | HTTP | Apache 2.2.8 |
| 111 | RPC | rpcbind v2 |
| 139 | NetBIOS/SMB | Samba smbd 3.X–4.X |
| 445 | SMB | Samba 3.0.20-Debian |
| 512 | rexec | netkit-rsh rexecd |
| 513 | rlogin | Chưa xác định phiên bản |
| 514 | rsh | Netkit rshd |
| 1099 | Java RMI | GNU Classpath grmiregistry |
| 1524 | Bind Shell | Metasploitable root shell |
| 2049 | NFS | NFS v2–v4 |
| 2121 | FTP | ProFTPD 1.3.1 |
| 3306 | MySQL | MySQL 5.0.51a |
| 3632 | distccd | distccd v1 |
| 5432 | PostgreSQL | PostgreSQL 8.3.0–8.3.7 |
| 5900 | VNC | Protocol 3.3 |
| 6000 | X11 | Access denied |
| 6667 | IRC | UnrealIRCd |
| 6697 | IRC | UnrealIRCd |
| 8009 | AJP13 | Apache JServ Protocol 1.3 |
| 8180 | HTTP | Apache Tomcat 5.5 (theo HTTP title) |
| 8787 | DRb | Ruby DRb RMI (Ruby 1.8) |
| 49828 | NLM | nlockmgr v1–v4 |
| 54667 | RPC Status | RPC Status v1 |
| 54981 | Java RMI | GNU Classpath grmiregistry |
| 60895 | RPC Mount | mountd v1–v3 |

**Nhận xét:** Metasploitable 2 đang mở 30 cổng TCP, bao gồm nhiều dịch vụ cũ và một số cấu hình đáng nghi.

**Output:** [03-services.nmap](scans/03-services.nmap)
## 3. Vulnerability Analysis

### 3.1. Unauthenticated Root Bind Shell — Port 1524

**Lỗ hổng:** Dịch vụ Bind Shell trên port 1524 cho phép kết nối và thực thi lệnh từ xa với quyền `root` mà không cần xác thực.

**Nguyên nhân:** Một shell chạy với quyền `root` được mở trực tiếp ra mạng mà không có cơ chế kiểm tra danh tính người kết nối.

**Cơ chế:** Khi kết nối đến port 1524, các lệnh được gửi trực tiếp đến shell trên máy mục tiêu. Do shell đang chạy với UID 0 nên các lệnh cũng được thực thi với quyền `root`.

**Tác động:** Cho phép kiểm soát hệ thống với quyền cao nhất.

**Finding:** [F01 — Unauthenticated Root Bind Shell](findings/F01-1524-bindshell.md)

---

### 3.2. VSFTPD 2.3.4 Backdoor — Port 21

**Lỗ hổng:** CVE-2011-2523 — Backdoor trong một bản phân phối vsftpd 2.3.4 bị xâm phạm.

**Nguyên nhân:** Mã nguồn của một bản phát hành vsftpd 2.3.4 đã bị chèn mã độc, tạo ra backdoor cho phép truy cập hệ thống trái phép.

**Cơ chế:** Khi FTP server nhận username chứa chuỗi đặc biệt `:)`, backdoor được kích hoạt và mở một shell trên TCP port 6200.

Metasploit lợi dụng cơ chế này để kích hoạt backdoor qua FTP port 21, sau đó kết nối đến port 6200 để truy cập shell.

**Tác động:** Cho phép thực thi lệnh từ xa với quyền `root` mà không cần xác thực.

**Finding:** [F02 — VSFTPD 2.3.4 Backdoor](findings/F02-vsftpd-backdoor.md)

### 3.3. distccd Remote Command Execution — Port 3632

**Lỗ hổng:** CVE-2004-2687 — Remote Command Execution (RCE).

**Nguyên nhân:** distccd là dịch vụ hỗ trợ biên dịch chương trình từ xa. Khi dịch vụ cho phép máy không đáng tin cậy gửi yêu cầu biên dịch mà thiếu kiểm soát truy cập và kiểm tra lệnh phù hợp, kẻ tấn công có thể lợi dụng để thực thi lệnh tùy ý.

**Cơ chế:** Kẻ tấn công gửi yêu cầu đặc biệt đến distccd qua port 3632, khiến dịch vụ thực thi lệnh trên hệ thống. Lệnh được chạy với quyền của tài khoản đang vận hành distccd.

Trong trường hợp này, dịch vụ chạy với quyền `daemon` (`uid=1`), không phải `root`.

**Tác động:** Cho phép thực thi lệnh từ xa và truy cập các tài nguyên mà tài khoản `daemon` có quyền sử dụng.

**Finding:** [F03 — distccd RCE](findings/F03-distcc-rce.md)

### 3.4. Samba Username Map Script RCE — Port 445

**Lỗ hổng:** CVE-2007-2447 — Remote Command Execution thông qua tính năng `username map script` của Samba.

**Nguyên nhân:** Một số phiên bản Samba xử lý username không an toàn khi sử dụng `username map script`. Dữ liệu username do người dùng cung cấp có thể bị đưa vào lệnh shell mà không được kiểm tra đúng cách.

**Cơ chế:** Kẻ tấn công gửi username chứa các ký tự đặc biệt để chèn lệnh hệ thống vào quá trình thực thi script ánh xạ tài khoản. Lệnh được thực thi với quyền của tiến trình Samba.

Trong lab này, lỗ hổng cho phép thực thi lệnh với quyền `root`.

**Tác động:** Cho phép thực thi lệnh từ xa, có thể dẫn đến kiểm soát toàn bộ hệ thống.

**Finding:** [F04 — Samba Username Map Script RCE](findings/F04-samba-usermap-rce.md)

### 3.5. UnrealIRCd Backdoor — Port 6667

**Lỗ hổng:** CVE-2010-2075 — Backdoor Remote Command Execution.

**Nguyên nhân:** Một bản phân phối UnrealIRCd 3.2.8.1 bị chèn mã độc vào mã nguồn, tạo ra backdoor trong dịch vụ IRC.

**Cơ chế:** Khi nhận chuỗi đặc biệt bắt đầu bằng `AB;` qua kết nối IRC, backdoor có thể thực thi phần lệnh đi kèm trên hệ thống mà không cần xác thực.

Lệnh được thực thi với quyền của tiến trình UnrealIRCd. Trong lab này, quyền đạt được là `root`.

**Tác động:** Cho phép thực thi lệnh từ xa và có khả năng kiểm soát toàn bộ hệ thống.

**Finding:** [F05 — UnrealIRCd Backdoor](findings/F05-unrealircd-backdoor.md)

### 3.6. NFS Root Export Misconfiguration — Port 2049

**Lỗ hổng:** Cấu hình NFS không an toàn, cho phép truy cập thư mục gốc của hệ thống qua mạng.

**Nguyên nhân:** NFS export thư mục `/` với client `*`, không giới hạn máy truy cập theo địa chỉ host trong cấu hình chia sẻ.

**Cơ chế:** NFS cho phép các máy khác mount thư mục được export và truy cập dữ liệu dựa trên quyền của filesystem cùng cấu hình NFS. Việc chia sẻ toàn bộ thư mục gốc làm tăng nguy cơ lộ dữ liệu hệ thống.

**Tác động:** Có thể làm lộ cấu trúc thư mục và các file được phép đọc qua NFS. Mức độ truy cập thực tế còn phụ thuộc vào quyền file và cấu hình export.

**Finding:** [F06 — NFS Misconfiguration](findings/F06-nfs-misconfiguration.md)
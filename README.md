# Metasploitable 2 — Network Pentest

Thực hành **Network Penetration Testing** trên Metasploitable 2 bằng Nmap, Metasploit và một số công cụ khác.

Mục tiêu là tìm hiểu các lỗ hổng mạng và thực hành khai thác trong môi trường lab.

## Lab Environment

- **Attacker:** Kali Linux
- **Target:** Metasploitable 2
- **Platform:** VMware Workstation
- **Tools:** Nmap, Metasploit, Netcat

[Lab Setup](setup/lab-info.md) | [Writeup](writeup.md)

## Findings


| ID  | Vulnerability                                                | Result                 |
| --- | ------------------------------------------------------------ | ---------------------- |
| F01 | [Bind Shell](findings/F01-1524-bindshell.md)                 | Root Shell             |
| F02 | [VSFTPD Backdoor](findings/F02-vsftpd-backdoor.md)           | Root Shell             |
| F03 | [distccd RCE](findings/F03-distcc-rce.md)                    | Daemon Shell           |
| F04 | [Samba Usermap RCE](findings/F04-samba-usermap-rce.md)       | Root Shell             |
| F05 | [UnrealIRCd Backdoor](findings/F05-unrealircd-backdoor.md)   | Root Shell             |
| F06 | [NFS Misconfiguration](findings/F06-nfs-misconfiguration.md) | Root Directory Listing |

## Project Files

- `setup/` — Cấu hình lab.
- `scans/` — Kết quả quét.
- `findings/` — Chi tiết kiểm chứng từng lỗ hổng.
- `evidence/` — Log kiểm thử.
- `screenshots/` — Ảnh kết quả.
- `writeup.md` — Phân tích các lỗ hổng.

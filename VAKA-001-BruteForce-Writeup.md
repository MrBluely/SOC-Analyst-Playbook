# VAKA-001: Brute Force → RDP → Discovery

**Tarih:** 23 Ekim 2026, 03:14
**Analist:** [Emir Bayram]
**Severity:** P1 (High)

---

##  Executive Summary

23 Ekim 2026 gece 03:14'te DC-01 sunucusuna 185.220.101.47 IP adresinden 
47 başarısız giriş denemesi (Brute Force) yapılmış, ardından `svc_backup` 
hesabıyla RDP (Logon Type 10) üzerinden başarılı giriş sağlanmıştır. 
Saldırgan sistemde keşif komutları çalıştırmış ve PowerShell -enc ile 
uzaktan zararlı script indirmeye çalışmıştır. Sunucu izole edilmiş, 
hesap kapatılmış ve adli inceleme başlatılmıştır.

---

##  Detection

- SIEM Alarm: "Multiple Failed Logons Followed by Success"
- Kaynak IP: 185.220.101.47
- Hedef Sunucu: DC-01
- Hedef Hesap: svc_backup
- Tetiklenme: 03:14

---

##  Analysis

| Saat | Event ID | Açıklama |
|------|----------|----------|
| 03:02-03:13 | 4625 x47 | Başarısız giriş denemesi (Bad Password) |
| 03:14:22 | 4624 | Başarılı giriş, Logon Type 10 (RDP) |
| 03:14:45 | 4672 | Yetki yükseltme (SeDebugPrivilege, SeBackupPrivilege) |
| 03:15:03 | 4688 | Discovery: whoami, net user, ipconfig |
| 03:16:12 | 4688 | PowerShell -enc → payload.ps1 indirme girişimi 🚨 |

**Değerlendirme:** True Positive. Saldırgan jitter (8 sn aralık) kullanarak 
tespit edilmemeye çalışmış, başarılı giriş sonrası keşif yapmıştır.

---

##  Response

1. DC-01 sunucusu ağdan izole edildi.
2. svc_backup hesabı kilitlendi, tüm oturumlar sonlandırıldı.
3. Loglar ve bellek dökümü alındı (forensics).
4. Aynı IP diğer sunucularda arandı (tehdit avcılığı).
5. Üst yönetime ve KVKK'ya bildirim yapıldı.

---

##  Lessons Learned

- Servis hesapları (svc_*) için MFA zorunlu olmalı.
- RDP erişimi sadece VPN + MFA ile sağlanmalı.
- PowerShell -enc komutları için SIEM kuralı yazılmalı.
- Servis hesaplarının parola politikası güçlendirilmeli.

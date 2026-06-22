# Use-the-runtipi-as-a-server-on-your-own-computer
# 🏗️ Proje Destanı: Konteyner Tabanlı (Docker) Runtipi Ev Sunucusu Mimarisi

> **Bir Staj Defterinden Daha Fazlası: Sıfırdan İnşa Edilmiş, Üretim Seviyesinde Bir Self-Hosted Altyapı Vakası**

[![Status](https://img.shields.io/badge/Durum-Aktif%20%26%20Operasyonel-success?style=for-the-badge)]()
[![Platform](https://img.shields.io/badge/Platform-WSL2%20%7C%20Ubuntu-orange?style=for-the-badge&logo=linux)]()
[![Orchestration](https://img.shields.io/badge/Orchestration-Docker%20Compose-blue?style=for-the-badge&logo=docker)]()
[![Core](https://img.shields.io/badge/Core-Runtipi%20v4.10.1-purple?style=for-the-badge)]()

---

## 👤 Proje Sahibi

**RadmelaSub** — 19 günlük staj programı kapsamında, geleneksel "gözlem defteri" formatını terk ederek; *Altyapı Mühendisliği (Infrastructure Engineering)*, *Ağ Mimarisi (Network Architecture)* ve *Site Reliability Engineering (SRE)* disiplinlerini bir araya getiren uçtan uca bir self-hosted ekosistem inşa etmiştir.

---

## 📜 Yönetici Özeti (Executive Summary)

Bu doküman, sıfırdan kurulmuş, izole edilmiş ve gözlemlenebilir (*observable*) bir konteyner orkestrasyon ortamının mimari yolculuğunu anlatmaktadır. Proje; salt bir "kurulum" sürecinden ibaret olmayıp, **Altyapı Tasarımı**, **Ağ Güvenliği**, **Servis Orkestrasyonu**, **İzlenebilirlik (Observability)** ve **Felaket Kurtarma (Disaster Recovery)** prensiplerinin canlı bir sistem üzerinde uygulanmasıdır.

Sistem; WSL2 (Windows Subsystem for Linux) üzerinde çalışan bir Ubuntu dağıtımı temelinde, Docker konteynerizasyon teknolojisi ile dokuz farklı mikroservisi tek bir çatı altında, birbirinden izole ama birbiriyle entegre şekilde çalıştıracak biçimde tasarlanmıştır. Süreç boyunca karşılaşılan kritik sistem hataları, kök neden analizi (*Root Cause Analysis*) yöntemiyle çözülmüş ve her biri bu raporda ayrı bir mühendislik vakası olarak belgelenmiştir.

---

## 🗂️ İçindekiler

- [Faz 0: Mimari Vizyon ve Tasarım Felsefesi](#-faz-0-mimari-vizyon-ve-tasarım-felsefesi)
- [Faz 1: Temel Altyapının İnşası (Foundation Layer)](#-faz-1-temel-altyapının-inşası-foundation-layer)
- [Faz 2: Orkestrasyon Çekirdeği — Runtipi Entegrasyonu](#-faz-2-orkestrasyon-çekirdeği--runtipi-entegrasyonu)
- [Faz 3: Ağ Mimarisi ve Güvenlik Katmanı](#-faz-3-ağ-mimarisi-ve-güvenlik-katmanı)
- [Faz 4: Mikroservis Ekosistemi](#-faz-4-mikroservis-ekosistemi)
- [Faz 5: İzlenebilirlik ve Gözlemlenebilirlik (Observability)](#-faz-5-izlenebilirlik-ve-gözlemlenebilirlik-observability)
- [Faz 6: Felaket Kurtarma ve Veri Dayanıklılığı (Disaster Recovery)](#-faz-6-felaket-kurtarma-ve-veri-dayanıklılığı-disaster-recovery)
- [🔥 Kritik Mühendislik Vakaları (Troubleshooting Vakaları)](#-kritik-mühendislik-vakaları-troubleshooting-vakaları)
- [Edinilen Mühendislik Yetkinlikleri](#-edinilen-mühendislik-yetkinlikleri)
- [Sonuç ve Gelecek Vizyonu](#-sonuç-ve-gelecek-vizyonu)

---

## 🎯 Faz 0: Mimari Vizyon ve Tasarım Felsefesi

Bu projenin çıkış noktası basit bir soruydu: *"Bulut sağlayıcılarına bağımlı olmadan, veri egemenliğini elimde tutan, kendi kendine yeten bir dijital ekosistem inşa edebilir miyim?"*

Cevap, **High Availability (Yüksek Erişilebilirlik)** prensiplerini gözeten, **Defense in Depth (Katmanlı Savunma)** mantığıyla güvenlik halkaları örülmüş ve **Single Point of Failure (Tek Hata Noktası)** riskini en aza indiren bir mimari ile şekillendi.

Tasarım felsefesinin üç temel sütunu şu şekilde belirlendi:

1. **İzolasyon (Isolation):** Her mikroservis kendi konteyner sınırları içinde çalışır; bir servisteki güvenlik açığı diğerlerine sirayet edemez.
2. **Gözlemlenebilirlik (Observability):** Sistemin "kara kutu" olmaktan çıkarılıp, her katmanın (CPU, RAM, ağ trafiği, konteyner logları) gerçek zamanlı izlenebilir hale getirilmesi.
3. **Dayanıklılık (Resilience):** Veri kaybı senaryolarına karşı otomatik, şifreli ve zamanlanmış yedekleme stratejilerinin sisteme baştan entegre edilmesi.

---

## 🧱 Faz 1: Temel Altyapının İnşası (Foundation Layer)

### 1.1. İşletim Sistemi Katmanı Seçimi

Proje, Windows ortamında çalışan ancak Linux çekirdeğinin native performansından ödün vermeyen bir hibrit yaklaşım olan **WSL2 (Windows Subsystem for Linux 2)** üzerine konuşlandırılmıştır. Bu tercih, geliştirme ortamı esnekliği ile üretim benzeri (*production-like*) bir Linux çalışma zamanını aynı potada birleştirmek amacıyla yapılmıştır.

```bash
# Örnek terminal oturumu (placeholder kullanıcı/host bilgileriyle)
ornek_kullanici@ornek-sunucu:~$ lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 22.04 LTS
```

### 1.2. Konteynerizasyon Motorunun Kurulumu

Sistemin omurgasını oluşturan **Docker Engine** ve **Docker Compose**, paket bağımlılıkları ve sürüm uyumluluk matrisleri titizlikle kontrol edilerek kurulmuştur. Bu aşamada, `cgroups` (control groups) yapılandırmasının WSL2 çekirdeği ile uyumluluğu doğrulanmış, böylece konteyner kaynak limitlemesinin (CPU/RAM throttling) sağlıklı çalıştığı teyit edilmiştir.

```bash
ornek_kullanici@ornek-sunucu:~$ docker --version
ornek_kullanici@ornek-sunucu:~$ docker compose version
```

### 1.3. Dosya Sistemi Hiyerarşisi ve İzin Mimarisi

Linux dosya sistemi izin yapısı (`rwx` permission triad), her servisin volume mount'ları için ayrı ayrı planlanmıştır. Bu, ilerleyen fazlarda kritik bir mühendislik vakasının (bkz. *Vaka 1*) temelini oluşturacak öngörülü bir adımdır.

---

## ⚙️ Faz 2: Orkestrasyon Çekirdeği — Runtipi Entegrasyonu

Mikroservislerin manuel `docker-compose.yml` dosyalarıyla tek tek yönetilmesi yerine, merkezi bir **Orchestration Layer** olarak **Runtipi (v4.10.1)** seçilmiştir. Runtipi; konteyner yaşam döngüsü yönetimini (*lifecycle management*), ağ yapılandırmasını ve uygulama mağazası (*App Store*) konseptini tek bir kontrol düzlemi (*control plane*) altında birleştiren bir self-hosted PaaS (Platform as a Service) çözümüdür.

### 2.1. Neden Runtipi?

| Kriter | Geleneksel Docker Compose | Runtipi Orkestrasyonu |
|---|---|---|
| Servis Yönetimi | Manuel YAML editörlüğü | Merkezi CLI + Web Arayüzü |
| Ağ İzolasyonu | Elle tanımlanan `networks` | Otomatik internal bridge network |
| Güncelleme Yönetimi | Manuel `pull` + `up -d` | Tek komutla CLI tabanlı upgrade |
| Reverse Proxy Entegrasyonu | Harici yapılandırma gerektirir | Yerleşik (built-in) destek |

### 2.2. Çekirdek Servisin Devreye Alınması

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ sudo ./runtipi-cli start
```

Bu komut; Runtipi'nin kendi içsel konteynerlerini (reverse proxy, worker, dashboard) tetikleyerek, üst katmandaki tüm mikroservislerin yaşayacağı izole ağ ortamını (*internal Docker bridge network*) ayağa kaldırmıştır.

---

## 🔐 Faz 3: Ağ Mimarisi ve Güvenlik Katmanı

Bu faz, projenin **Defense in Depth** stratejisinin somutlaştığı en kritik aşamadır. Ağ güvenliği, tek bir araçla değil; katmanlı bir güvenlik yığınıyla (*security stack*) sağlanmıştır.

### 3.1. Çevre Güvenliği — UFW (Uncomplicated Firewall)

Sunucunun dış dünyaya açılan tüm portları, **prensip olarak varsayılan reddetme (default-deny)** yaklaşımıyla yapılandırılmıştır. Yalnızca açıkça izin verilen portlar (SSH, HTTP/HTTPS, Runtipi reverse proxy portları) trafiğe açık tutulmuş, kalan tüm portlar `DENY` politikasıyla kilitlenmiştir.

```bash
ornek_kullanici@ornek-sunucu:~$ sudo ufw status verbose
Status: active
Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       Anywhere
80,443/tcp (HTTP/HTTPS)    ALLOW       Anywhere
```

Bu yaklaşım, olası bir port tarama (*port scanning*) saldırısında saldırı yüzeyini (*attack surface*) minimuma indirmiştir.

### 3.2. Trafik Yönlendirme ve SSL/TLS Sonlandırma — Nginx Proxy Manager

Tüm mikroservislere tek bir IP adresi üzerinden, **alt alan adları (subdomain)** aracılığıyla erişim sağlanması için **Nginx Proxy Manager (NPM)** konuşlandırılmıştır. NPM; gelen istekleri `Host` başlığına (header) göre ilgili konteynere yönlendiren bir **Reverse Proxy** katmanı olarak çalışmış, aynı zamanda **Let's Encrypt** entegrasyonu ile otomatik SSL/TLS sertifika sağlama ve yenileme süreçlerini de üstlenmiştir.
İstemci (Client)

│

▼

[ UFW Firewall — Port Filtreleme ]

│

▼

[ Nginx Proxy Manager — SSL Termination + Subdomain Routing ]

│

├──▶ nextcloud.ornek-domain.local

├──▶ jellyfin.ornek-domain.local

├──▶ vault.ornek-domain.local

└──▶ dashboard.ornek-domain.loca
Bu mimari sayesinde, dış dünyaya tek bir giriş noktası (*single entry point*) sunulmuş; her servis kendi şifrelenmiş (HTTPS) ve insan tarafından okunabilir alt alan adı üzerinden hizmet vermeye başlamıştır.

---

## 🧩 Faz 4: Mikroservis Ekosistemi

Aşağıdaki tabloda, sisteme entegre edilen her mikroservisin mimari rolü ve stratejik amacı detaylandırılmıştır:

| # | Servis | Kategori | Mimari Rolü |
|---|---|---|---|
| 1 | ☁️ **Nextcloud** | Veri Egemenliği | Üçüncü parti bulut sağlayıcılarından bağımsız, self-hosted dosya senkronizasyonu ve depolama |
| 2 | 🎬 **Jellyfin** | Medya Sunumu | Donanım hızlandırmalı (*hardware-accelerated transcoding*) medya akışı |
| 3 | 🔑 **Vaultwarden** | Kriptografi & Güvenlik | End-to-end şifreli parola kasası (Bitwarden-uyumlu API) |
| 4 | 📊 **Glances** | Telemetri | Gerçek zamanlı CPU/RAM/Disk/Network kaynak izleme |
| 5 | 📜 **Dozzle** | Observability | Tüm konteynerlerin canlı log akışının web arayüzünden izlenmesi |
| 6 | 🗄️ **Adminer** | Veritabanı Yönetimi | İzole edilmiş, hafif veritabanı yönetim paneli |
| 7 | 🏠 **Homepage** | Merkezi Dashboard | Tüm servisleri API entegrasyonlarıyla birleştiren tek ekran kontrol merkezi |
| 8 | 💾 **Duplicati** | Disaster Recovery | Zamanlanmış, şifreli, artımlı (*incremental*) yedekleme orkestrasyonu |
| 9 | 🛡️ **Pi-hole** | Ağ Güvenliği & DNS | Ağ geneli DNS-seviyesinde reklam ve izleyici (*tracker*) engelleme |

### 4.1. Servisler Arası Entegrasyon Mantığı

Bu dokuz servis, birbirinden kopuk adacıklar olarak değil; **Homepage** dashboard'u üzerinden API anahtarlarıyla entegre edilmiş bütünleşik bir ekosistem olarak tasarlanmıştır. Örneğin Homepage; Glances'tan canlı sistem metriklerini, Dozzle'dan log durumlarını ve diğer servislerin "health check" (sağlık kontrolü) sonuçlarını tek bir görsel panelde toplayarak, **tek bakışta sistem durumu (Single Pane of Glass)** prensibini hayata geçirmiştir.

---

## 👁️ Faz 5: İzlenebilirlik ve Gözlemlenebilirlik (Observability)

Modern DevOps pratiğinin temel ilkelerinden biri olan *"Eğer ölçemiyorsan, yönetemezsin"* yaklaşımı, bu projede iki katmanlı bir gözlemlenebilirlik mimarisiyle uygulanmıştır:

- **Metrik Katmanı (Metrics Layer) — Glances:** Sistem seviyesinde CPU yükü, bellek tüketimi, disk I/O ve ağ trafiği gerçek zamanlı olarak telemetri verisine dönüştürülmüştür.
- **Log Katmanı (Logging Layer) — Dozzle:** Her konteynerin `stdout`/`stderr` çıktıları merkezi bir web arayüzünden, filtrelenebilir ve aranabilir şekilde canlı olarak takip edilebilir hale getirilmiştir.

Bu iki katmanın birleşimi, projenin en kritik troubleshooting vakalarının (bkz. *Vaka 2*) tespit edilmesini sağlayan **kök neden analizi altyapısını** oluşturmuştur.

---

## 🛟 Faz 6: Felaket Kurtarma ve Veri Dayanıklılığı (Disaster Recovery)

Self-hosted bir sistemin en büyük riski, donanım arızası veya insan hatası kaynaklı veri kaybıdır. Bu riski azaltmak amacıyla **Duplicati** servisi, aşağıdaki Disaster Recovery stratejisiyle entegre edilmiştir:

- ✅ **Zamanlanmış Yedekleme (Scheduled Backups):** Belirlenen aralıklarla otomatik tetiklenen yedekleme görevleri.
- ✅ **Uçtan Uca Şifreleme (Encryption at Rest):** Yedek veri setleri, AES tabanlı şifreleme ile diskte saklanmadan önce korumaya alınmıştır.
- ✅ **Artımlı Yedekleme (Incremental Backup):** Tüm veri setinin tekrar tekrar kopyalanması yerine, yalnızca değişen blokların yedeklenmesiyle depolama verimliliği sağlanmıştır.

Bu strateji, **RPO (Recovery Point Objective)** ve **RTO (Recovery Time Objective)** metriklerini minimize etmeyi hedefleyen kurumsal seviye bir yedekleme felsefesinin, ev sunucusu ölçeğinde uygulanmış halidir.

---

## 🔥 Kritik Mühendislik Vakaları (Troubleshooting Vakaları)

> *Bu bölüm, projenin kalbidir. Her sistem kurulur; ama her sistem mühendisi, sistem çöktüğünde ayakta kalır.*

### 🚨 Vaka #1: Runtipi v4.10.1 Güncellemesinde "Permission Denied" Krizi

**Belirti:**
Runtipi'nin `v4.10.1` sürümüne yükseltilmesi sürecinde, `docker-compose.yml` dosyası üzerinde işlem yapılırken sistem `Permission Denied` hatası fırlatmıştır.

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ ./runtipi-cli update
Error: EACCES: permission denied, open 'docker-compose.yml'
```

**Kök Neden Analizi (Root Cause Analysis):**
İnceleme sonucunda, önceki güncelleme/kurulum işlemlerinde bazı dosyaların `root` kullanıcısı tarafından oluşturulduğu ve dosya sahipliğinin (*file ownership*) standart kullanıcı hesabına devredilmediği tespit edilmiştir. Bu durum, Linux'un katı izin hiyerarşisi (UID/GID tabanlı erişim kontrolü) nedeniyle CLI aracının dosyaya yazma erişimini engellemiştir.

**Uygulanan Mühendislik Çözümü:**

1. Güncelleme işlemi, yükseltilmiş yetkilerle (*elevated privileges*) zorunlu kılınarak yeniden tetiklenmiştir:

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ sudo ./runtipi-cli update
```

2. Root tarafından kilitlenen dosya sahipliği, dosya sistemi hiyerarşisine müdahale edilerek standart kullanıcıya geri devredilmiştir:

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ sudo chown -R $USER:$USER .
```

3. Sistem, veri kaybı olmaksızın (*zero data loss*) başarıyla yeniden başlatılmış ve servis bütünlüğü doğrulanmıştır.

**Mühendislik Çıkarımı:**
Bu vaka, Linux dosya sistemi izin modelinin (`rwx` + UID/GID) konteynerize ortamlarda ne denli kritik olduğunu göstermiştir. Çözüm; sadece komutu çalıştırmaktan ibaret kalmamış, **dosya sahipliği hiyerarşisinin** sistemsel olarak anlaşılmasını ve kalıcı bir çözümle düzeltilmesini gerektirmiştir.

---

### 🚨 Vaka #2: Pi-hole Kurulumunda DNS Port 53 Çakışması

**Belirti:**
Pi-hole servisinin devreye alınması sürecinde, kurulum sihirbazı `"Log check required"` uyarısı vererek servisin sağlıklı başlamadığını bildirmiştir.

**Kök Neden Analizi (Root Cause Analysis):**
Doğrudan hata mesajı yeterli bilgi sunmadığından, **Dozzle** üzerinden Pi-hole konteynerinin canlı loglarına gerçek zamanlı erişim sağlanmıştır. Loglar incelendiğinde, asıl problemin şu olduğu net biçimde ortaya çıkmıştır:
ERROR: failed to bind host port for 0.0.0.0:53: address already in use
Bu noktada analiz, konteyner seviyesinden **işletim sistemi seviyesine** indirilmiştir. Yapılan araştırma; Ubuntu'nun yerleşik servisi olan **`systemd-resolved`**'in, sistemin varsayılan DNS çözümleyicisi (*resolver*) olarak `53` numaralı portu zaten dinlemekte (*listening*) olduğunu ortaya koymuştur. Pi-hole de aynı portu talep ettiğinden, klasik bir **port çakışması (port conflict)** senaryosu oluşmuştur.

**Uygulanan Mühendislik Çözümü:**

1. Port çakışmasının doğrulanması:

```bash
ornek_kullanici@ornek-sunucu:~$ sudo lsof -i :53
systemd-resolved   853   root   13u  IPv4  LISTEN
```

2. Çakışan sistem servisinin durdurulması ve devre dışı bırakılması:

```bash
ornek_kullanici@ornek-sunucu:~$ sudo systemctl stop systemd-resolved
ornek_kullanici@ornek-sunucu:~$ sudo systemctl disable systemd-resolved
```

3. `/etc/resolv.conf` yapılandırmasının, sistemin DNS çözümlemesini artık Pi-hole üzerinden yapacak şekilde yeniden düzenlenmesi.

4. Pi-hole konteynerinin `53/tcp` ve `53/udp` portlarını sorunsuz biçimde bind edebildiğinin doğrulanması ve ağ geneli DNS trafiğinin başarıyla yönlendirilmesi.

**Mühendislik Çıkarımı:**
Bu vaka; **Observability** araçlarının (Dozzle) yalnızca "izleme" amaçlı değil, **aktif kök neden analizinde bir teşhis aracı** olarak ne kadar kritik olduğunu kanıtlamıştır. Sorun, konteyner katmanında başlamış ama çözümü işletim sistemi katmanında (*host-level networking*) bulunmuştur — bu da katmanlar arası sistemik düşünme yeteneğinin altını çizmektedir.

---

## 🎓 Edinilen Mühendislik Yetkinlikleri

Bu 19 günlük yoğun süreç sonunda kazanılan teknik ve analitik yetkinlikler şu şekilde özetlenebilir:

- 🐳 **Container Orchestration:** Docker & Docker Compose ile çoklu servis yönetimi
- 🌐 **Network Architecture:** Reverse Proxy, Subdomain Routing, SSL/TLS Termination tasarımı
- 🔥 **Security Hardening:** UFW ile çevre güvenliği ve default-deny prensiplerinin uygulanması
- 👁️ **Observability Engineering:** Log ve metrik katmanlarının entegre kök neden analizi için kullanılması
- 🛟 **Disaster Recovery Planning:** Şifreli, zamanlanmış ve artımlı yedekleme stratejileri tasarımı
- 🧩 **Systemic Troubleshooting:** Konteyner katmanından işletim sistemi katmanına inen çok katmanlı problem çözme metodolojisi
- 📁 **Linux Filesystem Permission Management:** UID/GID tabanlı erişim kontrolü ve dosya sahipliği yönetimi

---

## 🚀 Sonuç ve Gelecek Vizyonu

Bu proje; bir stajyerin "izleyici" rolünden çıkıp, **kendi kararlarını alan, kendi hatalarını teşhis eden ve kendi çözümlerini üreten bir altyapı mühendisi** kimliğine geçişinin somut bir kanıtıdır. 19 günlük süre zarfında inşa edilen sistem; sadece "çalışan" bir altyapı değil, **anlaşılan, izlenebilen ve sürdürülebilen** bir mimari haline gelmiştir.

Gelecek fazlarda hedeflenen geliştirmeler:

- [ ] **Kubernetes (K3s)** geçişiyle gerçek anlamda *High Availability* ve otomatik ölçeklendirme
- [ ] **Prometheus + Grafana** entegrasyonu ile gelişmiş metrik görselleştirme ve alerting (uyarı) mekanizmaları
- [ ] **CI/CD Pipeline** (GitHub Actions) entegrasyonu ile altyapı değişikliklerinin otomatik test ve deploy süreçlerine bağlanması
- [ ] **Infrastructure as Code (IaC)** prensipleriyle Ansible/Terraform tabanlı otomatik provisioning

---

<div align="center">

### 📌 Bu proje, bir "staj defteri" değil; bir mühendislik yolculuğunun belgesidir.

**Hazırlayan: RadmelaSub**

</div>
English= https://github.com/radmela5461/Use-the-runtipi-as-a-server-on-your-own-computer-English/tree/main

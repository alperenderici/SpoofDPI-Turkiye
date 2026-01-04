# macOS'ta SpoofDPI Kurulum Rehberi
## Discord'a VPN Olmadan Bağlanma Kılavuzu

Bu rehber, Discord'a VPN kullanmadan bağlanabilmek için SpoofDPI'ı macOS'ta nasıl kuracağınızı ve arka planda otomatik olarak çalışacak şekilde nasıl yapılandıracağınızı adım adım anlatmaktadır.

> **Not:** SpoofDPI'ın ne olduğu, nasıl çalıştığı, faydaları ve VPN ile karşılaştırması için [HAKKINDA.md](HAKKINDA.md) dosyasına bakınız.

---

## İçindekiler

- [Sistem Gereksinimleri](#sistem-gereksinimleri)
- [Adım 1: SpoofDPI Kurulumu](#adım-1-spoofdpi-kurulumu)
- [Adım 2: PATH Yapılandırması](#adım-2-path-yapılandırması)
- [Adım 3: Arka Planda Otomatik Çalıştırma (LaunchAgent)](#adım-3-arka-planda-otomatik-çalıştırma-launchagent)
- [Servis Yönetimi](#servis-yönetimi)
- [Kurulumu Doğrulama](#kurulumu-doğrulama)
- [Sorun Giderme](#sorun-giderme)
- [Nasıl Çalışır?](#nasıl-çalışır)
- [Sık Sorulan Sorular](#sık-sorulan-sorular)

---

## Sistem Gereksinimleri

- **İşletim Sistemi**: macOS 10.15 (Catalina) veya üzeri
- **İşlemci**: Intel veya Apple Silicon (M1/M2/M3)
- **Terminal**: zsh (varsayılan) veya bash

---

## Adım 1: SpoofDPI Kurulumu

### Mac Tipinizi Belirleyin

Öncelikle Mac'inizin işlemci tipini öğrenmeniz gerekiyor:

1. Sol üst köşedeki Apple logosu () → **About This Mac** (Bu Mac Hakkında)
2. İşlemcinize bakın:
   - **Apple M1/M2/M3** yazıyorsa → Apple Silicon
   - **Intel Core** yazıyorsa → Intel

### Kurulum Komutları

Terminal'i açın (Applications → Utilities → Terminal veya Spotlight'ta "Terminal" arayın) ve Mac tipinize uygun komutu çalıştırın:

#### Apple Silicon (M1/M2/M3) için:
```bash
curl -fsSL https://raw.githubusercontent.com/renardev/SpoofDPI-Turkiye/main/install.sh | bash -s darwin-arm64
```

#### Intel Mac için:
```bash
curl -fsSL https://raw.githubusercontent.com/renardev/SpoofDPI-Turkiye/main/install.sh | bash -s darwin-amd64
```

Kurulum tamamlandığında SpoofDPI `~/.spoofdpi/bin/` dizinine kurulacaktır.

---

## Adım 2: PATH Yapılandırması

SpoofDPI'ı Terminal'den her yerden çalıştırabilmek için PATH değişkenine eklemeniz gerekir.

### Hangi Shell'i Kullandığınızı Öğrenin

Terminal'de şu komutu çalıştırın:
```bash
echo $SHELL
```

- `/bin/zsh` → zsh kullanıyorsunuz (macOS Catalina ve sonrası için varsayılan)
- `/bin/bash` → bash kullanıyorsunuz

### PATH'e Ekleme

#### zsh Kullanıcıları için:

1. Terminal'de şu komutu çalıştırın:
```bash
echo 'export PATH=$PATH:~/.spoofdpi/bin' >> ~/.zshrc
```

2. Değişiklikleri yükleyin:
```bash
source ~/.zshrc
```

#### bash Kullanıcıları için:

1. Terminal'de şu komutu çalıştırın:
```bash
echo 'export PATH=$PATH:~/.spoofdpi/bin' >> ~/.bashrc
```

2. Değişiklikleri yükleyin:
```bash
source ~/.bashrc
```

### Doğrulama

PATH'in doğru eklendiğini kontrol edin:
```bash
which spoofdpi
```

Çıktı şu şekilde olmalıdır:
```
/Users/KULLANICI_ADINIZ/.spoofdpi/bin/spoofdpi
```

---

## Adım 3: Arka Planda Otomatik Çalıştırma (LaunchAgent)

SpoofDPI'ın Mac'inizi her açtığınızda otomatik olarak başlaması ve arka planda çalışması için bir LaunchAgent oluşturacağız.

### 1. LaunchAgents Dizinini Oluşturun

```bash
mkdir -p ~/Library/LaunchAgents
```

### 2. plist Dosyasını Oluşturun

Terminal'de şu komutu çalıştırın (tüm satırlar bir bütündür, kopyalayıp yapıştırın):

```bash
cat > ~/Library/LaunchAgents/com.spoofdpi.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.spoofdpi</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/YOUR_USERNAME/.spoofdpi/bin/spoofdpi</string>
        <string>-pattern</string>
        <string>(discord.com|discordapp.com|discord.gg|discordcdn.com|discord.media)</string>
        <string>-silent</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <true/>

    <key>StandardOutPath</key>
    <string>/tmp/spoofdpi.log</string>

    <key>StandardErrorPath</key>
    <string>/tmp/spoofdpi.error.log</string>
</dict>
</plist>
EOF
```

### 3. Kullanıcı Adınızı Düzenleyin

**ÖNEMLİ**: Dosyadaki `YOUR_USERNAME` kısmını kendi kullanıcı adınızla değiştirmeniz gerekiyor.

Kullanıcı adınızı öğrenmek için:
```bash
whoami
```

Ardından dosyayı düzenleyin:
```bash
nano ~/Library/LaunchAgents/com.spoofdpi.plist
```

`YOUR_USERNAME` yazan yeri kendi kullanıcı adınızla değiştirin.

Örnek:
```xml
<string>/Users/ahmet/.spoofdpi/bin/spoofdpi</string>
```

Kaydetmek için:
- **CTRL + O** → Enter (dosyayı kaydet)
- **CTRL + X** (çıkış)

### 4. LaunchAgent'ı Yükleyin

```bash
launchctl load ~/Library/LaunchAgents/com.spoofdpi.plist
```

### 5. Servisin Çalıştığını Doğrulayın

```bash
launchctl list | grep com.spoofdpi
```

Çıktıda `com.spoofdpi` görüyorsanız servis başarıyla yüklenmiştir.

---

## Servis Yönetimi

### Servisi Durdurmak

```bash
launchctl unload ~/Library/LaunchAgents/com.spoofdpi.plist
```

### Servisi Yeniden Başlatmak

```bash
launchctl unload ~/Library/LaunchAgents/com.spoofdpi.plist
launchctl load ~/Library/LaunchAgents/com.spoofdpi.plist
```

### Servisi Tamamen Kaldırmak

```bash
launchctl unload ~/Library/LaunchAgents/com.spoofdpi.plist
rm ~/Library/LaunchAgents/com.spoofdpi.plist
```

### Logları Kontrol Etmek

#### Normal Çıktı Logu:
```bash
tail -f /tmp/spoofdpi.log
```

#### Hata Logu:
```bash
tail -f /tmp/spoofdpi.error.log
```

Logları temizlemek için:
```bash
> /tmp/spoofdpi.log
> /tmp/spoofdpi.error.log
```

---

## Kurulumu Doğrulama

Kurulumun başarılı olduğundan emin olmak için şu kontrolleri yapın:

### 1. Binary Kontrolü
```bash
which spoofdpi
```
Çıktı: `/Users/KULLANICI_ADINIZ/.spoofdpi/bin/spoofdpi`

### 2. PATH Kontrolü
```bash
echo $PATH | grep spoofdpi
```
Çıktıda `.spoofdpi/bin` görmelisiniz.

### 3. LaunchAgent Kontrolü
```bash
launchctl list | grep com.spoofdpi
```
Çıktıda `com.spoofdpi` görmelisiniz.

### 4. Sistem Proxy Kontrolü

System Preferences (Sistem Tercihleri) → Network (Ağ) → Advanced (Gelişmiş) → Proxies

Şurada bir proxy yapılandırması görmelisiniz:
- **Web Proxy (HTTP)**: 127.0.0.1:8080
- **Secure Web Proxy (HTTPS)**: 127.0.0.1:8080

### 5. Discord Testi

Artık Discord'u açabilir ve bağlanabildiğinizi test edebilirsiniz!

---

## Sorun Giderme

### Discord'a Hala Bağlanamıyorum

1. **Servisi yeniden başlatın**:
   ```bash
   launchctl unload ~/Library/LaunchAgents/com.spoofdpi.plist
   launchctl load ~/Library/LaunchAgents/com.spoofdpi.plist
   ```

2. **Hata loglarını kontrol edin**:
   ```bash
   cat /tmp/spoofdpi.error.log
   ```

3. **Sistem proxy ayarlarını kontrol edin**:
   System Preferences → Network → Advanced → Proxies

### Servis Başlamıyor

1. **plist dosyasını kontrol edin**:
   ```bash
   cat ~/Library/LaunchAgents/com.spoofdpi.plist
   ```

   Kullanıcı adının doğru olduğundan emin olun.

2. **Binary'nin var olduğunu kontrol edin**:
   ```bash
   ls -la ~/.spoofdpi/bin/spoofdpi
   ```

3. **İzinleri kontrol edin**:
   ```bash
   chmod +x ~/.spoofdpi/bin/spoofdpi
   ```

### Terminal'i Yeniden Başlattığımda PATH Kayboldu

`~/.zshrc` veya `~/.bashrc` dosyanıza PATH'i eklemediğiniz anlamına gelir. [Adım 2](#adım-2-path-yapılandırması)'yi tekrarlayın.

### Proxy Ayarları Otomatik Ayarlanmıyor

macOS'ta SpoofDPI varsayılan olarak sistem proxy'sini otomatik ayarlar. Eğer ayarlanmadıysa:

1. **Manual olarak proxy ayarlayın**:
   - System Preferences → Network → Advanced → Proxies
   - Web Proxy (HTTP): `127.0.0.1` Port: `8080`
   - Secure Web Proxy (HTTPS): `127.0.0.1` Port: `8080`

2. **Tarayıcıda proxy kullanın**:
   ```bash
   open -a "Google Chrome" --args --proxy-server="http://127.0.0.1:8080"
   ```

---

## Nasıl Çalışır?

### SpoofDPI Nedir?

SpoofDPI (Deep Packet Inspection Spoofing), internet servis sağlayıcılarının (ISP) derin paket incelemesini (DPI) atlatmak için kullanılan bir araçtır.

### Discord Neden Engelleniyor?

Türkiye'de Discord, ISP'ler tarafından DPI kullanılarak engellenebilmektedir. ISP'ler, TLS handshake sırasında düz metin olarak gönderilen alan adlarını (discord.com, discordapp.com vb.) tespit edip bağlantıyı kesiyor.

### SpoofDPI Nasıl Atlatıyor?

1. **Paket Parçalama**: SpoofDPI, TLS Client Hello paketini (alan adı bilgisini içeren) küçük parçalara böler
2. **İlk Byte Gönderimi**: İlk 1 byte'ı sunucuya gönderir
3. **Kalan Kısmı Gönderimi**: Ardından kalan kısmı gönderir
4. **DPI Atlatma**: DPI sistemleri genellikle sadece ilk parçayı inceler ve alan adını tespit edemez

### LaunchAgent Nedir?

LaunchAgent, macOS'un sistemde kullanıcı giriş yaptığında otomatik olarak başlatılan servisleri yönetmek için kullandığı bir mekanizmadır. Bu sayede:
- Mac'inizi her açtığınızda SpoofDPI otomatik başlar
- Kapanırsa otomatik yeniden başlar (KeepAlive)
- Arka planda sessizce çalışır (silent mode)
- Sistem kaynaklarını verimli kullanır

---

## Sık Sorulan Sorular

### Güvenli mi?

Evet. SpoofDPI:
- Yerel bir proxy olarak çalışır (127.0.0.1)
- Verilerinizi şifrelemez veya değiştirmez
- Sadece paketleri parçalara böler
- Açık kaynak kodludur (GitHub'da inceleyebilirsiniz)

### Sadece Discord için mi Çalışır?

Hayır. Varsayılan olarak Discord için yapılandırılmıştır ama başka siteler için de kullanabilirsiniz. plist dosyasındaki `-pattern` parametresini değiştirerek farklı domainler ekleyebilirsiniz.

Örnek:
```xml
<string>-pattern</string>
<string>(discord.com|twitter.com|reddit.com)</string>
```

### Performansı Etkiler mi?

Minimal etki. SpoofDPI sadece TLS handshake sırasında devreye girer. Normal veri transferi sırasında performans kaybı yaşamazsınız.

### VPN ile Aynı Şey mi?

Hayır. SpoofDPI:
- IP adresinizi değiştirmez
- Şifreleme eklemez
- Sadece DPI engellemesini atlatır
- VPN'den çok daha hızlıdır

### Mac'i Yeniden Başlattığımda Çalışmaya Devam Eder mi?

Evet. LaunchAgent sayesinde Mac'inizi her açtığınızda otomatik olarak başlar.

### Başka Uygulamalar Etkilenir mi?

Hayır. `-pattern` parametresi sadece Discord domainlerini hedefler. Diğer tüm internet trafiğiniz normal şekilde devam eder.

### Kurulumu Kaldırmak İstersem?

```bash
# LaunchAgent'ı kaldır
launchctl unload ~/Library/LaunchAgents/com.spoofdpi.plist
rm ~/Library/LaunchAgents/com.spoofdpi.plist

# Binary'yi kaldır
rm -rf ~/.spoofdpi

# PATH'ten kaldır (nano ile düzenle)
nano ~/.zshrc
# "export PATH=$PATH:~/.spoofdpi/bin" satırını silin
```

---

## Ek Bilgiler

### Gelişmiş Kullanım

Manuel olarak özel parametrelerle çalıştırmak isterseniz:

```bash
spoofdpi -addr 127.0.0.1 -port 8080 -dns-addr 77.88.8.8 -pattern "(discord.com|discordapp.com)"
```

Tüm parametreler için:
```bash
spoofdpi -h
```

### Hata Ayıklama Modu

Debug mode ile çalıştırmak için plist dosyasına şunu ekleyin:

```xml
<string>-debug</string>
```

Ardından servisi yeniden yükleyin.

### Farklı DNS Kullanma

Varsayılan DNS (77.88.8.8 - Yandex DNS) yerine farklı bir DNS kullanmak için:

```xml
<string>-dns-addr</string>
<string>8.8.8.8</string>
```

### DNS over HTTPS (DoH)

DNS sorgularını şifrelemek için DoH'u etkinleştirin:

```xml
<string>-enable-doh</string>
```

---

## Destek ve Katkı

- **GitHub**: [SpoofDPI-Turkiye](https://github.com/renardev/SpoofDPI-Turkiye)
- **Sorun Bildirimi**: GitHub Issues
- **Katkıda Bulunma**: Pull Request gönderin

---

## Lisans

Bu proje açık kaynak kodludur. Detaylar için LICENSE dosyasına bakın.

---

**Not**: Bu rehber macOS kullanıcıları için hazırlanmıştır. Linux ve Windows için farklı kurulum adımları gerekebilir.

**Son Güncelleme**: Ocak 2026

# SpoofDPI Hakkında

## Bu Tam Olarak Nedir?

SpoofDPI, **Deep Packet Inspection (DPI)** engellemesini atlatan bir **yerel proxy sunucusu**dur. VPN değildir, TLS handshake'i manipüle eden bir paket parçalama aracıdır.

**Nasıl Çalışır:**
1. Client Hello paketini (alan adı bilgisini içeren) yakalar
2. Paketi küçük parçalara böler (ilk 1 byte, sonra geri kalan)
3. DPI sistemleri sadece ilk parçayı inceler ve alan adını tespit edemez
4. Bağlantı başarıyla kurulur

---

## Faydaları ve Eksileri

### ✅ Faydaları (Artılar)

#### Performans
- **Çok hızlı** - VPN'den 10-100x daha hızlı
- Sadece handshake sırasında devreye girer
- Normal veri transferinde gecikme yok
- CPU/RAM kullanımı minimal

#### Gizlilik
- **IP adresiniz değişmez** ama bu aslında bir avantaj olabilir
- Verileriniz üçüncü taraf sunuculara gitmez
- Loglamaz (kendiniz kontrol edebilirsiniz)
- Açık kaynak - kod incelenebilir

#### Kullanım
- Kurulum çok basit
- Arka planda sessizce çalışır
- Sadece engellenen siteleri hedefler (pattern ile)
- Sistem genelinde veya tarayıcı bazlı çalışabilir
- Ücretsiz

#### Teknik
- Şifrelemeyi bozmaz (TLS dokunulmaz)
- DNS over HTTPS (DoH) desteği
- Özelleştirilebilir (regex pattern, DNS, port)
- macOS/Linux/Windows desteği

### ❌ Eksileri (Dezavantajları)

#### Sınırlı Kapsam
- **Sadece DPI engellemesini atlatır**
- IP bazlı engelleri atlatamaz
- Coğrafi kısıtlamaları aşamaz (geo-blocking)
- ISP'niz hangi sitelere gittiğinizi görebilir

#### Gizlilik
- IP adresinizi gizlemez
- Trafik analizi yapılabilir (metadata)
- ISP DNS sorgularınızı görebilir (DoH kullanmazsanız)
- Anonimlik sağlamaz

#### Teknik Zorluklar
- Her engelleme yöntemine karşı etkili değil
- ISP yöntemini değiştirirse çalışmayabilir
- Bazı ağlarda proxy trafiği engellenebilir
- Kurulum teknik bilgi gerektirir (özellikle LaunchAgent)

#### Kullanım
- Tüm uygulamalar proxy'yi desteklemez
- macOS'ta sistem proxy bazen sorunlu olabilir
- LaunchAgent plist dosyasında manuel düzenleme gerekir
- Hata ayıklama VPN'den daha zor

---

## VPN vs SpoofDPI Karşılaştırması

| Özellik | SpoofDPI | VPN |
|---------|----------|-----|
| **Hız** | ⚡ Çok hızlı (native bağlantı) | 🐌 Yavaş (şifreleme overhead) |
| **DPI Atlama** | ✅ Evet | ✅ Evet |
| **IP Gizleme** | ❌ Hayır | ✅ Evet |
| **Geo-blocking** | ❌ Atlatamaz | ✅ Atlatır |
| **Anonimlik** | ❌ Yok | ✅ Var (güvenilen VPN'de) |
| **Ücretsiz** | ✅ Tamamen | ⚠️ Sınırlı/Ücretli |
| **Kurulum** | ⚠️ Teknik | ✅ Kolay |
| **CPU Kullanımı** | ✅ Minimal | ❌ Yüksek |
| **Güvenilirlik** | ⚠️ ISP'ye bağlı | ✅ Yüksek |
| **Tüm Trafik** | ✅/❌ Seçilebilir | ✅ Hepsi |

### Hangisi Tercih Edilmeli?

#### SpoofDPI Kullanın:
- ✅ **Sadece Discord/Twitter gibi belirli siteler engelleniyorsa**
- ✅ Hızlı bağlantı istiyorsanız (oyun, video)
- ✅ ISP'niz DPI kullanarak engelliyor
- ✅ IP gizlemeye ihtiyacınız yoksa
- ✅ Ücretsiz çözüm istiyorsanız
- ✅ Teknik bilginiz varsa

**Örnek Kullanım:** Discord'a bağlanmak, Twitter'ı açmak, belirli sitelere erişmek

#### VPN Kullanın:
- ✅ **Tam anonimlik istiyorsanız**
- ✅ IP adresinizi gizlemek gerekiyorsa
- ✅ Coğrafi kısıtlamaları aşmanız gerekiyorsa (Netflix başka ülke)
- ✅ ISP'niz IP bazlı engelleme yapıyorsa
- ✅ Hassas bilgilerle çalışıyorsanız
- ✅ Kolay kurulum istiyorsanız

**Örnek Kullanım:** Torrent, hassas araştırma, başka ülkenin içeriğine erişim

#### İkisini Birlikte Kullanın:
- Split tunneling: VPN üzerinden bazı uygulamalar, SpoofDPI üzerinden Discord
- Ancak genellikle gereksiz - biri yeterli

### Türkiye İçin Öneriler

**Şu an Discord için en iyi çözüm: SpoofDPI**
- Discord'un Türkiye'deki engeli DPI tabanlı
- SpoofDPI hızlı ve etkili
- Oynarken ping/gecikme problemi olmaz
- Ücretsiz

**VPN gerekli değil AMA:**
- Eğer ISP yöntemi değişirse (IP ban) VPN'e geçebilirsiniz
- Hem Discord hem başka şeyler için gizlilik istiyorsanız VPN
- Para ödemeye razıysanız VPN daha kolay

**Hibrit Yaklaşım:**
```bash
# Sadece Discord için SpoofDPI
spoofdpi -pattern "(discord.com|discordapp.com)"

# Diğer her şey normal bağlantı
# Gerektiğinde VPN açın
```

**Özet:** SpoofDPI hızlı, ücretsiz ve hafif ama sadece DPI için. VPN yavaş, çoğu ücretli ve ağır ama tam koruma sağlar. Türkiye'de Discord için **SpoofDPI yeterli ve tercih edilmeli**.

---

## Teknik Detaylar

### HTTP
Dünyadaki çoğu web sitesi artık HTTPS'yi desteklediğinden, SpoofDPI HTTP istekleri için Derin Paket Denetimlerini atlamaz, ancak yine de tüm HTTP istekleri için proxy bağlantısı sunar.

### HTTPS
TLS her handshake işlemini şifrelese de, İstemci dönüş paketinde alan adları hala düz metin olarak gösterilir. Başka bir deyişle, başka biri pakete baktığında, paketin nereye gittiğini kolayca tahmin edebilir. DPI işlenirken alan adı önemli bilgiler sunabilir ve aslında İstemci dönüş paketini gönderdikten hemen sonra bağlantının engellendiğini görebiliriz.

Bunu aşmak için bazı yollar denedik ve İstemci dönüş paketini parçalara bölerek gönderdiğimizde yalnızca ilk parçanın denetlendiğini fark ettik. SpoofDPI'ın bunu atlamak için yaptığı şey, bir isteğin ilk 1 baytını sunucuya göndermek ve sonra geri kalanını göndermektir.

---

## Daha Fazla Bilgi

- [README.md](README.md) - Ana sayfa ve hızlı başlangıç
- [MACOS_KURULUM.md](MACOS_KURULUM.md) - macOS için detaylı kurulum kılavuzu
- [_docs/INSTALL.md](_docs/INSTALL.md) - Genel kurulum dokümanı

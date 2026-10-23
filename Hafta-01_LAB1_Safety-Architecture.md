# Hafta 1 — LAB 1 · Safety Architecture → Çıktı: Safety Function Map

| Ad Soyad | Tarih |
|---|---|
| Mervan Oktan | 23.10.2026 |

> **Makine:** FADA Raysız Akülü Transfer Arabası (FTC) — https://www.fada.com.tr/urun/raysiz-akulu-transfer-arabasi-ftc/  
> **Kaynak föy:** INSPECCO × KRIPTARIUM — Safety 4.0 Bootcamp, Laboratuvar Föyleri (Genel Sürüm), Laboratuvar 1 (föydeki örnek makine şişeleme hattı; sorular FTC'ye uyarlanarak cevaplandı).  
> ❓ = sitede yazmayan, bizim yaptığımız varsayım. Bu bir eğitim çalışmasıdır; resmî risk değerlendirmesi veya CE uygunluğu yerine geçmez.

## LAB 1 · Safety Architecture → Çıktı: Safety Function Map

**Kavram (kendi cümlemle):** Makinenin "işini yapan" yolu (normal kontrol) ile "insanı koruyan" yolu (safety) ayrı olmalı. Normal PLC çökse, hacklense bile safety yolu aracı durdurabilmeli.

**1. Bu makinede tehlike nedir? İnsan neyden zarar görür?**
- **Ezilme / çarpma:** Ana tehlike. 400 tona kadar hareket eden kütle, yolundaki insanı ezer. Hız düşük (≈0,5 m/s) ama kütle çok büyük olduğu için durma mesafesi uzar.
- **Sıkışma:** Araç ile duvar, kolon, raf veya başka makine arasında kalma. Raysız olduğu için araç "her yere" gidebiliyor, yani sıkışma noktaları sabit değil.
- **Yük düşmesi / devrilmesi:** Ani fren, dönüş veya eğimde sabitlenmemiş yük kayar.
- **Elektrik:** Yüksek DC akü voltajı (bakım sırasında).
- **Yangın:** Akü ısıl kaçağı (lityum ise) / hidrojen (kurşun-asit ise) ❓, dizel versiyonda yakıt.
- **Kim zarar görür:** Araç yolunda yürüyen çalışanlar (en çok risk altında), yükleme-boşaltma yapan kişi, operatör, bakımcı.

**2. Safety function (emniyet fonksiyonu) hangisidir?**
Bu makinede tek değil, birkaç tane var:

| Kod | Emniyet fonksiyonu | Ne yapar |
|---|---|---|
| **SF1** | Acil durdurma (E-stop) | Araç üstündeki veya kumandadaki butona basılınca araç durur |
| **SF2** | Kişi/engel algılayınca yavaşla → dur | Lazer tarayıcının *uyarı alanında* biri varsa yavaşlar, *koruma alanına* girerse durur |
| **SF3** | Tampon teması → dur | Tampon bir şeye değerse anında durur (son savunma hattı) |
| **SF4** | Güvenli hız sınırlama | Araç belirlenen hızı (ör. 30 m/dk) hiçbir şekilde aşamaz |
| **SF5** | Kumanda bağlantısı koparsa dur | RF kumandadan belirli sürede sinyal gelmezse araç durur (watchdog) |
| **SF6** | Güvenli fren tutma | Enerji kesilince yaylı fren kapanır, araç kendi kendine kaymaz |

En kritik olanlar **SF2 (kişi algılama)** ve **SF1 (E-stop)**. Haritanın merkezine bunları koyuyoruz.

**3. Safety-related control system nerede? (şema)**
```
NORMAL KONTROL YOLU (üretim / hareket)
 Kumanda (hareket komutları) / Otonom navigasyon ─► Araç PLC ─► Motor sürücüleri ─► Tekerlek motorları
 Filo yöneticisi (Wi-Fi) ❓ ──┘          ▲
 Servis laptopu / uzak erişim ───────────┘  (bakım, güncelleme)

SAFETY YOLU (koruma) — AYRI, kablolu   ★ = safety-related control system
 ★ E-stop butonları (araç + kumanda safety kanalı)  ─┐
 ★ Lazer tarayıcılar (ön/arka, OSSD çıkış)          ├─► ★ Safety PLC / safety röle ─► ★ STO (sürücülerde) + ★ ana kontaktör + ★ yaylı fren ─► GÜVENLİ DURUM
 ★ Tampon (bumper) kontakları                        │
 ★ Hız encoder'ı (güvenli hız izleme)               ─┘

⚡ DİJİTAL KESİŞİM NOKTALARI (iki yolun birbirine dokunduğu yerler):
  - Araç PLC'si → lazer tarayıcıya "hangi koruma alanını kullan" bilgisini gönderir (hıza/yöne göre alan değiştirme)
  - Servis laptopu → lazer tarayıcı ve safety PLC'nin konfigürasyon yazılımı
  - RF kumanda → hem hareket komutu hem E-stop aynı telsiz bağlantısından geçer
```
Safety-related control system = ★ işaretli parçaların hepsi: girişteki sensörler + mantık (safety PLC) + çıkıştaki STO/kontaktör/fren.

**4. Normal PLC ile Safety PLC farkı nedir?**
| Normal PLC | Safety PLC |
|---|---|
| Aracı sürer, rota/hız/yön hesaplar | Sadece "durmalı mı?" kararını verir |
| Tek işlemci, hata olursa fark etmeyebilir | Çift kanallı (yedekli), kendi kendini test eder, hata görünce güvenli duruma geçer |
| Program sık değişir, esnek | Program az değişir, değişince checksum değişir ve **yeniden doğrulama** gerekir |
| Sertifika gerekmez | ISO 13849 / IEC 61508 sertifikalı (PL/SIL derecesi var) |
| Arızası = iş durur | Arızası = insan zarar görebilir |

**5. PLr / SIL neden var? Bu fonksiyon için neden önemli?**
- PLr (gerekli performans seviyesi) "bu emniyet fonksiyonu ne kadar güvenilir olmalı?" sorusunun cevabı. Risk ne kadar büyükse, fonksiyon o kadar az arızalanmalı.
- FTC için risk grafiği (ISO 13849-1) ile kaba tahmin:
  - **S2** (ciddi yaralanma / ölüm): 400 ton ezilme → evet
  - **F2** (sık maruz kalma): fabrikada yaya trafiği sürekli → evet
  - **P2** (kaçınmak zor): kör noktalar, sıkışma, operatör uzakta → evet
  - → **PLr = e** (en yüksek seviye) çıkabilir. Pratikte sürücüsüz endüstriyel araçlar standardı **ISO 3691-4** kişi algılama ve E-stop için genelde en az **PLd** ister.
- Önemi: Sıradan bir sensör + normal PLC ile bu güvenilirlik seviyesine ulaşılamaz. Bu yüzden safety yolu ayrı ve sertifikalı olmak zorunda. ❓ FADA sitesinde PL seviyesi yazmıyor.

**6. Safe state (güvenli durum) nedir?**
- Motor torku kesik (**STO**), yaylı frenler kapalı, araç **hareketsiz**, yük tablası (kaldırma varsa) kilitli, uyarı ışığı/sesi açık, otomatik yeniden kalkış **yok** (biri bilinçli olarak reset etmeli).
- ⚠️ FTC'ye özel incelik: 400 tonluk yükle **ani** fren yapmak yükü kaydırabilir/devirebilir. Bu yüzden ideal olan **kontrollü duruş (SS1)**: önce rampa ile hızlıca yavaşla, sonra STO + fren. Yani "en hızlı durmak" her zaman "en güvenli durmak" değil. Tampon teması gibi son-an durumlarında ise doğrudan durma (STO + fren) gerekir.

**7. Safety function hangi dijital bağımlılıklara sahip?**
1. **Lazer tarayıcının alan konfigürasyonu:** Koruma alanının boyutu yazılımla ayarlanıyor. Alan küçültülürse araç insanı geç görür.
2. **Alan değiştirme girdisi:** Hangi alanın aktif olacağını hız/yön bilgisine göre normal PLC söylüyor olabilir ❓ → normal yol safety yoluna dokunuyor.
3. **Safety PLC programı ve parametreleri** (checksum/CRC ile korunmalı).
4. **Encoder / hız ölçümü:** Güvenli hız izleme buna dayanıyor.
5. **RF kumanda eşleştirmesi ve E-stop kanalı:** Telsiz bağlantı protokolü, eşleştirme, watchdog süresi.
6. **Motor sürücüsü STO parametreleri ve firmware.**
7. **Otonom mod yazılımı:** Rota ve hız kararları; yazılım hata yaparsa araç yanlış yere gidebilir ❓.

**8. Siber müdahale hangi noktadan bu fonksiyonu bozar?**
- 🎯 **Servis laptopu / mühendislik yazılımı:** Lazer tarayıcının koruma alanını sessizce küçültmek veya safety PLC'de bir girişi köprülemek (bypass). En tehlikelisi bu, çünkü dışarıdan bakınca makine "normal" çalışır.
- 🎯 **Normal PLC üzerinden alan değiştirme girdisini sahtelemek:** Araç hızlıyken "yavaş gidiyorum, küçük alanı kullan" demek.
- 🎯 **RF kumanda:** Sinyal kaydedip tekrar gönderme (replay) ile istenmeyen hareket; karıştırma (jamming) ile E-stop komutunu engelleme (bu yüzden SF5 "sinyal yoksa dur" şart).
- 🎯 **Wi-Fi / filo yöneticisi ❓:** Hız limiti veya rota parametrelerini değiştirmek.
- 🎯 **Firmware güncellemesi:** Değiştirilmiş (zararlı) firmware yüklenmesi.
- Fiziksel: Lazer tarayıcıyı kapatmak/kirletmek, tamponu bantlamak.

> **Evidence Pack:** Bu cevaplar EP 03 Safety Functions başlığına gider.

---

## Önce kısaca: şişeleme hattıyla FTC arasındaki karşılıklar
| Föydeki (şişeleme hattı) | Bizim makinedeki (FTC) |
|---|---|
| Konveyör / robot | Hareket eden 5–400 tonluk araç + yük tablası |
| Işık perdesi | **Lazer tarayıcı** (koruma alanı) + ultrasonik sensör + **tampon** |
| E-stop | Araç üstündeki mantar butonlar + **kumandadaki E-stop** |
| PLC | Araç kontrol PLC'si / navigasyon bilgisayarı |
| Robot kontrolörü | Motor sürücüleri (PMAC/AC inverter) + otonom navigasyon kontrolcüsü |
| SCADA istemcisi | Filo/trafik yönetim yazılımı (otonom modda) ❓ |
| Uzak servis erişimi | Üreticinin (FADA) uzaktan destek bağlantısı ❓ |
| Kamera → YZ | Otonom moddaki engel/insan algılama yazılımı ❓ |

---

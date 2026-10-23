# Hafta 4 — LAB 4 · Secure Remote Access → Çıktı: Secure Architecture

| Ad Soyad | Tarih |
|---|---|
| Mervan Oktan | 23.10.2026 |

> **Makine:** FADA Raysız Akülü Transfer Arabası (FTC) — https://www.fada.com.tr/urun/raysiz-akulu-transfer-arabasi-ftc/  
> **Kaynak föy:** INSPECCO × KRIPTARIUM — Safety 4.0 Bootcamp, Laboratuvar Föyleri (Genel Sürüm), Laboratuvar 4 (föydeki örnek makine şişeleme hattı; sorular FTC'ye uyarlanarak cevaplandı).  
> ❓ = sitede yazmayan, bizim yaptığımız varsayım. Bu bir eğitim çalışmasıdır; resmî risk değerlendirmesi veya CE uygunluğu yerine geçmez.

## LAB 4 · Secure Remote Access → Çıktı: Secure Architecture

**Kavram:** İki katman, ikisi de şart:
1. **Erişim güvenliği:** İçeri girmeyi zorlaştır.
2. **Emniyet bağımsızlığı:** Saldırgan içeri girse bile insan korunsun.

### Görev A — mevcut → hedef
❓ "Mevcut" durum FADA'nın gerçek sistemi değil; bu tip makinelerde sahada **sık görülen kötü durumu** varsaydık.

| | MEVCUT (tipik / varsayım) | HEDEF (beş ilke ile) |
|---|---|---|
| Bağlantı yolu | Araç üzerinde 4G modem veya TeamViewer, servis doğrudan araç PLC'sine bağlanıyor | **Ara nokta:** Servis yalnızca fabrikanın DMZ'sindeki jump server'a bağlanır, oradan sadece izinli araca gider |
| Tünel | İnternete açık port / şifresiz bağlantı | **VPN:** Şifreli tünel, internete açık port yok |
| Kimlik | Tek parola (herkes biliyor) | **MFA + kişisel hesap:** Her teknisyenin kendi hesabı, ikinci faktör |
| Kayıt | Kim bağlandı, ne yaptı bilinmiyor | **Kayıt:** Tam oturum kaydı + PLC/safety konfigürasyon değişiklik logu |
| Zamanlama | 7/24 açık | **Varsayılan kapalı:** Fabrikadan bir yetkili onaylayınca, belirli süre (ör. 2 saat) açık, sonra otomatik kapanır |
| Safety erişimi | Uzaktan safety konfigürasyonu da değiştirilebiliyor | Safety (Z4) **uzaktan hiç değiştirilemez**; sadece yerinde, fiziksel anahtarla |

```
MEVCUT:  [FADA servis] ──internet── [4G modem] ── [Araç PLC + Safety PLC + Lazer config]   (tek parola, log yok, hep açık)

HEDEF:   [FADA servis] ─VPN+MFA─► [Jump server (DMZ, kayıt)] ─onaylı/süreli─► [Araç PLC (Z3)]
                                                                         ✖ Safety (Z4)'e uzaktan yol YOK
```

### Görev B — functional safety köprüsü

**1. Saldırgan tüm önlemlere rağmen içeri girerse emniyet fonksiyonu etkilenir mi? Neden?**
- **Hedef mimaride: etkilenmemeli.** Çünkü safety yolu (E-stop, lazer tarayıcı, tampon → safety PLC → STO/fren) ayrı ve kablolu; Z3'ten Z4'e yazma yolu yok. Saldırgan aracı hareket ettirebilse bile, önüne insan çıkınca lazer tarayıcı ve safety PLC aracı durdurur.
- **Ama dikkat:** Etkilenme ihtimali şu kesişim noktalarında kalıyor: (a) alan değiştirme girdisi normal PLC'den geliyorsa, (b) kumandada hareket ve E-stop aynı telsiz bağlantısını kullanıyorsa, (c) servis laptopu hem normal hem safety konfigürasyonuna erişebiliyorsa. Bu yüzden gereksinimleri aşağıdaki gibi yazıyoruz.

**2. Emniyet fonksiyonunun kontrol sisteminden bağımsız çalışması için GEREKSİNİM** (çözüm değil):
- **G1:** Emniyet fonksiyonları (SF1–SF6), normal kontrol sistemi, fabrika ağı ve uzak erişim kanalı tamamen ele geçirilmiş olsa bile, gerekli PL seviyesinde çalışmaya devam etmelidir.
- **G2:** Safety konfigürasyonu ve parametreleri uzaktan değiştirilememelidir; her değişiklik makine başında fiziksel varlık, kişisel yetki ve kayıt gerektirmelidir.
- **G3:** Safety sistemi, normal kontrolden gelen her bilgiye (ör. hız, alan seçimi) kendi bağımsız ölçümüyle makullük kontrolü yapmalı; tutarsızlıkta güvenli duruma geçmelidir.
- **G4:** Kumanda veya ağ bağlantısı kaybı ya da beklenmeyen komut, aracın kontrollü şekilde güvenli duruma geçmesine yol açmalıdır.
- **G5:** Safety yazılımı/konfigürasyonundaki her değişiklik tespit edilebilir olmalı (checksum/CRC) ve değişiklik sonrası araç, yeniden doğrulama yapılmadan çalışmamalıdır.

> **Tarafsızlık notu:** Bu föy çözüm değil gereksinim istiyor; nasıl yapılacağı (hangi ürün, hangi mimari) FADA'nın tasarım kararı.
> **Evidence Pack:** EP 02 + EP 06.

---

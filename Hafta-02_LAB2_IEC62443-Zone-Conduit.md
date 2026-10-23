# Hafta 2 — LAB 2 · IEC 62443 Mini Workshop → Çıktı: Zone / Conduit Diagram

| Ad Soyad | Tarih |
|---|---|
| Mervan Oktan | 23.10.2026 |

> **Makine:** FADA Raysız Akülü Transfer Arabası (FTC) — https://www.fada.com.tr/urun/raysiz-akulu-transfer-arabasi-ftc/  
> **Kaynak föy:** INSPECCO × KRIPTARIUM — Safety 4.0 Bootcamp, Laboratuvar Föyleri (Genel Sürüm), Laboratuvar 2 (föydeki örnek makine şişeleme hattı; sorular FTC'ye uyarlanarak cevaplandı).  
> ❓ = sitede yazmayan, bizim yaptığımız varsayım. Bu bir eğitim çalışmasıdır; resmî risk değerlendirmesi veya CE uygunluğu yerine geçmez.

## LAB 2 · IEC 62443 Mini Workshop → Çıktı: Zone / Conduit Diagram

**Kavram (kendi cümlemle):** Benzer önemdeki cihazları aynı "bölgeye" (zone) koy, bölgeler arasındaki her geçişe (conduit) kontrollü bir kapı koy. Saldırgan bir bölgeye girse bile diğerine kolayca geçemesin.

### Görev A — varlıkları bölgelere ayırma

**1. Bölgeler ve içindeki varlıklar**
| Bölge | İçindeki varlıklar (föydeki listenin FTC karşılığı) | Kritiklik / Hedef SL |
|---|---|---|
| **Z0 – Kurumsal** | Ofis ağı (ERP, e-posta), MES/WMS | Düşük – SL 1 |
| **Z1 – DMZ / Uzak erişim** | Uzak servis erişimi (jump server, VPN geçidi), yama/güncelleme sunucusu | Orta – SL 2 |
| **Z2 – Operasyon / Filo** | SCADA istemcisi → filo/trafik yöneticisi, harita sunucusu, fabrika Wi-Fi erişim noktaları ❓ | Orta-Yüksek – SL 2 |
| **Z3 – Araç kontrol** | Araç PLC'si, HMI/operatör paneli, robot kontrolörü → navigasyon bilgisayarı + motor sürücüleri, BMS | Yüksek – SL 2–3 |
| **Z4 – Safety** | Safety PLC, lazer tarayıcılar, E-stop, tampon, STO, kumandanın safety kanalı | En yüksek – SL 3 |
| **Z5 – Mühendislik (geçici)** | Mühendislik istasyonu / servis laptopu | Yüksek (en güçlü yetkiye sahip!) – SL 3 |

**2. Safety PLC'yi hangi bölgeye koydunuz — neden?**
**Ayrı bir bölgeye (Z4)**, araç kontrolüyle aynı yere değil. Nedenleri:
- İnsan hayatını koruyan son katman; normal PLC'nin (Z3) ele geçirilmesi safety'yi etkilememeli.
- Z4'ün dış dünyayla **hiç ağ bağlantısı olmamalı**; sadece kablolu sinyaller (E-stop, OSSD, STO) ile çalışmalı.
- Z3'e sadece **durum bilgisi verebilir** (okuma), Z3 safety'ye **komut yazamaz**.
- Konfigürasyonu sadece yerinde, fiziksel olarak, Z5'teki yetkili laptopla değiştirilebilir.

**3. Bölgeler arası geçişler (conduit) ve kontrolleri**
| Conduit | Geçiş | Ne kontrol edilecek |
|---|---|---|
| **C1** | Z0 ↔ Z1 | Firewall; ofisten OT'ye **doğrudan** geçiş yok; sadece jump server'a izinli kullanıcılar |
| **C2** | Z1 ↔ Z2 | Jump server üzerinden, MFA, oturum kaydı, sadece gerekli protokol/port |
| **C3** | Z2 ↔ Z3 (**kablosuz!**) | Wi-Fi WPA3-Enterprise (sertifikalı), araç başına kimlik, sadece görev/telemetri mesajları; uzaktan programlama portları kapalı ❓ |
| **C4** | Z3 ↔ Z4 | Tek yönlü: safety → kontrol durum bilgisi. Kontrolden safety'ye sadece alan seçimi gibi **sınırlı, safety tarafında makullük kontrolü yapılan** sinyaller (ör. tarayıcı, gerçek hızı kendi encoder'ıyla kontrol etmeli) |
| **C5** | RF kumanda ↔ Z3/Z4 | Kumanda–araç eşleştirmesi (benzersiz ID), şifreli/rolling code, sinyal kaybında dur (IEC 62745 mantığı) |
| **C6** | Z5 ↔ Z3/Z4 | Sadece fiziksel bağlantı, anahtarlı mod seçici, kişisel parola, her değişiklikte checksum karşılaştırması ve kayıt |

```
 [Z0 Kurumsal] ──C1──► [Z1 DMZ / Jump server] ──C2──► [Z2 Filo / Wi-Fi]
                                                        │
                                                       C3 (kablosuz)
                                                        ▼
 [RF Kumanda] ──C5──────────────────────────────► [Z3 Araç kontrol PLC + sürücüler]
                                                        │ C4 (tek yönlü, sınırlı)
 [Z5 Servis laptopu] ──C6 (fiziksel, anahtarlı)──►  [Z4 SAFETY: Safety PLC, lazer, E-stop, STO]
```

### Görev B — uzak erişim kararı
| Karar | Seçenekler | Seçimimiz + nedeni |
|---|---|---|
| Aracı? | Doğrudan / Jump server | **Jump server** — FADA servisi araca veya PLC'ye doğrudan bağlanmamalı; tek bir kontrollü kapıdan geçmeli |
| Tünel? | Açık / VPN | **VPN** — veriler şifreli gitmeli, internete açık port olmamalı |
| Kimlik? | Parola / MFA | **MFA** — çalınan parola tek başına yetmesin |
| Hesap? | Paylaşılan / Kişisel | **Kişisel** — kim ne değiştirdi bilinsin; işten ayrılan kişinin erişimi kapatılabilsin |
| Kayıt? | Yok / Tam oturum kaydı | **Tam oturum kaydı** — olay olursa kim ne yaptı kanıtlanabilsin |
| Zamanlama? | Sürekli açık / Varsayılan kapalı | **Varsayılan kapalı** — sadece talep edildiğinde, fabrikadan biri onaylayınca, süreli açılır |

> **Evidence Pack:** EP 05 Zone & Conduit.

---

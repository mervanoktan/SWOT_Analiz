# Safety 4.0 Bootcamp — Haftalık Lab Föyleri (FADA FTC)

Seçilen makine: **FADA Raysız Akülü Transfer Arabası (FTC)** — 5–400 ton, 30 m/dk, RF kumanda / otonom, lazer + ultrasonik + tampon, akü/dizel.

Her hafta bir föy doldurulur; yeni haftalar eklendikçe tablo güncellenir. Dosya adları `Hafta-XX_LABX_...` şeklindedir.

| Hafta | Föy (Laboratuvar) | Dosya |
|---|---|---|
| **Hafta 1** | LAB 1 · Safety Architecture → Çıktı: Safety Function Map | [Hafta-01_LAB1_Safety-Architecture.md](Hafta-01_LAB1_Safety-Architecture.md) |
| **Hafta 2** | LAB 2 · IEC 62443 Mini Workshop → Çıktı: Zone / Conduit Diagram | [Hafta-02_LAB2_IEC62443-Zone-Conduit.md](Hafta-02_LAB2_IEC62443-Zone-Conduit.md) |
| **Hafta 3** | LAB 3 · Threat & Risk + Risk Matrix → Çıktı: Risk Register | [Hafta-03_LAB3_Threat-Risk-Matrix.md](Hafta-03_LAB3_Threat-Risk-Matrix.md) |
| **Hafta 4** | LAB 4 · Secure Remote Access → Çıktı: Secure Architecture | [Hafta-04_LAB4_Secure-Remote-Access.md](Hafta-04_LAB4_Secure-Remote-Access.md) |

## Ortak kodlar (tüm haftalarda aynı)
- **SF1–SF6:** Emniyet fonksiyonları (Hafta 1)
- **Z0–Z5, C1–C6:** Bölgeler ve geçişler (Hafta 2)
- **R1–R4:** Risk senaryoları (Hafta 3)
- **G1–G5:** Emniyet bağımsızlığı gereksinimleri (Hafta 4)

## SWOT'a yansıyanlar (Hafta 1–4)
- **W (Zayıf):** Safety PL seviyesi, kumandanın şifreleme/eşleştirme yöntemi ve uzak servis erişiminin nasıl korunduğu sitede belirtilmiyor.
- **O (Fırsat):** "Safety yolu siber saldırıdan bağımsızdır" ve "güvenli uzak erişim" özelliklerini belgeleyip satış avantajına çevirmek (AB Makine Yönetmeliği 2023/1230).
- **T (Tehdit):** RF kumanda replay ve uzak servis hesabı üzerinden emniyet ayarlarının değiştirilmesi, insan yaralanmasına kadar giden senaryolar (bkz. Hafta 3, R1–R4).

> ❓ = varsayım. Eğitim çalışmasıdır; resmî denetim değildir.

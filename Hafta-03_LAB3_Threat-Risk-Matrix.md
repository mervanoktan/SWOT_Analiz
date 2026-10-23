# Hafta 3 — LAB 3 · Threat & Risk + Risk Matrix → Çıktı: Risk Register

| Ad Soyad | Tarih |
|---|---|
| Mervan Oktan | 23.10.2026 |

> **Makine:** FADA Raysız Akülü Transfer Arabası (FTC) — https://www.fada.com.tr/urun/raysiz-akulu-transfer-arabasi-ftc/  
> **Kaynak föy:** INSPECCO × KRIPTARIUM — Safety 4.0 Bootcamp, Laboratuvar Föyleri (Genel Sürüm), Laboratuvar 3 (föydeki örnek makine şişeleme hattı; sorular FTC'ye uyarlanarak cevaplandı).  
> ❓ = sitede yazmayan, bizim yaptığımız varsayım. Bu bir eğitim çalışmasıdır; resmî risk değerlendirmesi veya CE uygunluğu yerine geçmez.

## LAB 3 · Threat & Risk + Risk Matrix → Çıktı: Risk Register

**Kavram:** Senaryo = **kim → hangi kapı → ne yapar → zincir → FİZİKSEL sonuç**.
Skor = **L** (olasılık) × **S** (şiddet) × **E** (maruziyet), her biri 1–3.
Kritik 18–27 · Yüksek 9–17 · Orta 4–8 · Düşük 1–3. **S = 3 ise skor düşük olsa bile öne alınır.**

| # | Senaryo (kim → kapı → ne yapar → zincir → fiziksel sonuç) | L | S | E | Skor | Öncelik |
|---|---|---|---|---|---|---|
| **R1** | Fabrikaya giren yüklenici / ziyaretçi → **RF kumanda bağlantısı** → ucuz bir SDR cihazıyla kumanda sinyalini kaydedip tekrar gönderir (replay) → araç operatör istemeden hareket eder → yükleme yapan çalışan araç ile tezgâh arasında **sıkışır / ezilir** | 2 | 3 | 3 | **18** | **Kritik** |
| **R2** | Dış saldırgan → **paylaşılan parolalı uzak servis hesabı** → VPN/uzak masaüstü ile servis laptopuna → mühendislik yazılımıyla lazer tarayıcının koruma alanını küçültür ve hız limitini artırır → araç insanı geç algılar, durma mesafesi yetmez → yaya çalışana **çarpar** | 2 | 3 | 2 | **12** | Yüksek |
| **R3** | Oltalama (phishing) e-postası → **ofis ağı** → düz ağ yüzünden filo yöneticisine ulaşır ❓ → otonom araçlara sahte rota/görev gönderir → 300 tonluk yük ile araç dar koridorda kolona çarpar, **yük devrilir**, yakındaki kişiler yaralanır | 2 | 3 | 2 | **12** | Yüksek |

**1. En yüksek öncelikli senaryo hangisi? Neden?**
**R1 (kumanda replay).** Çünkü saldırı için ağa girmeye bile gerek yok, sadece fabrikada fiziksel olarak bulunmak ve ucuz bir cihaz yeterli; endüstriyel telsiz kumandalarda bu açık bilinen bir problem ve yükleme sırasında insan araca en yakın konumda.

**2. S=3 olup skoru düşük kalan senaryo var mı? Nasıl görünür kılacağız?**
Evet:
| # | Senaryo | L | S | E | Skor | Öncelik |
|---|---|---|---|---|---|---|
| **R4** | Kötü niyetli içeriden biri → **BMS servis arayüzü** ❓ → akü şarj/ısı limitlerini değiştirir → gece şarjda aşırı şarj → ısıl kaçak → **akü yangını**, yangına müdahale eden kişi yaralanır | 1 | 3 | 1 | **3** | Düşük → **öne alındı** |

Görünür kılmak için:
- Risk kaydında **"S=3 bayrağı"** ile ayrı renkle (kırmızı çerçeve) işaretleriz; skora göre sıralansa bile listenin en üstünde ayrı bir "insan hayatı" bölümünde dururlar.
- Her S=3 senaryosuna mutlaka bir **sorumlu kişi ve önlem** atanır; "düşük skor" diye kapatılamaz.

**Altın kural kontrolü:** Hiçbir senaryomuz "veri çalındı" ile bitmiyor; hepsi hattaki insana ne olacağına kadar gidiyor (ezilme, çarpma, yük devrilmesi, yangın). ✅

> **Evidence Pack:** EP 04 Risk Assessment.

---

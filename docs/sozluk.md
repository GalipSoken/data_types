# Sözlük

[← İçindekiler](../README.md)

---

Rehberde geçen terimlerin kısa açıklamaları.

| Terim | Açıklama |
|---|---|
| **ABT** (Analytical Base Table) | Modelleme için hazırlanmış, her satırın bir müşteri/hesap/dönem olduğu, çok kolonlu analiz tablosu. |
| **Arrow** | Veriyi kolon kolon ve verimli tutan bir bellek formatı. pandas'ın yeni sürümleri ve Polars bunu kullanır. |
| **BIN** | Kart numarasının ilk 6–8 hanesi; kartı çıkaran bankayı ve kart tipini gösterir. |
| **bit / byte** | Bit: 0 veya 1. Byte: 8 bit; bir harf veya 0–255 arası bir sayı. |
| **Collation** | Bir veritabanında yazıların nasıl sıralanıp karşılaştırılacağını belirleyen dil kuralı (örneğin Türkçe). |
| **Columnstore / kolon bazlı saklama** | Veriyi satır satır değil kolon kolon saklamak. Analiz sorgularında çok daha hızlı ve küçük. |
| **DataFrame** | pandas ve Polars'ta tablo. |
| **Decimal (kesin ondalık)** | Ondalıklı sayıyı tam olarak saklayan tip. Para için kullanılır. |
| **dtype** | pandas'ta bir kolonun veri tipi. |
| **Feature (özellik)** | Modelin tahmin yaparken kullandığı kolon. |
| **Flag** | Evet/hayır bilgisi tutan kolon. |
| **Float (yaklaşık ondalık)** | Ondalıklı sayıyı yaklaşık olarak saklayan, hızlı tip. `float32` ~7 hane, `float64` ~15 hane güvenilir. |
| **IBAN** | Uluslararası banka hesap numarası; Türkiye'de 26 karakter. |
| **ISIN** | Menkul kıymetlerin uluslararası 12 karakterlik kimliği. |
| **Join (birleştirme)** | İki tabloyu ortak bir anahtar kolonu üzerinden yan yana getirmek. |
| **KVKK** | Kişisel Verilerin Korunması Kanunu. |
| **NaN / NaT / pd.NA** | Python'da boş değerin farklı gösterimleri (sayı, tarih, genel). |
| **NACE** | Avrupa'da ve Türkiye'de kullanılan ekonomik faaliyet sınıflaması. |
| **NULL** | "Değer yok." Sıfır veya boş yazı değildir. |
| **OOM** (Out Of Memory) | Bellek yetersiz hatası; tablo RAM'e sığmadığında oluşur. |
| **Parquet** | Veriyi kolon kolon, sıkıştırarak ve tip bilgisiyle saklayan dosya formatı. |
| **Partition (bölümleme)** | Büyük bir tabloyu (örneğin aylara göre) parçalara ayırmak; sorgular sadece ilgili parçayı okur. |
| **PCI-DSS** | Kart verisinin güvenliği için uluslararası standart. |
| **Precision / scale** | Kesin ondalık tipte toplam hane sayısı / virgülden sonraki hane sayısı. `(18,2)` gibi. |
| **RAM** | Bilgisayarın çalışma belleği. Python tabloları burada tutar. |
| **Recipe** | Dataiku'da veriyi dönüştüren adım (SQL, Python veya görsel). |
| **Sentinel (özel değer)** | "Bilinmiyor" veya "sonsuz" anlamında kullanılan özel değer: `9999-12-31`, `-1` gibi. |
| **Surrogate key (vekil anahtar)** | Gerçek kimlik (TC kimlik no gibi) yerine kullanılan anlamsız sıra numarası. |
| **Target encoding** | Çok değerli bir kategoriyi, her değerin hedef değişkendeki ortalamasıyla sayıya çevirme yöntemi. |
| **Tepe bellek** (peak memory) | Bir işlem sırasında anlık olarak ihtiyaç duyulan en yüksek bellek; genelde tablo boyutunun 2–3 katı. |
| **UTC** | Evrensel saat. Türkiye saati UTC+3'tür. |
| **Valör** | Paranın/menkulün gerçekten el değiştirdiği, faizin işlemeye başladığı tarih. |

---

[← İçindekiler](../README.md)

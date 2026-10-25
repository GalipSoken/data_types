# 9. Değişken Rehberi: Her Kolon İçin Doğru Tip

[← Önceki: Dataiku](08-dataiku.md) · [İçindekiler](../README.md) · [Sonraki: Karar Tablosu →](10-karar-tablosu.md)

---

## Kısaca

Bu bölüm, finansal veride en sık karşılaşılan kolonları tek tek ele alır. Her kolon için:

* **Ne tutar?** Hangi değerleri alabilir?
* **Tuzak:** Yanlış tip seçilirse ne bozulur?
* **Öneri:** Veritabanında ve Python'da hangi tip?
* **Modelde:** Makine öğrenmesinde nasıl kullanılır?

Önce önemli bir ayrım: **kimlik** ile **sayı** farklıdır.

> TC kimlik numarası rakamlardan oluşur ama bir **sayı değildir**. İki TC kimlik numarasını toplamak veya
> ortalamasını almak anlamsızdır. Bu tür kolonlar **tanımlayıcıdır** (kimliktir): tabloları birbirine
> bağlamaya yarar, modelde özellik (feature) olarak kullanılmaz. Vergi no, hesap no, kart no, sözleşme no,
> müşteri no da böyledir.

---

## Kimlik kolonları

### TC Kimlik No (TCKN)

| | |
|---|---|
| **Ne tutar?** | 11 haneli numara. İlk hane 0 olamaz, son iki hane kontrol hanesidir. |
| **Tuzak** | En büyük değer ~100 milyardır; 4 byte'lık tam sayıya (en fazla 2,1 milyar) **sığmaz**. |
| **Öneri (veritabanı)** | `CHAR(11)` veya 8 byte'lık tam sayı (`NUMBER(11)` / `bigint`) |
| **Öneri (Python)** | `int64` (8 byte). Yazı olarak ~60 byte tutar. **Asla ondalıklı sayı (float) değil.** |
| **Modelde** | Kullanılmaz. |
| **Hassasiyet** | **Kişisel veridir (KVKK).** Analiz ortamında açık hali bulunmamalı. |

**Analiz ortamı için en iyi uygulama:** TC kimlik no yerine anlamsız bir sıra numarası kullanın
(`musteri_sk` = 1, 2, 3…). TC kimlik no ↔ sıra numarası eşleşmesi güvenli bir tabloda durur.
Buna **surrogate key** (vekil anahtar) denir. Hem KVKK açısından güvenlidir hem de 4 byte'a sığar.

### Vergi Kimlik No (VKN)

| | |
|---|---|
| **Ne tutar?** | 10 haneli numara. **Başında 0 olabilir.** |
| **Tuzak** | Sayıya çevrilirse baştaki sıfırlar silinir: `0012345678` → `12345678` |
| **Öneri** | Yazı: `CHAR(10)` / `VARCHAR2(10 CHAR)` / `varchar(10)`. Python'da `string`. |
| **Modelde** | Kullanılmaz. |

Şahıs firmalarında VKN yerine TC kimlik no kullanılabilir. Bu yüzden şu yapı sık kullanılır:
`kimlik_tipi` (T = TCKN, V = VKN) + tek bir `kimlik_no` kolonu (en fazla 11 karakter).

### Müşteri Numarası

| | |
|---|---|
| **Ne tutar?** | Bankanın verdiği iç numara; genelde sıralı tam sayı. |
| **Öneri** | 4 byte tam sayı (`NUMBER(10)` / `integer` / `int`) → Python `int32`. 2,1 milyarı aşabilecekse 8 byte. |
| **Dikkat** | **Tüm tablolarda aynı tipte** olmalı. Bir tabloda yazı, diğerinde sayıysa birleştirme (join) yavaşlar, bazen hiç eşleşmez. |
| **Modelde** | Kullanılmaz. |

### Sözleşme No

| | |
|---|---|
| **Ne tutar?** | Kaynağa göre değişir: sıralı sayı veya `KRD2026000123` gibi harf+rakam. |
| **Öneri** | Sadece rakamsa 8 byte tam sayı (`bigint`). Harf içeriyorsa `VARCHAR(20)`. |
| **İpucu** | Numaranın içine gömülü bilgiyi (örneğin `KRD` = kredi) yazı parçalayarak kullanmayın; ayrı bir "ürün tipi" kolonuna çıkarın. |
| **Modelde** | Kullanılmaz. |

### Kart Numarası

| | |
|---|---|
| **Ne tutar?** | Genelde 16 hane. İlk 6–8 hane kartı çıkaran bankayı ve kart tipini gösterir (BIN). Türkiye'nin yerli kartı Troy **9792** ile başlar. |
| **Tuzak** | **Asla sayı olarak saklanmaz.** 8 byte'lık ondalıklı sayı (float64) 16 haneli büyük sayıları tam tutamaz: `9792123456789013` → `9792123456789012`. Excel de 15. haneden sonrasını sıfırlar. |
| **Öneri** | Analiz ortamında maskeli hali (`979212******9013`, `CHAR(16)`) veya anlamsız bir sıra numarası. |
| **Modelde** | Numaranın kendisi kullanılmaz. BIN (ilk 6–8 hane) ayrı bir kolon olarak kart tipini anlatabilir. |
| **Hassasiyet** | **Kart verisi güvenlik standardı (PCI-DSS) kapsamında.** Açık kart numarası analiz ortamında olmamalı. |

### Hesap Numarası ve IBAN

| | |
|---|---|
| **Ne tutar?** | İç hesap no genelde şube kodu + hesap no + ek no şeklindedir; baştaki sıfırlar anlamlıdır. IBAN: `TR` + 24 rakam = **26 karakter**. |
| **Tuzak** | Sayıya çevrilirse baştaki sıfırlar silinir; IBAN'daki harfler sayıya zaten çevrilemez. |
| **Öneri** | IBAN `CHAR(26)`. İç hesap no `VARCHAR(20)` veya parçalarına ayrılmış (şube kodu 2 byte, hesap no 4 byte, ek no 2 byte). Analizde sıra numarası. |
| **Modelde** | Kullanılmaz. Şube kodu ayrı bir kolon olarak kullanılabilir. |

---

## Yazı kolonları

### Müşteri Adı / Firma Adı

| | |
|---|---|
| **Ne tutar?** | Serbest yazı, Türkçe karakterler, uzun firma unvanları ("… SANAYİ VE TİCARET ANONİM ŞİRKETİ"). |
| **Tuzak** | Oracle'da `VARCHAR2(50)` yazılırsa Türkçe karakterler sığmayabilir ([Bölüm 4](04-metin.md)). Python'da `category` yapılırsa yer kazanmak yerine kaybeder. |
| **Öneri** | Oracle `VARCHAR2(100 CHAR)` (firma için 250), PostgreSQL `text`, MSSQL `varchar(250)` + Türkçe collation. Python'da `string`. |
| **Modelde** | Kullanılmaz (kişisel veri + model ezberler). |
| **Kullanım alanı** | Aynı firmayı farklı kayıtlarda eşleştirmek (grup firma tespiti gibi). Bunun için standartlaştırılmış bir kopya üretin: Türkçe büyük harf, "A.Ş." / "ANONİM ŞİRKETİ" birleştirme, noktalama temizliği. |

---

## Kod (kategori) kolonları

### NACE Kodu (ekonomik faaliyet kodu)

| | |
|---|---|
| **Ne tutar?** | Firmanın faaliyet alanını gösteren **katmanlı** kod. Harf (A–U, 21 ana bölüm) → 2 hane → 3 hane → 4 hane (615 sınıf) → Türkiye'de 6 hane. Örnek: `01.11` (tahıl yetiştiriciliği), `10.71.01`. |
| **Tuzak** | Sayıya çevrilirse `01.11` → `1.11`, `10.10` → `10.1`. Tarım kodları (01–03) bozulur, katmanlar kaybolur. |
| **Öneri** | Yazı: `VARCHAR(8)` + ayrı bir NACE kod tablosu. Python'da `category`. |
| **Modelde** | Katmanları ayrı kolonlar olarak üretin: `nace_harf` (21 değer), `nace_2` (~88), `nace_4` (615). Çok değerli katmanlarda hedef oranına dayalı kodlama (target encoding) veya LightGBM gibi kategori destekleyen modeller. |

### Sektör Kodu / Sektör Adı

| | |
|---|---|
| **Ne tutar?** | Bankanın kendi sektör sınıflaması; onlarca–yüzlerce değer. |
| **Öneri** | Ana tabloda sadece **kod** (2 byte tam sayı veya kısa yazı); **ad** ayrı kod tablosunda. Python'da `category`. |
| **Modelde** | Kod kategori olarak kullanılır. Ad, kodla aynı bilgiyi taşıdığı için kullanılmaz. |

### Cinsiyet

| | |
|---|---|
| **Ne tutar?** | `E`, `K`. Firmalar için uygulanamaz; bazı bireysel müşterilerde bilinmez. |
| **Tuzak** | "Bilinmiyor" ile "uygulanamaz" aynı boş değere sıkıştırılırsa bilgi kaybolur. |
| **Öneri** | `CHAR(1)` + izin verilen değerler kuralı (`E`, `K`, `B` = bilinmiyor). Firma olduğu `kimlik_tipi`'nden anlaşılır. Python'da `category`. |
| **Modelde** | Kredi kararı modellerinde cinsiyeti özellik olarak kullanmak ayrımcılık açısından sorgulanır. Genelde sadece modelin farklı gruplara adil davranıp davranmadığını ölçmek için tutulur. |

### İşlem Tipi

| | |
|---|---|
| **Ne tutar?** | EFT, HAVALE, FAST, POS, ATM, VİRMAN, SWIFT… Az sayıda değer, çok tekrar. |
| **Öneri** | Kod tablosuna bağlı küçük sayı (Oracle `NUMBER(3)`, PostgreSQL `smallint`, MSSQL `tinyint`) veya kısa yazı `VARCHAR(10)`. Python'da `category`. |
| **Ölçek** | 100 milyon satırlık işlem tablosunda 20 karakterlik yazı yerine 1 byte'lık kod: ~1,5 GB tasarruf. |
| **Modelde** | Müşteri bazında işlem tipine göre adet ve tutar kolonlarına dönüştürülür ("son 3 ayda FAST adedi" gibi). |

### Döviz Kodu

| | |
|---|---|
| **Ne tutar?** | Uluslararası 3 harfli kod: `TRY`, `USD`, `EUR`. |
| **Öneri** | `CHAR(3)`, zorunlu alan. Python'da `category`. |
| **Dikkat** | Her tutarın yanında olmalı. Boşsa "TL'dir" diye varsaymayın. |

---

## Sayı kolonları

### Evet/Hayır (Flag) Değişkenleri
Ayrıntısı [Bölüm 7](07-evet-hayir.md)'de. Kısaca: Oracle `NUMBER(1)`, PostgreSQL `boolean`, MSSQL `bit`;
Python'da `int8` veya boş destekli `boolean`. Boş değerin anlamını koruyun.

### Tutar Değişkenleri (bakiye, limit, risk, ciro)

| | |
|---|---|
| **Ne tutar?** | Negatif olabilir (borç, net pozisyon). Çoğu değer 0'da toplanır, az sayıda çok büyük değer vardır. Büyük şirketlerde trilyonlarca TL'ye çıkabilir. |
| **Öneri (veritabanı)** | Kesin ondalıklı `(18,2)` + yanında döviz kodu. TL karşılığı ayrı kolonda. |
| **Öneri (Python)** | Model için `float32`; toplam ve mutabakat için `float64` veya kesin tip ([Bölüm 3](03-sayilar.md)). |
| **Tuzak** | Veritabanından `Decimal` nesnesi olarak gelebilir: çok yavaş ve büyük. Okur okumaz sayıya çevirin. |
| **Boş değer** | "Hesap yok" mu, "veri gelmedi" mi? ([Bölüm 6.1](06-null.md#61-önce-iş-anlamı-boş-ne-demek)) |
| **Modelde** | Genelde logaritması, oranları (limit kullanım oranı gibi) ve dönemler arası değişimi kullanılır. |

### Adet Değişkenleri (işlem sayısı, ürün sayısı, gecikme günü)

| | |
|---|---|
| **Ne tutar?** | Negatif olmayan tam sayılar; çoğu küçük. Ürün sayısı 0–20, aylık işlem 0–5.000, gecikme günü 0–3.650. |
| **Öneri (veritabanı)** | 2 byte tam sayı (`NUMBER(5)` / `smallint`); büyük olabilecekler 4 byte. |
| **Öneri (Python)** | `int16`; boş değer varsa `Int16`. Güvenli varsayılan `int32`. |
| **Tuzak** | Tipi **bugünkü** en büyük değere göre değil, olabilecek en büyük değere göre seçin; küçük tip taşarsa uyarı vermeden bozulur. MSSQL'de toplamı ayrıca büyük tipe çevirin. |

---

## Piyasa ve işlem verisi

| Kolon | Ne tutar? | Veritabanı | Python | Boşsa ne demek? |
|---|---|---|---|---|
| **ISIN** | Menkul kıymetin uluslararası 12 karakterlik kimliği (`TRAGARAN91N1`) | `CHAR(12)` | `category` | Menkul kıymet değil |
| **Sembol** (ticker) | Borsadaki kısa ad | `VARCHAR(16)` + borsa kodu | `category` | — |
| **Fiyat** | 4–6 ondalık | Kesin ondalık `(18,6)` | `float64` | İşlem görmedi → son fiyatı kullan, işaretle |
| **Kur** | 4–8 ondalık | Kesin ondalık `(18,8)` | `float64` | Tatil → önceki iş gününün kuru (kural yazılı olmalı) |
| **Faiz oranı** | Birimi belli oran | Kesin ondalık `(12,8)` | `float64` | Değişken faiz, henüz belli değil |
| **Getiri** | Küçük, + veya − | 8 byte float | `float64` | İlk gün |
| **İşlem hacmi** | ≥ 0, büyük | 8 byte tam sayı (kesirli pay varsa kesin ondalık) | `int64` | İşlem yoksa **0** (boş değil) |
| **İşlem zamanı** | Saniye altı hassasiyet | Saat dilimli zaman tipi | `datetime64[us, UTC]` | — |
| **Valör / vade tarihi** | Gün | Saatsiz tarih | `datetime64[s]` | Vadesiz ürün |
| **Dönem** | `202610` | 4 byte tam sayı | `int32` | — |

---

[← Önceki: Dataiku](08-dataiku.md) · [İçindekiler](../README.md) · [Sonraki: Karar Tablosu →](10-karar-tablosu.md)

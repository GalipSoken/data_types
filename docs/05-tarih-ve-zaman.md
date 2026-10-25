# 5. Tarih ve Zaman

[← Önceki: Metin](04-metin.md) · [İçindekiler](../README.md) · [Sonraki: Boş Değer (NULL) →](06-null.md)

---

## Kısaca

Tarih, finansal veride en çok hata üreten konudur. Bunun üç nedeni var:

1. **"Tarih" tek bir şey değildir.** Bir işlemin birden fazla tarihi olur.
2. **Bazı tarih tipleri gizlice saat de taşır**; saat bilgisi fark edilmeden sorguları bozar.
3. **Saat dilimleri** veri bir sistemden ötekine geçerken tarihleri bir gün kaydırabilir.

## 5.1 Bir işlemin birden fazla tarihi vardır

| Tarih | Ne demek? | Örnek |
|---|---|---|
| **İşlem tarihi** | İşlemin yapıldığı gün | Hisseyi pazartesi aldınız |
| **Valör tarihi** | Paranın/menkulün gerçekten el değiştirdiği, faizin işlemeye başladığı gün | Takas çarşamba (T+2) |
| **Muhasebe tarihi** | Kaydın deftere işlendiği gün | Gece 23:30'da yapılan EFT ertesi günün kaydı olabilir |
| **Olay zamanı** | İşlemin saniyesine kadar zamanı | Kart onayı 14:32:05.123 |
| **Dönem** | Raporun ait olduğu ay | 2026 Ekim ay sonu bakiyesi |
| **Vade tarihi** | Kredinin/tahvilin bittiği gün | 2031-10-25 |

**Temel kural:**
* İş günüyle ilgili tarihler (işlem, valör, muhasebe, vade) → **saatsiz tarih**.
* Olay zamanı → **tarih + saat + saat dilimi**.

## 5.2 Tipler

| Ne saklıyoruz? | Oracle | PostgreSQL | MSSQL | Dataiku | Python |
|---|---|---|---|---|---|
| Sadece tarih | `DATE` *(dikkat: saat de taşır!)* | `date` (4 B) | `date` (3 B) | `dateonly`* | `datetime64[s]` |
| Tarih + saat (saniye) | `DATE` (7 B) | `timestamp(0)` | `datetime2(0)` (6 B) | `datetimenotz`* | `datetime64[s]` |
| Tarih + saat (mikrosaniye) | `TIMESTAMP(6)` | `timestamp` (8 B) | `datetime2(6)` (8 B) | — | `datetime64[us]` |
| Saat dilimli zaman | `TIMESTAMP WITH TIME ZONE` | `timestamptz` (8 B) | `datetimeoffset` | `date` (UTC) | `datetime64[us, UTC]` |
| Dönem (yıl-ay) | `NUMBER(6)` → 202610 | `integer` | `int` | `int` | `int32` |

\* Dataiku'nun yeni sürümlerinde bulunan tipler; eski sürümlerde tek tip `date`'tir (aşağıya bakın).

> Bankacılık veri ambarlarında tarihin `20261025` gibi **tam sayı** olarak saklandığı sık görülür.
> Saklamak için verimlidir (4 byte) ama gün farkı hesaplamak için mutlaka gerçek tarih tipine çevrilmelidir.
> Aksi halde `20261101 − 20261031 = 70` gün çıkar.

## 5.3 Ortamlara göre tuzaklar

### Oracle

**`DATE` her zaman saat de taşır.** Adı "tarih" olsa da Oracle `DATE` aslında "tarih + saat (saniyeye kadar)"dır.

```sql
-- islem_tarihi = 2026-10-25 14:30:00 olan kayıt bu sorguya GELMEZ:
WHERE islem_tarihi = DATE '2026-10-25'

-- Doğrusu: o günün başından ertesi günün başına kadar
WHERE islem_tarihi >= DATE '2026-10-25' AND islem_tarihi < DATE '2026-10-26'
```

İki Oracle tarihi arasındaki fark **küsuratlı gün** verir (1,5 = 1 gün 12 saat). Faiz hesabında saat
bilgisi taşıyan tarihler kuruş farkı yaratır. Tarih kolonlarını yazarken saatini sıfırlayın (`TRUNC`).

Tarihi yazı olarak yazmayın: `WHERE tarih = '25/10/2026'` sorgusu, bağlanan bilgisayarın dil ayarına göre
çalışır veya çalışmaz. Her zaman `DATE '2026-10-25'` biçimini kullanın.

### PostgreSQL

* `date` gerçekten sadece tarihtir. Oracle'daki sorun burada yoktur.
* `timestamptz` adındaki "tz" (time zone) yanıltıcıdır: saat dilimini **saklamaz**. Zamanı evrensel saate
  (UTC) çevirip saklar, okurken sizin saat diliminize çevirerek gösterir. Olay zamanları için doğru tip budur.
* En hassas birim mikrosaniyedir (saniyenin milyonda biri); nanosaniye yoktur.

### MSSQL

* **Eski `datetime` tipi milisaniyeyi yuvarlar:** `23:59:59.999` değeri **ertesi günün 00:00:00'ı** olarak
  kaydedilir. "Gün sonuna kadar" filtresi bu yüzden ertesi günün kayıtlarını da getirir.
  Yeni tablolarda `datetime` yerine `date` veya `datetime2` kullanın.
* `smalldatetime` 2079 yılında biter. 50 yıllık konut kredisi vadesi buna sığmaz.
* **`DATEDIFF` süre ölçmez, sınır sayar:**
  ```sql
  DATEDIFF(year, '2025-12-31', '2026-01-01')  →  1   (aradaki fark 1 gün!)
  ```
  Yaş, kıdem, gecikme ayı hesaplarında yanlış sonuç verir.

### Python (pandas)

* **pandas'ın varsayılan tarih tipi 2262 yılında biter** (ve 1677'den önce başlayamaz).
  Veri ambarlarında "bitiş tarihi yok" anlamında kullanılan **`9999-12-31`** değeri bu aralığa sığmaz.
  pandas'ın eski sürümleri hata verir veya değeri sessizce boşa çevirir. Yeni sürümler daha geniş birimler
  (saniye, mikrosaniye) kullanarak bu sorunu çözer. Dataiku'daki Python ortamınızın pandas sürümünü bilmeden
  bu davranışa güvenmeyin.
* Saat dilimli ve saat dilimsiz iki tarih kolonu birbiriyle karşılaştırılamaz; pandas hata verir.

## 5.4 Ay ekleme: dört ortam, iki farklı cevap

Taksit ve vade tarihleri üretirken "bir ay sonra" çok sık hesaplanır:

| Başlangıç | Oracle (`ADD_MONTHS`) | PostgreSQL | MSSQL (`DATEADD`) | pandas |
|---|---|---|---|---|
| 31 Ocak 2026 + 1 ay | 28 Şubat | 28 Şubat | 28 Şubat | 28 Şubat |
| **28 Şubat 2026 + 1 ay** | **31 Mart** | 28 Mart | 28 Mart | 28 Mart |

Oracle'ın kuralı: başlangıç ayın **son günüyse**, sonuç da ayın son günü olur. Diğerleri gün numarasını
korur. Bir taksit planı Oracle'da üretilip Python'da kontrol edilirse, Şubat'tan sonraki tüm taksitler farklı çıkar.
"Ay sonu" mantığı gerekiyorsa bunu açıkça kodlayın (`LAST_DAY` / `EOMONTH` / `MonthEnd`).

## 5.5 Saat dilimi

* **Türkiye 2016'dan beri tüm yıl UTC+3'tür** (yaz/kış saati yok).
* ABD ve Avrupa ise yaz/kış saati uygular. New York ile İstanbul arasındaki fark yılın bir kısmında
  7, bir kısmında 8 saattir. Yurt dışı piyasa fiyatlarını Türkiye tarihleriyle eşleştirirken saat dilimini
  sabit bir fark (+3) olarak değil, **bölge adıyla** (`Europe/Istanbul`, `America/New_York`) saklayın.

### Dataiku'da bir gün kayması

Dataiku'nun klasik `date` tipi her tarihi **evrensel saatle (UTC)** saklar. İstanbul UTC+3 olduğu için:

```
Veritabanında:  2026-10-25 00:00 (İstanbul)
Dataiku'da:     2026-10-24T21:00:00Z
Tarihe çevir:   2026-10-24   ← bir gün geri!
```

Çözüm: Sadece tarih olan kolonlar için Dataiku'nun "sadece tarih" tipini kullanın veya bağlantı ayarlarında
saat dilimini açıkça belirtin. Her yeni veri kaynağında birkaç kaydı gözle kontrol edin.

## 5.6 Gün sayımı

Faiz, iki tarih arasındaki gün sayısıyla hesaplanır ve ürüne göre farklı kurallar vardır
(gerçek gün/360, gerçek gün/365, her ay 30 gün/360 gibi). Ortamlar tarih farkını farklı hesapladığı için
(Oracle küsuratlı gün, MSSQL sınır sayımı, PostgreSQL tam gün) gün sayımını **tek bir yerde**, açıkça
yazılmış bir fonksiyonla yapın ve iki tarihi de **saatsiz** tutun.

## 5.7 "Özel anlamlı" tarihler

Bazı sistemler bilinmeyen veya sonsuz tarih için özel değerler kullanır:

| Değer | Genelde anlamı | Sorun |
|---|---|---|
| `9999-12-31` | Bitiş yok, hâlâ geçerli | pandas'ın eski sürümlerinde aralık dışı |
| `1900-01-01` | Bilinmiyor | Ortalama yaş hesabında müşteri 126 yaşında görünür |
| `0001-01-01` | Bilinmiyor | pandas'ta aralık dışı |
| Boş (NULL) | Bilinmiyor | **Önerilen** |

**Kural:** "Bilinmiyor" için boş değer (NULL); "bitiş yok" için **tek** bir özel değer (`9999-12-31`).
Bu anlaşmayı veri sözlüğüne yazın ve analize alırken özel değerleri boşa veya ayrı bir evet/hayır kolonuna çevirin.

---

[← Önceki: Metin](04-metin.md) · [İçindekiler](../README.md) · [Sonraki: Boş Değer (NULL) →](06-null.md)

# 8. Dataiku'da Veri Tipleri

[← Önceki: Evet/Hayır Değişkenleri](07-evet-hayir.md) · [İçindekiler](../README.md) · [Sonraki: Değişken Rehberi →](09-degisken-rehberi.md)

---

## Kısaca

Dataiku veriyi Oracle, PostgreSQL veya MSSQL gibi kaynaklardan okur, kendi tiplerine çevirir ve Python'a
aktarır. Yani veri iki kez tip değiştirir. Bu bölüm, bu geçişlerde nelere dikkat edileceğini anlatır.

## İki kavram: "storage type" ve "meaning"

Dataiku'da bir kolonun iki ayrı etiketi vardır:

| Kavram | Ne demek? | Örnek |
|---|---|---|
| **Storage type** (saklama tipi) | Verinin fiziksel olarak nasıl tutulduğu. Bellek, disk ve veritabanı tipini **bu** belirler. | `string`, `int`, `bigint`, `double`, `boolean`, `date` |
| **Meaning** (anlam) | Değerin ne olduğuna dair etiket. Sadece kontrol ve görselleştirme içindir. | "Integer", "Text", "Gender", "Country" |

Bir kolonun meaning'i "Integer" görünüp storage type'ının `string` olması mümkündür. Tip konuşurken
**storage type**'a bakın.

## Dataiku'nun tipleri

| Dataiku | Ne tutar? | Python'daki eşdeğer boyut |
|---|---|---|
| `tinyint` | 1 byte tam sayı | `int8` |
| `smallint` | 2 byte tam sayı | `int16` |
| `int` | 4 byte tam sayı | `int32` |
| `bigint` | 8 byte tam sayı | `int64` |
| `float` | 4 byte ondalıklı | `float32` |
| `double` | 8 byte ondalıklı | `float64` |
| `boolean` | Evet/hayır | `bool` |
| `string` | Yazı | `object` / `string` |
| `date` | Tarih + saat (evrensel saat, UTC) | `datetime64` (UTC) |
| `dateonly`, `datetimenotz` | Sadece tarih / saat dilimsiz tarih-saat *(yeni sürümler)* | `datetime64` |

**Kesin ondalıklı (decimal) tip yoktur.** Oracle'daki `NUMBER(18,2)` Dataiku'da genelde `double` olarak görünür.
Kuruşu kuruşuna tutması gereken toplamları Python'a çekip toplamak yerine veritabanında (SQL recipe ile) yapın.

## Sık yaşanan sorunlar

### 1. Tip tahmini baştaki sıfırları siler

CSV veya Excel dosyası yüklediğinizde Dataiku kolon tiplerini **verilere bakarak tahmin eder**
("Infer types from data"). Vergi no, hesap no, NACE kodu gibi kolonları sayı sanabilir:

```
0012345678  →  12345678        (vergi no bozuldu)
01.11       →  1.11            (NACE kodu bozuldu)
```

**Çözüm:** Bu kolonların tipini şemada elle `string` yapın.

### 2. Tarihler bir gün geri kayar

Klasik `date` tipi her şeyi evrensel saatle (UTC) tutar. İstanbul'da gece yarısı olan bir tarih,
UTC'de bir önceki günün 21:00'ıdır. Ayrıntısı [Bölüm 5.5](05-tarih-ve-zaman.md#55-saat-dilimi)'te.

### 3. NULL ile boş yazı karışır

Dosya tabanlı veri setlerinde (CSV gibi) NULL ile boş yazı (`""`) çoğu zaman ayırt edilemez.
Oracle'dan gelen veride bu ayrım zaten yoktur ([Bölüm 6.2](06-null.md#62-en-büyük-fark-oracleda-boş-yazı--null)).

### 4. Uzunluk verilmeyen yazı kolonları

Dataiku sizin yerinize bir veritabanı tablosu oluştururken yazı kolonlarına uzunluk belirtmezseniz
geniş, varsayılan uzunlukta kolonlar açılır. Kısa kodlar için (döviz kodu, cinsiyet) uzunluğu şemada belirtin.

## Python recipe'inde doğru okuma

```python
import dataiku

ds = dataiku.Dataset("musteri_abt")

# 1) Sadece gereken kolonları oku
# 2) Tipleri Dataiku şemasından al, pandas'ın tahmin etmesine izin verme
df = ds.get_dataframe(
    columns=["musteri_sk", "segment_kodu", "nace_kodu", "bakiye"],
    infer_with_pandas=False,
)

# Büyük tablolarda parça parça oku (örneğin 500 bin satırlık parçalar)
for parca in ds.iter_dataframes(chunksize=500_000, infer_with_pandas=False):
    ...
```

Yazarken iki seçenek vardır:
* `write_with_schema(df)` → Şemayı DataFrame'deki tiplere göre **otomatik** oluşturur.
* `write_dataframe(df)` → Önceden tanımladığınız şemayı korur. Tipleri kontrol altında tutmak istiyorsanız bunu kullanın.

---

[← Önceki: Evet/Hayır Değişkenleri](07-evet-hayir.md) · [İçindekiler](../README.md) · [Sonraki: Değişken Rehberi →](09-degisken-rehberi.md)

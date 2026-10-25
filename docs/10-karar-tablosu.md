# 10. Karar Tablosu ve Örnek Tablo Tanımları

[← Önceki: Değişken Rehberi](09-degisken-rehberi.md) · [İçindekiler](../README.md) · [Sonraki: Python ile Uygulama →](11-python-ile-uygulama.md)

---

## Kısaca

Önceki bölümlerdeki tüm önerilerin tek sayfalık özeti. Bir kolonun tipine karar verirken bu tabloya bakabilirsiniz.
Gerekçeler için ilgili bölüme gidin.

## Karar tablosu

| Kolon | Kural | Oracle | PostgreSQL | MSSQL | Dataiku | Python | Modelde |
|---|---|---|---|---|---|---|---|
| TC kimlik no | 11 hane, ilk ≠ 0 | `CHAR(11)` / `NUMBER(11)` | `char(11)` / `bigint` | `char(11)` / `bigint` | `string` / `bigint` | `int64` veya sıra no | Kullanılmaz (KVKK) |
| Vergi no | 10 hane, başı 0 olabilir | `VARCHAR2(10 CHAR)` | `varchar(10)` | `varchar(10)` | `string` | `string` veya sıra no | Kullanılmaz |
| Müşteri no | İç numara | `NUMBER(10)` | `integer` | `int` | `int` | `int32` | Kullanılmaz |
| Müşteri adı | Serbest yazı | `VARCHAR2(100 CHAR)` | `text` | `varchar(100)` | `string` | `string` | Kullanılmaz |
| Firma adı | Uzun unvan | `VARCHAR2(250 CHAR)` | `text` | `varchar(250)` | `string` | `string` | Kullanılmaz |
| NACE kodu | `01.11`, katmanlı | `VARCHAR2(8 CHAR)` | `varchar(8)` | `varchar(8)` | `string` | `category` | Katmanlara ayır |
| Sektör kodu | Yüzlerce değer | `NUMBER(4)` | `smallint` | `smallint` | `smallint` | `category` | Kategori |
| Sektör adı | Koddan türer | kod tablosu | kod tablosu | kod tablosu | `string` | `category` | Kullanılmaz |
| Cinsiyet | E / K / B | `CHAR(1)` | `char(1)` | `char(1)` | `string` | `category` | Adillik analizi |
| Flag | 1 / 0 / boş | `NUMBER(1)` | `boolean` | `bit` | `boolean` | `int8` / `boolean` | Evet/hayır + eksik |
| Tutar | ±, döviz cinsli | `NUMBER(18,2)` | `numeric(18,2)` | `decimal(18,2)` | `double` | `float32` / `float64` | Sürekli |
| Adet | ≥ 0, küçük | `NUMBER(5)` | `smallint` | `smallint` | `smallint` | `int16` / `Int16` | Sayısal |
| Sözleşme no | Sayı veya harf+rakam | `NUMBER(19)` / `VARCHAR2(20 CHAR)` | `bigint` / `varchar(20)` | `bigint` / `varchar(20)` | `bigint` / `string` | `int64` veya sıra no | Kullanılmaz |
| Kart no | 16 hane, PCI | maskeli `CHAR(16)` | maskeli `char(16)` | maskeli `char(16)` | `string` | `string` veya sıra no | BIN ayrı kolon |
| Hesap no / IBAN | Başı 0, 26 karakter | `VARCHAR2(26 CHAR)` | `varchar(26)` | `varchar(26)` | `string` | sıra no | Kullanılmaz |
| İşlem tipi | Az değer | `NUMBER(3)` | `smallint` | `tinyint` | `tinyint` / `string` | `category` | Adet/tutar kolonlarına dönüştür |
| Döviz kodu | 3 harf | `CHAR(3)` | `char(3)` | `char(3)` | `string` | `category` | Kategori |
| İşlem/valör/vade tarihi | Gün | `DATE` (saatsiz) | `date` | `date` | `dateonly` | `datetime64[s]` | Gün farkı, ay |
| İşlem zamanı | Saniye altı | `TIMESTAMP(6) WITH TIME ZONE` | `timestamptz` | `datetime2(6)` | `date` | `datetime64[us, UTC]` | Saat, gün içi |
| Kur / fiyat | 4–8 ondalık | `NUMBER(18,8)` | `numeric(18,8)` | `decimal(18,8)` | `double` | `float64` | Getiriye çevir |
| Faiz oranı | Birimi belli | `NUMBER(12,8)` | `numeric(12,8)` | `decimal(12,8)` | `double` | `float64` | Sürekli |

## Örnek tablo tanımları

Aynı tablonun (kısaltılmış) üç veritabanındaki tanımı. Kimlikler analiz katmanında sıra numarasına
(`musteri_sk`) çevrilmiş durumda.

### Oracle

```sql
CREATE TABLE musteri_aylik (
    donem_id          NUMBER(6)          NOT NULL,          -- 202610
    musteri_sk        NUMBER(10)         NOT NULL,          -- anlamsız sıra no
    kimlik_tipi       CHAR(1)            NOT NULL CHECK (kimlik_tipi IN ('T','V')),
    segment_kodu      NUMBER(2)          NOT NULL,
    doviz_kodu        CHAR(3)            NOT NULL,
    nace_kodu         VARCHAR2(8 CHAR),                     -- yazı! sayı değil
    sektor_kodu       NUMBER(4),
    cinsiyet          CHAR(1)            CHECK (cinsiyet IN ('E','K','B')),
    kredi_risk        NUMBER(18,2),
    mevduat_bakiye    NUMBER(18,2),
    ort_faiz_orani    NUMBER(12,8),
    islem_adedi_3a    NUMBER(6),
    flag_takipte      NUMBER(1)          CHECK (flag_takipte IN (0,1)),
    -- saati sıfırlanmış tarihler
    acilis_tarihi     DATE               CHECK (acilis_tarihi = TRUNC(acilis_tarihi)),
    kapanis_tarihi    DATE               CHECK (kapanis_tarihi = TRUNC(kapanis_tarihi))
);
```

### PostgreSQL

```sql
CREATE TABLE musteri_aylik (
    donem_id          integer        NOT NULL,
    musteri_sk        integer        NOT NULL,
    kimlik_tipi       char(1)        NOT NULL CHECK (kimlik_tipi IN ('T','V')),
    segment_kodu      smallint       NOT NULL,
    doviz_kodu        char(3)        NOT NULL,
    nace_kodu         varchar(8)     CHECK (nace_kodu <> ''),   -- boş yazı yerine NULL
    sektor_kodu       smallint,
    cinsiyet          char(1)        CHECK (cinsiyet IN ('E','K','B')),
    kredi_risk        numeric(18,2),
    mevduat_bakiye    numeric(18,2),
    ort_faiz_orani    numeric(12,8),
    islem_adedi_3a    integer,
    flag_takipte      boolean,
    acilis_tarihi     date,
    kapanis_tarihi    date
);
```

### MSSQL

```sql
CREATE TABLE dbo.musteri_aylik (
    donem_id          int            NOT NULL,
    musteri_sk        int            NOT NULL,
    kimlik_tipi       char(1)        NOT NULL CHECK (kimlik_tipi IN ('T','V')),
    segment_kodu      tinyint        NOT NULL,
    doviz_kodu        char(3)        NOT NULL,
    nace_kodu         varchar(8)     COLLATE Turkish_CI_AS CHECK (nace_kodu <> ''),
    sektor_kodu       smallint,
    cinsiyet          char(1)        CHECK (cinsiyet IN ('E','K','B')),
    kredi_risk        decimal(18,2),
    mevduat_bakiye    decimal(18,2),
    ort_faiz_orani    decimal(12,8),
    islem_adedi_3a    int,
    flag_takipte      bit,
    acilis_tarihi     date,
    kapanis_tarihi    date
);

-- Tablo on milyonlarca satıra çıkacaksa, kolon bazlı saklama:
CREATE CLUSTERED COLUMNSTORE INDEX cci_musteri_aylik ON dbo.musteri_aylik;
```

---

[← Önceki: Değişken Rehberi](09-degisken-rehberi.md) · [İçindekiler](../README.md) · [Sonraki: Python ile Uygulama →](11-python-ile-uygulama.md)

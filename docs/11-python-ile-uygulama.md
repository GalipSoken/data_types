# 11. Python ile Uygulama

[← Önceki: Karar Tablosu](10-karar-tablosu.md) · [İçindekiler](../README.md) · [Sonraki: Sık Hatalar ve Kontrol Listesi →](12-sik-hatalar-ve-kontrol-listesi.md)

---

## Kısaca

Bu bölümde önceki bölümlerdeki önerilerin pandas ve Polars'ta nasıl uygulandığını görüyoruz.
Kod örnekleri kısa tutuldu; her birinin üstünde ne yaptığı yazıyor.

## 11.1 Bir tablonun ne kadar yer kapladığını ölçmek

```python
def bellek_mb(df):
    # deep=True olmadan yazı kolonlarının gerçek boyutu görünmez
    return df.memory_usage(deep=True).sum() / 1024**2

print(f"{bellek_mb(df):.1f} MB")
print(df.dtypes.value_counts())   # hangi tipten kaç kolon var?
```

`object` tipinde kolon görüyorsanız, büyük ihtimalle optimize edilecek ilk yer orasıdır.

## 11.2 Tipleri okurken vermek

En etkili yöntemdir. Tipler sonradan dönüştürülürse, okuma anında dikkatsiz tiplerle tüm tablo bir kez
belleğe yüklenmiş olur.

```python
import pandas as pd

TIPLER = {
    "musteri_sk":  "int32",
    "vkn":         "string",     # baştaki sıfır korunur
    "nace_kodu":   "category",   # yazı olarak kalır, 01.11 bozulmaz
    "sektor_kodu": "category",
    "cinsiyet":    "category",
    "doviz_kodu":  "category",
    "islem_tipi":  "category",
}
# Çok sayıda benzer kolon için:
TIPLER |= {f"flag_{i:02d}":  "Int8"    for i in range(60)}   # boş değer destekli
TIPLER |= {f"adet_{i:02d}":  "Int16"   for i in range(30)}
TIPLER |= {f"tutar_{i:02d}": "float32" for i in range(70)}   # sadece model için!

# CSV'den okurken
df = pd.read_csv("musteri.csv", usecols=list(TIPLER), dtype=TIPLER)

# Parquet'ten okurken (Parquet tipleri zaten korur; sadece gereken kolonlar)
df = pd.read_parquet("musteri.parquet", columns=list(TIPLER)).astype(TIPLER)
```

## 11.3 Hazır bir tabloyu küçültmek

Kurallara göre otomatik küçültme. **Kimlik kolonlarına ve hassas ondalıklılara dokunmaz.**

```python
import pandas as pd

KIMLIKLER    = {"tckn", "vkn", "musteri_no", "sozlesme_no", "kart_no", "hesap_no", "iban", "isin"}
HASSAS_FLOAT = {"kur", "fiyat", "faiz_orani", "getiri"}   # float32'ye düşürülmez

def kucult(df, kategori_esigi=0.5):
    sonuc = {}
    for kolon in df.columns:
        s = df[kolon]
        if kolon in KIMLIKLER:
            sonuc[kolon] = s                                      # dokunma
        elif pd.api.types.is_bool_dtype(s):
            sonuc[kolon] = s.astype("boolean" if s.isna().any() else "bool")
        elif pd.api.types.is_integer_dtype(s):
            sonuc[kolon] = pd.to_numeric(s, downcast="integer")   # en küçük uygun tam sayı
        elif pd.api.types.is_float_dtype(s):
            sonuc[kolon] = s if kolon in HASSAS_FLOAT else s.astype("float32")
        elif s.dtype == object or pd.api.types.is_string_dtype(s):
            oran = s.nunique() / max(len(s), 1)
            sonuc[kolon] = s.astype("category") if oran < kategori_esigi else s.astype("string")
        else:
            sonuc[kolon] = s
    return pd.DataFrame(sonuc)
```

> `downcast="integer"` bugünkü en büyük değere bakar. Kolon ileride büyüyebilecekse tipi elle belirleyin.

## 11.4 Veritabanından gelen `Decimal` kolonları düzeltmek

```python
from decimal import Decimal

for kolon in df.columns:
    ilk = df[kolon].dropna().head(1)
    if len(ilk) and isinstance(ilk.iloc[0], Decimal):
        df[kolon] = df[kolon].astype("float64")
```

## 11.5 Boş değerleri SQL ile aynı davranışa getirmek

```python
import numpy as np

# Boş ve sadece boşluktan oluşan yazıları NULL yap (Oracle ile aynı davranış)
for kolon in df.select_dtypes(include=["object", "string"]).columns:
    df[kolon] = df[kolon].str.strip().replace("", pd.NA)

# Gruplarken boş grubu SİLME (pandas varsayılanı siler)
# Tümü boş grubun toplamı 0 değil boş olsun (min_count=1)
rapor = df.groupby("sektor_kodu", dropna=False, observed=True)["tutar_00"].sum(min_count=1)

# Birleştirmeden önce boş anahtarları ayır (pandas boş anahtarları birbirine bağlar)
birlesik = sol.dropna(subset=["anahtar"]).merge(sag.dropna(subset=["anahtar"]), on="anahtar")

# Veritabanına yazmadan önce sonsuz değerleri temizle
df = df.replace([np.inf, -np.inf], np.nan)
```

## 11.6 Tarihleri güvenli hale getirmek

```python
# Saniye biriminde tut: 9999-12-31 gibi değerler sığar
for kolon in ["vade_tarihi", "kapanis_tarihi"]:
    tarih = pd.to_datetime(df[kolon]).astype("datetime64[s]")
    # "Bitiş yok" anlamındaki özel tarihi boşa çevir
    df[kolon] = tarih.mask(tarih.dt.year >= 9999)

# 20261025 gibi tam sayı tarihleri gerçek tarihe çevir
df["islem_tarihi"] = pd.to_datetime(df["islem_tarihi_int"].astype("string"), format="%Y%m%d")
```

## 11.7 Kuruş yuvarlama

```python
from decimal import Decimal, ROUND_HALF_UP

round(2.5)                                                   # 2  (Python: en yakın çift)
Decimal("2.675").quantize(Decimal("0.01"), ROUND_HALF_UP)    # 2.68 (yarım yukarı)
```

## 11.8 Türkçe büyük/küçük harf

```python
TR_BUYUK = str.maketrans({"i": "İ", "ı": "I"})
TR_KUCUK = str.maketrans({"I": "ı", "İ": "i"})

def tr_upper(metin):  return metin.translate(TR_BUYUK).upper()
def tr_lower(metin):  return metin.translate(TR_KUCUK).lower()

tr_upper("istanbul ışık")   # 'İSTANBUL IŞIK'
```

## 11.9 Çok büyük veride Polars

Polars, veriyi tek seferde belleğe almadan, parça parça işleyebilir:

```python
import polars as pl

sonuc = (
    pl.scan_parquet("islemler/*.parquet")              # henüz okumaz, plan kurar
      .select(["musteri_sk", "nace_kodu", "tutar", "flag_takipte"])
      .with_columns(
          pl.col("nace_kodu").cast(pl.Categorical),
          pl.col("tutar").cast(pl.Float32),
          pl.col("flag_takipte").cast(pl.Boolean),       # bit olarak saklanır
          pl.col("nace_kodu").str.slice(0, 2).alias("nace_2"),
      )
      .group_by("nace_2")
      .agg(pl.col("tutar").sum())
      .collect(engine="streaming")                       # parça parça çalıştır
)
```

---

[← Önceki: Karar Tablosu](10-karar-tablosu.md) · [İçindekiler](../README.md) · [Sonraki: Sık Hatalar ve Kontrol Listesi →](12-sik-hatalar-ve-kontrol-listesi.md)

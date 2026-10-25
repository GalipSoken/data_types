# 7. Evet/Hayır (Flag) Değişkenleri

[← Önceki: Boş Değer (NULL)](06-null.md) · [İçindekiler](../README.md) · [Sonraki: Dataiku →](08-dataiku.md)

---

## Kısaca

"Kredi kartı var mı?", "Takipte mi?", "Yapılandırılmış mı?" gibi sorulara cevap veren kolonlara
**flag** (bayrak) denir. Analiz tablolarında genelde en kalabalık gruptur; örnek tablomuzda 200 kolonun 60'ı flag.

Bu kolon aslında tek bir bit ile ifade edilebilir: 0 ya da 1. Ama pratikte çoğu flag **üç durumludur**:

| Değer | Anlamı |
|---|---|
| 1 | Evet |
| 0 | Hayır |
| boş (NULL) | Bilinmiyor / bilgi yok |

## Tipler

| | Oracle | PostgreSQL | MSSQL | Dataiku | Python |
|---|---|---|---|---|---|
| Tip | `NUMBER(1)` + yalnızca 0/1 kuralı *(23ai ve sonrası: `BOOLEAN`)* | `boolean` | `bit` | `boolean` | `bool` / `boolean` / `int8` |
| Yer | 1–2 byte | 1 byte | 8 flag = **1 byte** | — | 1 byte (Polars/Arrow'da 1 bit) |

* **Oracle'ın eski sürümlerinde gerçek evet/hayır tipi yoktur.** Bu yüzden 0/1 sayısı veya `'E'/'H'` harfi kullanılır.
* **MSSQL `bit` tipi çok verimlidir:** 8 tanesini tek byte'a sığdırır. 60 flag sadece 8 byte tutar.
* **Python'da:**
  * Boş değer yoksa `bool` veya `int8` (1 byte).
  * Boş değer anlamlıysa `boolean` (baş harfi küçük, pandas'ın boş destekli tipi) veya `Int8`.
  * Polars ve Arrow evet/hayır değerlerini **bit** olarak saklar: 60 flag × 100 milyon satır
    yalnızca 750 MB tutar. Aynı veri 8 byte'lık tam sayıyla 48 GB eder.

## Kaynak sistemlerden gelen karmaşa

Farklı sistemler aynı bilgiyi farklı yazar:

```
'E' / 'H'     'Y' / 'N'     'EVET' / 'HAYIR'     '1' / '0'     'T' / 'F'     1 / 0
```

Veriyi yüklerken hepsini **tek bir standarda** çevirin (örneğin 1 / 0 / NULL). Aksi halde biri `'E'`,
diğeri `'Y'` saydığı için aynı soru iki farklı cevap üretir.

## Modelde kullanırken

* Boş değerleri otomatik olarak 0'a çevirmeden önce düşünün: "kart sisteminde kaydı yok" bilgisi tek başına
  güçlü bir sinyal olabilir.
* Gerekirse ayrı bir "bu bilgi eksik mi?" kolonu ekleyin (`flag_kk_var_eksik`).
* MSSQL'de `bit` kolonlar doğrudan toplanamaz; önce tam sayıya çevirin.

---

[← Önceki: Boş Değer (NULL)](06-null.md) · [İçindekiler](../README.md) · [Sonraki: Dataiku →](08-dataiku.md)

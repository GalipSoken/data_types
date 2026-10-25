# 3. Sayılar: Tam Sayı, Ondalıklı Sayı ve Para

[← Önceki: Bellek ve Ölçek](02-bellek-ve-olcek.md) · [İçindekiler](../README.md) · [Sonraki: Metin →](04-metin.md)

---

## Kısaca

Sayılar üç aileye ayrılır:

| Aile | Ne için? | Örnek |
|---|---|---|
| **Tam sayı** (integer) | Küsuratı olmayan değerler | işlem adedi, gecikme günü, müşteri no |
| **Kesin ondalıklı** (decimal) | Kuruşu kuruşuna doğru olması gereken değerler | bakiye, kredi tutarı, kur |
| **Yaklaşık ondalıklı** (float) | Hızlı hesap, küçük hataların önemsiz olduğu değerler | model özellikleri, getiri, olasılık |

Bu bölümde her ailenin tiplerini, sınırlarını ve hangi durumda hangisinin seçileceğini göreceğiz.

---

## 3.1 Tam sayılar

Bir tam sayı tipi seçerken tek soru şudur: **Bu kolon en fazla kaç olabilir? Negatif olabilir mi?**

Bilgisayar tam sayıları sabit sayıda byte'ta saklar. Byte sayısı arttıkça sığabilecek en büyük sayı da büyür:

| Boyut | Sığan aralık | Kafada tutmak için | Örnek kullanım |
|---:|---|---|---|
| 1 byte | 0 … 255 (işaretsiz) veya −128 … 127 | "yüzlerce" | ürün sayısı, risk sınıfı, ay numarası |
| 2 byte | −32.768 … 32.767 | "on binler" | gecikme günü, aylık işlem adedi |
| 4 byte | −2,1 milyar … 2,1 milyar | "milyarlar" | müşteri no, şube bazında işlem sayısı |
| 8 byte | ±9,2 × 10¹⁸ | "pratikte sınırsız" | TC kimlik no, büyük sıra numaraları |

> **İşaretsiz** (unsigned): negatif değer alamayan tip. Aynı byte'ta iki kat büyük pozitif sayı sığar.
> Ama bir farkın (bugünkü bakiye − dünkü bakiye) negatif çıkabileceği yerde kullanılamaz.

### Aynı boyut, beş ortamda farklı isim

| Boyut | Oracle | PostgreSQL | MSSQL | Dataiku | Python (NumPy / pandas) |
|---|---|---|---|---|---|
| 1 byte | `NUMBER(3)` | *(yok)* | `tinyint` (0…255) | `tinyint` | `int8` / `uint8` |
| 2 byte | `NUMBER(5)` | `smallint` | `smallint` | `smallint` | `int16` |
| 4 byte | `NUMBER(10)` | `integer` | `int` | `int` | `int32` |
| 8 byte | `NUMBER(19)` | `bigint` | `bigint` | `bigint` | `int64` |

**Ortamlara özel notlar:**

* **Oracle'da gerçek bir tam sayı tipi yoktur.** Oracle her sayıyı `NUMBER` olarak saklar ve sayının
  büyüklüğüne göre otomatik olarak az ya da çok yer kullanır: `7` → 2 byte, `12345678901` → 7 byte.
  `NUMBER(5)` yazmak "en fazla 5 hane olsun" demektir; yer kaplamayı değiştirmez, sadece **kuraldır**.
  Oracle'da `INTEGER` veya `SMALLINT` yazsanız bile arka planda bu yine `NUMBER`'dır.
* **PostgreSQL'de 1 byte'lık tam sayı yoktur**; en küçüğü `smallint` (2 byte).
* **MSSQL'in `tinyint`'i sadece 0–255 alır**, negatif sayı alamaz. "Bilinmiyor" için −1 kullanmak
  isterseniz `tinyint` olmaz.
* **Python'da sınırı aşan sayı uyarı vermeden bozulur.** 1 byte'lık işaretsiz tipte `200 + 100`
  sonucu `300` değil `44`'tür. Bu yüzden tipi **bugünkü en büyük değere göre değil**, iş kuralının
  izin verdiği en büyük değere göre seçin. Bugün en fazla 200 olan işlem adedi, kampanya ayında 300 olabilir.

### Boş değer içeren tam sayılar (Python'a özel tuzak)

Klasik Python (NumPy) tam sayılarında "boş" (NULL) kavramı yoktur. Bu yüzden pandas, içinde tek bir
boş değer olan tam sayı kolonunu **sessizce ondalıklı sayıya** çevirir:

```
müşteri_no:  12345678901, NULL   →   12345678901.0, NaN
```

Sonuçları:
* Kimlik numaraları `12345678901.0` gibi görünür; yazıya çevirince sonuna `.0` eklenir ve eşleşmeler bozulur.
* 2 byte'lık bir adet kolonu 8 byte yer kaplar.

**Çözüm:** pandas'ın boş değer destekleyen tipleri: `Int8`, `Int16`, `Int32`, `Int64`
(baş harfi **büyük** yazılır). Bunlar her değer için sadece 1 byte fazladan "boş mu?" bilgisi tutar.

---

## 3.2 Kesin ondalıklı sayılar (decimal)

Para, kur ve faiz gibi değerlerde **her kuruş doğru olmalıdır**. Bunun için kesin ondalıklı tipler kullanılır.

Bu tipler iki parametre alır: `(p, s)`
* **p (precision, toplam hane):** Sayının toplam kaç basamağı olabilir?
* **s (scale, ondalık hane):** Bu basamakların kaçı virgülden sonra?

> Örnek: `(18, 2)` → virgülden önce 16, sonra 2 hane. En büyük değer 9.999.999.999.999.999,99.
> Bu, en büyük şirketlerin TL bakiyeleri için bile yeterlidir.

| | Oracle | PostgreSQL | MSSQL | Dataiku | Python |
|---|---|---|---|---|---|
| Tip | `NUMBER(18,2)` | `numeric(18,2)` | `decimal(18,2)` | *(yok — `double` olarak gelir)* | `Decimal` (yavaş) |
| Yer | değere göre 1–22 byte | değere göre değişir | p ≤ 9: 5 B · p ≤ 19: 9 B · p ≤ 28: 13 B · p ≤ 38: 17 B | — | ~100 B |

### Dikkat 1: Fazla ondalık hata vermez, sessizce yuvarlanır

`(18,2)` tanımlı bir kolona `32,45678` yazarsanız üç veritabanı da hata vermez; değeri `32,46` olarak kaydeder.
Kur gibi 4–6 ondalık gerektiren bir alanı yanlışlıkla 2 ondalıkla tanımlarsanız, **hiçbir hata almadan**
hassasiyet kaybedersiniz. Virgülden **önceki** kısım taşarsa ise üçü de hata verir.

### Dikkat 2: Python'a geldiğinde `Decimal` nesnesine dönüşebilir

Python'un veritabanı bağlantı kütüphaneleri bu kolonları bazı ayarlarda `Decimal` nesnesi olarak getirir.
pandas bunları `object` tipinde tutar: değer başına ~100 byte ve toplama gibi işlemler **100 kat yavaş**.
Okur okumaz sayı tipine çevirin (Bölüm [11](11-python-ile-uygulama.md)).

### Finansal büyüklükler için hazır ölçüler

| Büyüklük | Önerilen | Neden? |
|---|---|---|
| Tutar, bakiye, limit | `(18,2)` | Kuruş hassasiyeti |
| Nominal / büyük tutar | `(20,2)` | Çok büyük işlem tutarları |
| Kur | `(18,6)` – `(18,8)` | Çapraz kurlar 6+ ondalık ister |
| Hisse/tahvil fiyatı | `(18,6)` | Küçük adım büyüklükleri |
| Faiz oranı | `(12,8)` | Bkz. aşağıdaki not |
| Baz puan (bps) | `(9,4)` | 1 bps = %0,01 |

> **Faiz oranında birimi yazın.** Aynı faiz `0,425`, `42,5` veya `4250` (baz puan) olarak saklanabilir.
> Bu bilgi kolon adında (`faiz_orani_yuzde`) veya veri sözlüğünde yoksa, birisi mutlaka 100 kat hatalı hesap yapar.

---

## 3.3 Yaklaşık ondalıklı sayılar (float)

Float tipleri sayıyı bilimsel gösterime benzer bir biçimde (örneğin 1,2345 × 10⁷) saklar.
Hızlıdırlar ve az yer kaplarlar ama **her ondalık sayıyı tam olarak saklayamazlar**:

```
0.1 + 0.2  →  0.30000000000000004
```

İki boyutu vardır:

| Tip | Byte | Güvenilir hane sayısı | Oracle | PostgreSQL | MSSQL | Dataiku | Python |
|---|---:|---:|---|---|---|---|---|
| Tek hassasiyet | 4 | ~7 | `BINARY_FLOAT` | `real` | `real` | `float` | `float32` |
| Çift hassasiyet | 8 | ~15–16 | `BINARY_DOUBLE` | `double precision` | `float` | `double` | `float64` |

"~7 hane" ne demek? Sayının **toplam** 7 basamağı güvenilirdir, virgülün yeri fark etmez:

```
float32(1.234.567,89)   →  1.234.567,9     kuruş kayboldu
float32(12.345.678,91)  →  12.345.679      lira bile kayboldu
```

### float32 ne zaman kullanılabilir?

| Kullanım | float32 olur mu? | Neden? |
|---|---|---|
| Makine öğrenmesi özelliği (tutar, oran, log tutar) | ✔ Evet | Model için 7 hane fazlasıyla yeterli; bellek yarıya iner |
| Muhasebe, mutabakat, bakiye toplamı | ✘ Hayır | Kuruş kaybolur, toplamda hata birikir |
| Kur, fiyat (saklama) | ✘ Hayır | Büyük tam kısım + çok ondalık 7 haneyi aşar |
| Getiri, birikimli getiri | ✘ Hayır | Çarpım zincirinde küçük hatalar büyür |
| Risk ölçüleri (volatilite, VaR) | Genelde hayır | Küçük farkların farkı alınır |

### İsim karışıklıkları

* **Oracle'daki `FLOAT` aslında float değildir**; `NUMBER`'ın bir türüdür. Gerçek float için
  `BINARY_FLOAT` / `BINARY_DOUBLE` kullanılır.
* **MSSQL'de `float` 8 byte'tır** (Python'daki `float64` ile aynı), `real` 4 byte'tır.
  `float(10)` yazmak 10 hanelik hassasiyet **vermez**.
* **Dataiku'da `float` 4 byte, `double` 8 byte'tır.**

### Sonsuz ve "sayı değil" (NaN) değerleri

Bir oranı hesaplarken paydanın sıfır olması finansta sık görülür (örneğin özkaynağı sıfır olan firmanın
kaldıraç oranı). Python bu durumda `inf` (sonsuz) veya `NaN` (sayı değil) üretir. Veritabanları bu
değerlere farklı davranır:

| | Oracle | PostgreSQL | MSSQL |
|---|---|---|---|
| `NaN` / `inf` saklanabilir mi? | Sadece `BINARY_*` tiplerinde | Evet | **Hayır — yazarken hata verir** |

PostgreSQL `NaN`'ı kabul eder ama onu tüm sayılardan **büyük** sayar; "en büyük değer" sorgusu `NaN` döndürür.
**Kural:** Veritabanına yazmadan önce Python'da `inf` ve `NaN` değerlerini boş (NULL) değere çevirin.

---

## 3.4 Para tipleri: kullanmayın

| | Tip | Sorun |
|---|---|---|
| MSSQL | `money`, `smallmoney` | Sabit 4 ondalık; çarpma/bölmede yuvarlama hataları; `smallmoney` en fazla 214.748 |
| PostgreSQL | `money` | Gösterim sunucunun dil ayarına bağlı; döviz cinsini bilmez |

Bunun yerine: kesin ondalıklı tip + yanında **döviz kodu** kolonu (`TRY`, `USD`, `EUR`).

---

## 3.5 Aynı hesap, farklı sonuç

Aynı SQL sorgusu üç veritabanında farklı sonuç verebilir:

| İşlem | Oracle | PostgreSQL | MSSQL | Python |
|---|---|---|---|---|
| `5 / 2` | 2,5 | **2** | **2** | 2,5 |
| Tam sayı kolonun ortalaması | 12,7 | 12,7 | **12** (küsurat atılır) | 12,7 |
| Tam sayı kolonun toplamı 2,1 milyarı aşarsa | sorun yok | sorun yok | **hata** | sorun yok |
| `2,5`'i yuvarla | 3 | 3 | 3 | **2** |

Açıklamalar:
* **Tam sayı bölmesi:** PostgreSQL ve MSSQL'de iki tam sayıyı bölerseniz sonuç da tam sayıdır ve küsurat atılır.
  "Ödenen taksit / toplam taksit" oranı hep **0** çıkar. Çözüm: birini ondalıklı tipe çevirin
  (`odenen * 1.0 / toplam`).
* **MSSQL'de ortalama ve toplam:** Tam sayı kolonun ortalaması da tam sayı döner; toplamı da 4 byte'lık tam
  sayıdır ve taşabilir. Çözüm: hesaplamadan önce büyük tipe çevirin.
* **Python'un yuvarlaması farklıdır:** Python yarımları en yakın **çift** sayıya yuvarlar
  (2,5 → 2; 3,5 → 4). Buna "bankacı yuvarlaması" denir. Sözleşme veya mevzuat "yarım yukarı" diyorsa
  Python'da bunu açıkça belirtmeniz gerekir. Aksi halde SQL ile Python arasında kuruş farkları çıkar.

---

## Özet: hangi sayıya hangi tip?

| Değişken | Veritabanında | Python'da (analiz/model) |
|---|---|---|
| Tutar, bakiye | kesin ondalık `(18,2)` + döviz kodu | `float32` (model), `float64` (toplam) |
| Kur, fiyat | kesin ondalık `(18,6)` | `float64` |
| Faiz oranı | kesin ondalık `(12,8)`, birimi belli | `float64` |
| Getiri | 8 byte float | `float64` |
| Olasılık, skor | 8 byte float veya `(9,8)` | `float32` |
| Küçük adet (0–255) | Oracle `NUMBER(3)`, PG `smallint`, MSSQL `tinyint` | `uint8` / `Int16` |
| Adet (0–32.767) | `NUMBER(5)` / `smallint` | `int16` / `Int16` |
| Büyük adet, hacim | `NUMBER(15)` / `bigint` | `int64` |

---

[← Önceki: Bellek ve Ölçek](02-bellek-ve-olcek.md) · [İçindekiler](../README.md) · [Sonraki: Metin →](04-metin.md)

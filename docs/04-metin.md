# 4. Metin (Yazı) Tipleri

[← Önceki: Sayılar](03-sayilar.md) · [İçindekiler](../README.md) · [Sonraki: Tarih ve Zaman →](05-tarih-ve-zaman.md)

---

## Kısaca

Yazı tutan kolonlar üç gruba ayrılır ve her grup farklı ele alınır:

| Grup | Örnek | Nasıl saklanır? |
|---|---|---|
| **Kod** — az sayıda farklı değer, çok tekrar | cinsiyet (E/K), işlem tipi (EFT, FAST…), döviz kodu, sektör kodu | Kısa yazı veya kod tablosuna bağlı küçük sayı; Python'da **kategori** |
| **Kimlik** — sayıya benzeyen ama sayı olmayan | vergi no, IBAN, kart no, NACE kodu | Sabit uzunlukta yazı |
| **Serbest metin** — neredeyse her satır farklı | müşteri adı, firma unvanı, açıklama | Değişken uzunlukta yazı |

## Tipler

| | Oracle | PostgreSQL | MSSQL | Dataiku | Python |
|---|---|---|---|---|---|
| Değişken uzunluk | `VARCHAR2(n CHAR)` | `text` veya `varchar(n)` | `varchar(n)` / `nvarchar(n)` | `string` | `string` |
| Sabit uzunluk | `CHAR(n)` | `char(n)` *(önerilmez)* | `char(n)` | `string` | — |
| Çok tekrar eden kod | kod tablosu + sayı | kod tablosu + sayı | kod tablosu + sayı | `string` | `category` |

* **Değişken uzunluk:** "En fazla n karakter" demektir; kısa değer kısa yer kaplar.
* **Sabit uzunluk:** Her değer tam n karakterdir, kısa olanların sonu boşlukla doldurulur.
  Hep aynı uzunlukta olan değerler (IBAN 26, cinsiyet 1, döviz kodu 3 karakter) için uygundur.

## Türkçe karakter tuzakları

Türkçe harfler (`ş, ğ, ı, İ, ü, ö, ç`) bilgisayarda çoğunlukla **2 byte** yer kaplar; İngilizce harfler 1 byte.
Bu, beklenmedik hatalara yol açar.

### Oracle: "50 karakter" mi, "50 byte" mı?

Oracle'da `VARCHAR2(50)` yazarsanız varsayılan anlamı **50 byte**'tır, 50 karakter değil.

```
"MEHMET YILMAZ"        → 13 karakter, 13 byte  → sığar
"ŞÜKRÜ GÜNEŞ ÖZTÜRK"   → 18 karakter, 25 byte  → (20) tanımlı kolona sığmaz!
```

**Çözüm:** Her zaman `VARCHAR2(50 CHAR)` yazın. `CHAR` kelimesi "50 karakter" demektir.

### MSSQL: `varchar` mı, `nvarchar` mı?

* `nvarchar` her karakter için **2 byte** kullanır, tüm dilleri destekler.
* `varchar` doğru **collation** (dil/karşılaştırma kuralı) ile Türkçe'yi de destekler ve yarı yer kaplar:
  `Turkish_CI_AS` ya da yeni sürümlerdeki UTF-8 seçenekleri.
* Farklı collation'a sahip iki kolonu birleştirmeye (join) çalışırsanız hata alırsınız veya sorgu yavaşlar.
  Bir veritabanında tek bir kuralda karar kılın.

### Python: büyük harf sorunu

```python
"istanbul".upper()   →  "ISTANBUL"     (Türkçe'de doğrusu "İSTANBUL")
"IŞIK".lower()       →  "işik"         (doğrusu "ışık")
```

İsim veya firma eşleştirirken bu fark eşleşmeleri kaçırır. Türkçe'ye özel dönüşüm yapın
(örneği [Bölüm 11](11-python-ile-uygulama.md)'de).

## Kategori tipi: ne zaman kullanılır?

Python'daki `category` tipi, her farklı değeri **bir kez** saklar ve satırlarda sadece küçük bir numara tutar.

```
Normal:    "KOBİ", "KOBİ", "TİCARİ", "KOBİ", ...     (her satır kendi yazısını taşır)
Kategori:  sözlük = {0: "KOBİ", 1: "TİCARİ"}
           satırlar = 0, 0, 1, 0, ...                 (her satır 1 byte)
```

* **Kullanın:** Farklı değer sayısı az, tekrar çok (cinsiyet, segment, il, işlem tipi, NACE).
* **Kullanmayın:** Neredeyse her satır farklıysa (müşteri adı). Bu durumda sözlük tablonun kendisi kadar büyür
  ve kategori normal yazıdan **daha çok** yer kaplar.
* **Kaba kural:** Farklı değer sayısı / satır sayısı **%50'nin altındaysa** kategori düşünün.

## Kod ve açıklamayı ayırın

`sektor_kodu` ve `sektor_adi` aynı bilgiyi taşır. Açıklamayı 100 milyon satırda tekrar etmek yerine
ayrı bir **kod tablosunda** bir kez saklayın:

```
Ana tablo:   musteri_no | sektor_kodu         Kod tablosu:  sektor_kodu | sektor_adi
             1001       | 12                                12          | Tekstil Ürünleri İmalatı
             1002       | 12                                45          | Toptan Ticaret
```

> 100 milyon satırda ortalama 30 karakterlik bir açıklama ≈ **3 GB**; 2 byte'lık bir kod ≈ **200 MB**.

---

[← Önceki: Sayılar](03-sayilar.md) · [İçindekiler](../README.md) · [Sonraki: Tarih ve Zaman →](05-tarih-ve-zaman.md)

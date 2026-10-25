# 6. Boş Değer (NULL)

[← Önceki: Tarih ve Zaman](05-tarih-ve-zaman.md) · [İçindekiler](../README.md) · [Sonraki: Evet/Hayır Değişkenleri →](07-evet-hayir.md)

---

## Kısaca

**NULL**, "burada bir değer yok" demektir. Sıfır değildir, boş yazı (`''`) değildir, "hayır" değildir.

Basit görünür ama iki nedenle finansta en çok yanlış rakam üreten konulardan biridir:

1. **NULL'un iş anlamı belirsizdir.** Boş bir bakiye "hesap yok" mu, "bakiye sıfır" mı, "veri gelmedi" mi?
2. **Her ortam NULL'a biraz farklı davranır.** Aynı sorgu Oracle'da ve MSSQL'de farklı liste verebilir.

## 6.1 Önce iş anlamı: boş ne demek?

Bir kolonun boş olması farklı şeyler anlatabilir. Her birinin doğru ele alınışı farklıdır:

| Kolon | Boş olması ne demek olabilir? | Yanlış | Doğru |
|---|---|---|---|
| Bakiye | Hesap yok / henüz hesaplanmadı / veri gelmedi | Hepsini 0 yapmak | "Hesap yok" → 0; "veri gelmedi" → boş bırak ve kontrol et |
| Fiyat, kur | O gün işlem görmedi, tatil | 0 yapmak (getiri %−100 çıkar!) | Son geçerli fiyatı kullan (sınırlı gün sayısı kadar, işaretleyerek) |
| Getiri | İlk gün (önceki fiyat yok) | 0 yapmak | Boş bırak |
| Faiz oranı | Değişken faiz, henüz belli değil | 0 yapmak | Boş bırak + faiz tipi kolonu |
| Vade | Vadesiz ürün (vadesiz mevduat, kart) | `1900-01-01` | Boş veya tek bir özel değer ([Bölüm 5.7](05-tarih-ve-zaman.md#57-özel-anlamlı-tarihler)) |
| Evet/hayır | Bilinmiyor | 0 (hayır) yapmak | Boş bırak; modelde "eksik mi?" diye ayrı bir kolon ekle |
| Cinsiyet | Tüzel kişi (uygulanamaz) / bilinmiyor | İkisini aynı kodla tutmak | İkisini ayrı kodla |
| İşlem hacmi | O gün işlem olmadı | Boş bırakmak | 0 (burada boş değil sıfır doğrudur) |

> **Kural:** Her kolon için "boş ne demek?" sorusunun cevabını veri sözlüğüne yazın.
> Toptan `fillna(0)` (tüm boşları sıfırla doldurmak) en sık yapılan hatadır.

## 6.2 En büyük fark: Oracle'da boş yazı = NULL

Oracle'da **boş yazı (`''`) otomatik olarak NULL'a dönüşür.** PostgreSQL ve MSSQL'de ise ikisi farklı şeylerdir.

| | Oracle | PostgreSQL | MSSQL |
|---|---|---|---|
| `''` nedir? | **NULL** | boş yazı | boş yazı |
| `WHERE ad = ''` | hiçbir şey bulmaz | boş adları bulur | boş adları bulur |
| Zorunlu (NOT NULL) kolona `''` yazmak | **hata** | olur | olur |
| `'A'` ile NULL'u birleştirmek | `'A'` | NULL | NULL |

**Ne zaman sorun çıkar?**
* Oracle'dan PostgreSQL/MSSQL'e taşınan veride boş yazılar artık NULL olarak gelir.
* Tersi yönde, PostgreSQL/MSSQL'deki boş yazılar Oracle'a gidince NULL olur ve zorunlu kolon kuralları hata verir.
* PostgreSQL/MSSQL'de "boş olanları bul" demek için iki koşul gerekir: `ad IS NULL OR ad = ''`.

**Çözüm:** Veriyi yüklerken boş ve sadece boşluktan oluşan yazıları **NULL'a çevirin**. Böylece tüm
ortamlarda aynı davranırsınız.

## 6.3 NULL ile karşılaştırma ve hesaplama

Bu kurallar üç veritabanında **aynıdır**, ama Python'dan farklıdır.

**NULL hiçbir şeye eşit değildir, kendisine bile.** `NULL = NULL` sonucu "doğru" değil "bilinmiyor"dur.
Bu yüzden NULL'ları bulmak için `= NULL` değil `IS NULL` yazılır.

### Tehlikeli sorgu: `NOT IN`

```sql
-- Kara listede OLMAYAN müşteriler:
SELECT * FROM musteri
WHERE musteri_no NOT IN (SELECT musteri_no FROM kara_liste);
```

Kara listede **tek bir** NULL müşteri no varsa bu sorgu **hiç satır döndürmez.** Hata da vermez.
Doğrusu `NOT EXISTS` kullanmaktır.

### Toplama ve ortalama

| İşlem | Veritabanları | pandas | Not |
|---|---|---|---|
| Toplam (bazı değerler boş) | Boşları atlar | Boşları atlar | Aynı |
| Toplam (**tüm** değerler boş) | **NULL** | **0** | pandas'ta "veri yok" ile "sıfır" karışır |
| Ortalama | Boşları paydaya katmaz | Aynı | Aşağıdaki örneğe bakın |
| Satır sayma | `COUNT(*)` tümü, `COUNT(kolon)` dolu olanlar | `len()` / `.count()` | |

**Ortalama örneği:** 4 hesabın bakiyesi `100, 200, NULL, NULL` olsun.
* `AVG(bakiye)` = (100 + 200) / **2** = 150 → "bakiyesi bilinen hesapların ortalaması"
* `AVG(COALESCE(bakiye, 0))` = 300 / **4** = 75 → "boş = 0 kabul edilirse ortalama"

İkisi de doğru olabilir; hangisinin sorulduğuna NULL'un iş anlamı karar verir.

### Gruplama ve birleştirme (join)

| | Veritabanları | pandas | Polars |
|---|---|---|---|
| Gruplamada boş grup | Boşlar **tek bir grup** olur | Boş grup **silinir** | Tek grup olur |
| Birleştirmede boş anahtarlar | **Eşleşmez** | **Birbiriyle eşleşir** | Eşleşmez |

Bu iki pandas davranışı test edilerek doğrulanmıştır ve çok tehlikelidir:
* "Sektöre göre toplam risk" raporunda **sektörü boş olan müşterilerin riski raporda görünmez.**
* SQL'de yazılmış bir birleştirmeyi Python'da tekrarlarsanız, boş anahtarlar birbirine bağlanır ve
  **fazladan satırlar** oluşur.

Çözümler [Bölüm 11](11-python-ile-uygulama.md)'de.

## 6.4 Sıralama: NULL'lar başa mı, sona mı?

| | Oracle | PostgreSQL | MSSQL |
|---|---|---|---|
| Küçükten büyüğe | NULL'lar **sonda** | NULL'lar **sonda** | NULL'lar **başta** |
| Büyükten küçüğe | NULL'lar **başta** | NULL'lar **başta** | NULL'lar **sonda** |
| Yeri elle seçmek (`NULLS LAST`) | Var | Var | **Yok** |

**Neden önemli?** "En riskli 100 müşteri" sorgusu (riske göre büyükten küçüğe sırala, ilk 100'ü al):
* Oracle ve PostgreSQL'de listenin başına **riski boş olan müşteriler** gelir.
* MSSQL'de gelmez.

Aynı sorgu iki ortamda iki farklı liste verir. Çözüm: sıralamada NULL'ların yerini her zaman açıkça belirtin.

## 6.5 Benzersizlik kuralı (UNIQUE) ve NULL

"Vergi numarası benzersiz olsun ama boş olabilsin" gibi bir kural koyduğunuzda:

| | Oracle | PostgreSQL | MSSQL |
|---|---|---|---|
| Birden fazla boş değer olabilir mi? | Evet | Evet | **Hayır, sadece bir tane** |

MSSQL'de bu kural için özel bir indeks gerekir ("sadece dolu olanlar benzersiz olsun"):

```sql
CREATE UNIQUE INDEX ux_vkn ON musteri(vkn) WHERE vkn IS NOT NULL;
```

Oracle'a özel bir not: Oracle'ın tek kolonlu indeksleri NULL değerleri saklamaz. Bu yüzden
`WHERE kapanis_tarihi IS NULL` (açık hesaplar) sorgusu indeksi kullanamaz ve tüm tabloyu tarar.

## 6.6 NULL ile çalışan fonksiyonlar

| Amaç | Oracle | PostgreSQL | MSSQL |
|---|---|---|---|
| Boşsa yerine şunu koy | `NVL`, `COALESCE` | `COALESCE` | `ISNULL`, `COALESCE` |
| Şu değerse boş yap | `NULLIF` | `NULLIF` | `NULLIF` |

İki pratik ipucu:
* **Sıfıra bölmeyi önlemek için:** `pay / NULLIF(payda, 0)` → payda sıfırsa sonuç hata değil NULL olur.
* **MSSQL'in `ISNULL`'una dikkat:** Sonucu ilk kolonun uzunluğuna keser.
  2 karakterlik bir kolon için `ISNULL(kod, 'BILINMIYOR')` sonucu `'BI'` olur. Bunun yerine `COALESCE` kullanın.

## 6.7 Python'da boşluğun dört hali

| Değer | Ne zaman görülür? |
|---|---|
| `NaN` | Ondalıklı sayı kolonlarında. "Eksik" ile "0/0 sonucu" aynı görünür, ayırt edilemez. |
| `None` | Python nesnesi tutan kolonlarda |
| `pd.NA` | pandas'ın boş destekli yeni tiplerinde (`Int64`, `boolean`, `string`) |
| `NaT` | Tarih kolonlarında |

Finansta "paydası sıfır olduğu için hesaplanamadı" ile "veri yok" farklı bilgilerdir. Bu ayrım önemliyse,
oranı hesaplamadan önce "payda sıfır mı?" bilgisini ayrı bir kolona yazın.

---

[← Önceki: Tarih ve Zaman](05-tarih-ve-zaman.md) · [İçindekiler](../README.md) · [Sonraki: Evet/Hayır Değişkenleri →](07-evet-hayir.md)

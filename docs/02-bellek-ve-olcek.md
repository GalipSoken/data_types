# 2. Bellek (RAM) ve Ölçek: 100 Bin'den 100 Milyon Satıra

[← Önceki: Veri Tipi Nedir?](01-veri-tipi-nedir.md) · [İçindekiler](../README.md) · [Sonraki: Sayılar →](03-sayilar.md)

---

## Kısaca

Python'da bir tabloyu (DataFrame) açtığınızda, tablonun tamamı bilgisayarın **RAM**'ine (çalışma belleği)
yüklenir. RAM, disk alanından çok daha azdır: tipik bir dizüstü bilgisayarda 16 GB, paylaşımlı bir
Dataiku sunucusunda kullanıcı başına belki 16–64 GB.

Tablo RAM'e sığmazsa iş yarıda kesilir (**OOM — Out Of Memory, bellek yetersiz** hatası).
Doğru veri tipleri, aynı tablonun RAM'de **6 kat daha az** yer kaplamasını sağlayabilir.

## Üç farklı yaklaşımı karşılaştırdık

200 kolonlu örnek tabloyu (bkz. [Bölüm 1](01-veri-tipi-nedir.md#bu-rehberde-kullanılan-örnek-tablo))
100.000 satırla oluşturup gerçekten ölçtük. Üç durum var:

| Yaklaşım | Ne demek? |
|---|---|
| **Dikkatsiz** | Veri nasıl geldiyse öyle bırakılmış. Yazılar Python nesnesi (`object`), tüm sayılar en büyük tipte (64 bit). pandas 2 ve öncesinde varsayılan budur. |
| **pandas 3 varsayılanı** | pandas'ın yeni sürümü yazıları otomatik olarak daha verimli saklıyor; sayılar hâlâ en büyük tipte. |
| **Optimize** | Her kolona bu rehberde önerilen, ihtiyacı kadar büyük tip verilmiş. |

## Satır başına ne kadar yer kaplıyor?

| Kolon grubu | Dikkatsiz | pandas 3 varsayılanı | Optimize |
|---|---:|---:|---:|
| Kimlik (6) | 311 B | 106 B | 60 B |
| İsim (2) | 242 B | 60 B | 60 B |
| Kategori (27) | **1.629 B** | 419 B | **29 B** |
| Evet/hayır (60) | 480 B | 480 B | 60 B |
| Tutar (70) | 560 B | 560 B | 280 B |
| Adet (30) | 240 B | 240 B | 60 B |
| Tarih (5) | 295 B | 90 B | 40 B |
| **Toplam (bir satır)** | **≈ 3.757 B** | **≈ 1.956 B** | **≈ 589 B** |

**Buradan çıkan iki ders:**

1. **En büyük israf tekrar eden yazılarda.** "İSTANBUL", "KOBİ", "EFT" gibi değerler milyonlarca satırda
   tekrar ediyor. Python her birini ayrı ayrı saklarsa kategori kolonları tek başına toplam belleğin
   neredeyse yarısını yer. Bu kolonları **kategori tipine** (`category`) çevirmek, onları **56 kat** küçültüyor.
   Kategori tipi her farklı değeri bir kez saklar, satırlarda sadece küçük bir numara tutar.
   Tıpkı bir sözlükte kelimeyi bir kez yazıp metinde sayfa numarasıyla göndermek gibi.
2. **İkinci büyük kalem tutarlar.** 70 tutar kolonunu 8 byte'lık sayı yerine 4 byte'lık sayıda tutmak
   yeri yarıya indiriyor. Ama bu her zaman güvenli değil; ayrıntısı [Sayılar](03-sayilar.md) bölümünde.

## Satır sayısı arttıkça toplam bellek

| Satır sayısı | Dikkatsiz | pandas 3 varsayılanı | Optimize | En düşük (model için)* |
|---:|---:|---:|---:|---:|
| 100 bin | 358 MB | 187 MB | 56 MB | ~40 MB |
| 1 milyon | 3,5 GB | 1,8 GB | 0,55 GB | ~0,4 GB |
| 10 milyon | 35 GB | 18 GB | 5,5 GB | ~3,8 GB |
| 100 milyon | 350 GB | 182 GB | 55 GB | ~38 GB |

\* İsim kolonları çıkarılmış, kimlikler küçük sıra numaralarına çevrilmiş, evet/hayır kolonları bit
olarak saklanmış hali.

### "Tepe bellek" nedir?

Tablonun RAM'deki boyutu, işlem sırasında ihtiyaç duyulan belleğin **tamamı değildir**. Dosyayı okurken,
iki tabloyu birleştirirken (`merge`) ya da bir kopya oluştururken, Python kısa süreliğine tablonun
**bir kopyasını daha** tutar. Bu yüzden pratikte tablo boyutunun **2–3 katı** RAM gerekir.

> Örnek: 1 milyon satırlık dikkatsiz tablo 3,5 GB'tır, ama okunurken 7–10 GB RAM ister.
> 16 GB'lık paylaşımlı bir sunucuda iki kişi bunu aynı anda yaparsa sunucu çöker.

## Diskte ne kadar yer kaplıyor?

| Dosya formatı | 100 bin | 1 milyon | 10 milyon | 100 milyon |
|---|---:|---:|---:|---:|
| CSV | 108 MB | ~1,1 GB | ~11 GB | ~106 GB |
| Parquet | 45–59 MB | ~0,5 GB | ~5 GB | ~45–58 GB |

**Parquet** nedir? Veriyi kolon kolon ve sıkıştırarak saklayan bir dosya formatıdır. CSV'ye göre
avantajları:

* Daha küçüktür (gerçek veride genelde 3–8 kat).
* **Tip bilgisini korur.** CSV'de her şey yazıdır; tekrar okurken tarih, sayı, baştaki sıfır gibi bilgiler
  yeniden tahmin edilir ve sık sık yanlış tahmin edilir.
* Boş değer (NULL) ile boş yazıyı (`""`) birbirinden ayırır. CSV ayıramaz.
* Sadece istediğiniz kolonları okuyabilirsiniz; 200 kolondan 10'una ihtiyacınız varsa 190'ı diskte kalır.

> **Kural:** Ara çıktılar için Parquet, başka bir kişiye/sisteme göndermek için CSV.

## Veritabanında ne kadar yer kaplıyor?

Aynı 200 kolonlu tablo, önerilen tiplerle veritabanında yaklaşık şu kadar yer kaplar
(bunlar tiplerden hesaplanmış tahminlerdir):

| Veritabanı | Bir satır | 100 milyon satır |
|---|---:|---:|
| Oracle | ~850–900 B | ~80–85 GB |
| PostgreSQL | ~950–1.050 B | ~90–100 GB |
| MSSQL | ~850–900 B | ~80–85 GB |
| Kolon bazlı saklama (columnstore, Parquet) | — | ~10–20 GB |

Klasik veritabanları veriyi **satır satır** saklar; bu, tek bir müşteriyi bulup güncellemek için idealdir.
Analizde ise genelde "tüm müşterilerin bakiyesini topla" gibi **birkaç kolonu çok sayıda satırda** okuruz.
Bunun için veriyi **kolon kolon** saklayan yapılar (MSSQL columnstore, Oracle HCC, Parquet) 5–10 kat
daha az yer kaplar ve çok daha hızlıdır.

## Ölçeğe göre ne yapmalı?

### 100 bin satır: "Her şey çalışır, ama doğru alışkanlık edinin"
* Dikkatsiz haliyle bile ~360 MB. Her bilgisayarda rahat çalışır.
* Tip seçimi burada **hız için değil, doğruluk için** önemli: baştaki sıfırlar, bozulan kodlar,
  kayan tarihler bu ölçekte de projeyi bozar.
* Tipleri burada doğru kurarsanız veri büyüdüğünde baştan yazmanız gerekmez.

### 1 milyon satır: "Tip seçimi artık zorunlu"
* Dikkatsiz haliyle okurken 7–10 GB RAM ister; paylaşımlı sunucularda sorun çıkar.
* Optimize haliyle ~550 MB; rahat.
* Yapılacaklar:
  * Sadece ihtiyacınız olan kolonları okuyun.
  * Tipleri **okurken** verin. Okuduktan sonra dönüştürmek, tepe bellek sorununu çözmez.
  * Ara sonuçları Parquet'e yazın.

### 10 milyon satır: "Hesabı verinin olduğu yerde yapın"
* Dikkatsiz haliyle 35 GB; tek bir Python işlemine sığmaz.
* Yapılacaklar:
  * Toplama, gruplama gibi işleri **veritabanında** yapın (SQL ya da Dataiku'nun veritabanında çalışan
    recipe'leri). 200 kolonun çoğu zaten "müşteri bazında topla" işlemleridir; bunu veritabanı çok daha
    iyi yapar.
  * Python'a sadece son tabloyu çekin. Gerekirse parça parça okuyun.
  * Model eğitmek için tüm veriye ihtiyacınız olmayabilir: hedef oranı korunarak alınmış 1–2 milyonluk
    bir **örneklem** çoğu zaman aynı başarıyı verir.

### 100 milyon satır: "RAM'e hiç almayın"
* Optimize haliyle bile 55 GB. Finansta bu ölçek çok yaygın: kart işlemleri, EFT kayıtları,
  günlük bakiye tabloları, borsa fiyat verisi.
* Yapılacaklar:
  * Veritabanında tabloyu **bölümleyin** (partition; örneğin her ay ayrı bir bölüm) ve kolon bazlı
    saklama kullanın.
  * Hesaplamaları SQL, Spark ya da parça parça çalışabilen araçlarla (Polars, DuckDB) yapın.
  * Bu ölçekte her byte önemlidir: tek bir kolonu 8 yerine 2 byte'lık tipte tutmak **600 MB** kazandırır.

---

[← Önceki: Veri Tipi Nedir?](01-veri-tipi-nedir.md) · [İçindekiler](../README.md) · [Sonraki: Sayılar →](03-sayilar.md)

# 1. Veri Tipi Nedir ve Neden Önemlidir?

[← İçindekiler](../README.md) · [Sonraki: Bellek ve Ölçek →](02-bellek-ve-olcek.md)

---

## Kısaca

Bir tabloda her kolonun bir **veri tipi** vardır. Veri tipi, bilgisayara iki şey söyler:

1. **Bu kolonda ne tür değerler olabilir?** (tam sayı mı, ondalıklı sayı mı, yazı mı, tarih mi?)
2. **Her değer için ne kadar yer ayrılsın?**

Bunu bir dolaba eşya yerleştirmek gibi düşünebilirsiniz. Ataç koymak için ayakkabı kutusu kullanırsanız
dolap çabucak dolar. Ayakkabıyı ataç kutusuna sığdırmaya çalışırsanız ayakkabı sığmaz ya da ezilir.
Doğru tip, **doğru boyutta kutu** seçmektir.

## Temel birimler

| Birim | Ne kadar? | Günlük karşılığı |
|---|---|---|
| **bit** | 0 ya da 1 | Bir lamba: açık/kapalı |
| **byte (B)** | 8 bit | Bir harf ya da 0–255 arası bir sayı |
| **KB** | ~1.000 byte | Yarım sayfa düz yazı |
| **MB** | ~1.000 KB | Kısa bir şarkı |
| **GB** | ~1.000 MB | Bir film |

Bir kolonda değer başına 1 byte yerine 8 byte kullanmak küçük bir fark gibi görünür. Ama:

> 100 milyon satır × 7 byte fazlalık = **700 MB** — tek bir kolon için.
> 200 kolonluk bir tabloda bu fark **onlarca GB**'a çıkar.

## Yanlış tip seçince ne olur?

Yanlış tip yalnızca yer israfı değildir; çoğu zaman **veri bozulur** ve kimse fark etmez:

| Ne yaptık? | Ne oldu? |
|---|---|
| Vergi numarası `0012345678`'i sayı olarak sakladık | Baştaki sıfırlar silindi: `12345678` |
| NACE kodu `01.11`'i sayı yaptık | `1.11` oldu, tarım sektörü kodu bozuldu |
| 16 haneli kart numarasını ondalıklı sayı (float) yaptık | Son hane değişti |
| 1.234.567,89 TL'yi küçük ondalıklı sayıda (float32) tuttuk | 1.234.567,90 oldu, kuruş kayboldu |
| Tarih kolonunu saat dilimine dikkat etmeden taşıdık | Tüm tarihler bir gün geri kaydı |

Bu rehberdeki her bölüm bu tür hataların **neden** olduğunu ve **nasıl önleneceğini** anlatır.

## Veri hangi ortamlardan geçer?

Bir finansal veri genelde birkaç ortamdan geçer ve her ortamın kendi tip sistemi vardır:

```
 Kaynak sistem          Veri ambarı / DB           Analiz platformu        Python
 (çekirdek bankacılık)  (Oracle, PostgreSQL,       (Dataiku)               (pandas, Polars)
                         MSSQL)
        │                       │                        │                      │
        └──────── tip dönüşümü ─┴──────── tip dönüşümü ──┴──── tip dönüşümü ────┘
```

Her ok bir **tip dönüşümü** demektir. Veri kayıpları çoğunlukla bu geçişlerde yaşanır.
Bu yüzden rehber her konuyu bu beş ortam için yan yana anlatır:

| Ortam | Ne işe yarar? |
|---|---|
| **Oracle** | Bankalarda çok yaygın, büyük ilişkisel veritabanı |
| **PostgreSQL** | Açık kaynak ilişkisel veritabanı |
| **MSSQL** (SQL Server) | Microsoft'un ilişkisel veritabanı |
| **Dataiku** | Veri hazırlama ve modelleme platformu; veriyi bu veritabanlarından okur |
| **Python** (pandas, NumPy, Polars, Arrow) | Analiz ve makine öğrenmesi kütüphaneleri |

## Bir kolonun tipini seçerken sorulacak 6 soru

Bu sorular rehberin geri kalanında sürekli karşımıza çıkacak:

1. **Hangi değerleri alabilir?** En küçük ve en büyük değer ne? Kaç hane? Kaç ondalık?
2. **Üzerinde hesap yapılacak mı?** Toplanıp ortalaması alınacaksa sayıdır. Sadece "kimlik" ise
   (TC kimlik no, hesap no) sayı gibi görünse de **sayı değildir**.
3. **Biçimi önemli mi?** Baştaki sıfır, nokta, tire anlam taşıyor mu?
4. **Boş (NULL) olabilir mi ve boşsa ne anlama gelir?** "Bilinmiyor" mu, "yok" mu, "sıfır" mı?
5. **Kişisel veya hassas mı?** KVKK (kişisel veriler) ve PCI-DSS (kart verisi) kuralları var mı?
6. **Modelde nasıl kullanılacak?** Bir özellik (feature) mi, yoksa sadece tabloları birleştirmeye yarayan
   bir anahtar mı?

## Bu rehberde kullanılan örnek tablo

Rehber boyunca **200 kolonlu** bir finansal analiz tablosunu örnek alıyoruz. Bu tablo, her müşteri
için bir satır ve o müşteriyi anlatan 200 özellikten oluşuyor. Veri biliminde buna
**ABT (Analytical Base Table, analitik temel tablo)** denir.

| Kolon grubu | Adet | Örnekler |
|---|---:|---|
| Kimlik / anahtar | 6 | TC kimlik no, vergi no, müşteri no, sözleşme no, hesap no, kart no |
| Serbest yazı | 2 | müşteri adı, firma adı |
| Kategori (az sayıda farklı değer) | 27 | cinsiyet, NACE kodu, sektör, işlem tipi, segment, il, döviz kodu |
| Evet/hayır (flag) | 60 | kredi kartı var mı, takipte mi, … |
| Tutar | 70 | bakiye, limit, risk, ciro, … |
| Adet | 30 | işlem sayısı, ürün sayısı, gecikme günü, … |
| Tarih | 5 | hesap açılış, son işlem, vade, … |

---

[← İçindekiler](../README.md) · [Sonraki: Bellek ve Ölçek →](02-bellek-ve-olcek.md)

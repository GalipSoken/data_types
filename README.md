# Finansal Veride Veri Tipleri Rehberi

**Oracle · PostgreSQL · MSSQL · Dataiku · Python (pandas / Polars)**

Bir kolonu sayı mı, yazı mı, tarih mi olarak saklamalıyız? Kaç byte'lık tip yeter? Boş değer ne anlama gelir?
Bu rehber, finansal veriyle (müşteri, hesap, kredi, işlem, piyasa verisi) çalışan ve bu konulara
**ilk kez** detaylı bakan veri bilimciler için hazırlandı.

Her bölüm kısa bir özetle başlar, örneklerle ilerler ve beş ortamı yan yana karşılaştırır.

## Neden okumalı?

200 kolonluk bir müşteri tablosu, tiplere dikkat edilmeden Python'a alındığında:

| Satır sayısı | Dikkatsiz tiplerle | Doğru tiplerle |
|---:|---:|---:|
| 100 bin | 358 MB | 56 MB |
| 1 milyon | 3,5 GB | 0,55 GB |
| 10 milyon | 35 GB | 5,5 GB |
| 100 milyon | 350 GB | 55 GB |

Daha önemlisi, yanlış tip **veriyi sessizce bozar**: baştaki sıfırlar silinir, kart numaralarının son hanesi
değişir, tarihler bir gün kayar, boş değerler raporlardan kaybolur.

## İçindekiler

| # | Bölüm | Ne öğreneceksiniz? |
|---|---|---|
| 1 | [Veri Tipi Nedir?](docs/01-veri-tipi-nedir.md) | Temel kavramlar, byte, tip seçerken sorulacak 6 soru |
| 2 | [Bellek ve Ölçek](docs/02-bellek-ve-olcek.md) | 100 bin – 100 milyon satırda RAM ve disk; ölçeğe göre strateji |
| 3 | [Sayılar](docs/03-sayilar.md) | `tinyint`, `smallint`, `int`, `bigint`, `decimal`, `float`, `real`, `double`, `money`; hesaplama farkları |
| 4 | [Metin](docs/04-metin.md) | Yazı tipleri, Türkçe karakter sorunları, kategori tipi |
| 5 | [Tarih ve Zaman](docs/05-tarih-ve-zaman.md) | Tarih tipleri, saat dilimi, ay ekleme, gün sayımı, özel tarihler |
| 6 | [Boş Değer (NULL)](docs/06-null.md) | NULL'un iş anlamı ve ortamlar arası davranış farkları |
| 7 | [Evet/Hayır Değişkenleri](docs/07-evet-hayir.md) | Flag kolonları, üç durumlu mantık |
| 8 | [Dataiku](docs/08-dataiku.md) | Dataiku tipleri ve sık yaşanan sorunlar |
| 9 | [Değişken Rehberi](docs/09-degisken-rehberi.md) | TCKN, VKN, müşteri no, isimler, NACE, sektör, cinsiyet, tutar, adet, sözleşme no, kart no, hesap no, işlem tipi, piyasa verisi |
| 10 | [Karar Tablosu](docs/10-karar-tablosu.md) | Tek sayfalık özet ve örnek tablo tanımları |
| 11 | [Python ile Uygulama](docs/11-python-ile-uygulama.md) | pandas ve Polars kod örnekleri |
| 12 | [Sık Hatalar ve Kontrol Listesi](docs/12-sik-hatalar-ve-kontrol-listesi.md) | "Bu belirtinin sebebi ne?" tablosu ve kontrol listesi |
| — | [Sözlük](docs/sozluk.md) | Terimlerin kısa açıklamaları |

## Nasıl okunmalı?

* **İlk kez okuyorsanız:** Bölüm 1, 2 ve 9'u okuyun; diğer bölümlere ihtiyaç duydukça dönün.
* **Bir kolonun tipine karar vermek istiyorsanız:** [Karar Tablosu](docs/10-karar-tablosu.md).
* **Bir hatanın sebebini arıyorsanız:** [Sık Hatalar](docs/12-sik-hatalar-ve-kontrol-listesi.md).

---

*Bellek rakamları 100.000 satırlık örnek bir tabloyla pandas 3 üzerinde ölçülmüş ve satır sayısıyla orantılı
olarak büyütülmüştür. Veritabanı boyutları tip tanımlarından hesaplanmış tahminlerdir.*

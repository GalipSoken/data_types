# 12. Sık Yapılan Hatalar ve Kontrol Listesi

[← Önceki: Python ile Uygulama](11-python-ile-uygulama.md) · [İçindekiler](../README.md) · [Sözlük →](sozluk.md)

---

## Sık yapılan hatalar

"Bu belirtiyi görüyorsam sebebi ne?" sorusu için hızlı başvuru tablosu.

### Kimlik ve yazı

| Belirti | Sebep | Çözüm | Bölüm |
|---|---|---|---|
| Numaralar `12345678901.0` görünüyor | Boş değer içeren tam sayı kolonu ondalıklıya döndü | `Int64` tipi veya yazı | [3](03-sayilar.md) |
| Vergi no, hesap no, NACE bozuk | Tip tahmini sayıya çevirdi | Şemada `string` | [8](08-dataiku.md) |
| Kart numarasının son hanesi değişmiş | 16 haneli sayı float'a sığmaz | Kart no asla sayı değil | [9](09-degisken-rehberi.md) |
| Oracle "value too large" hatası | `VARCHAR2(n)` byte sayar, Türkçe harf 2 byte | `VARCHAR2(n CHAR)` | [4](04-metin.md) |
| Türkçe isimler eşleşmiyor | Python `upper()` Türkçe'yi bilmez | Türkçe dönüşüm | [11](11-python-ile-uygulama.md) |

### Sayılar

| Belirti | Sebep | Çözüm | Bölüm |
|---|---|---|---|
| Tablo çok büyük ve yavaş, tutarlar `object` | `Decimal` nesnesi olarak geldi | `float64`'e çevir | [3](03-sayilar.md) |
| Oran hesabı hep 0 | PostgreSQL/MSSQL tam sayı bölmesi | Birini ondalıklıya çevir | [3](03-sayilar.md) |
| MSSQL'de ortalama küsuratsız | Tam sayı kolonun ortalaması tam sayı döner | Önce ondalıklıya çevir | [3](03-sayilar.md) |
| MSSQL toplamda taşma hatası | Tam sayı toplamı 2,1 milyarı aştı | Önce 8 byte'a çevir | [3](03-sayilar.md) |
| Kur 2 ondalığa yuvarlanmış | Kolon `(18,2)` tanımlı, sessizce yuvarladı | Doğru ondalık sayısı | [3](03-sayilar.md) |
| Bakiye toplamı raporla tutmuyor | Toplamda `float32` kullanıldı | `float64` veya kesin tip | [3](03-sayilar.md) |
| SQL ile Python arasında kuruş farkı | Python yarımları çifte yuvarlar | `ROUND_HALF_UP` | [3](03-sayilar.md) |
| Küçük tipte saçma değerler | Tip taştı (uyarı vermez) | Olabilecek en büyük değere göre seç | [3](03-sayilar.md) |
| MSSQL'e yazarken hata | Python'dan `inf`/`NaN` geldi | Yazmadan önce temizle | [3](03-sayilar.md) |

### Tarih

| Belirti | Sebep | Çözüm | Bölüm |
|---|---|---|---|
| Tarihler bir gün geri kaymış | UTC dönüşümü (İstanbul UTC+3) | Sadece tarih tipi / saat dilimi ayarı | [5](05-tarih-ve-zaman.md) |
| Oracle tarih filtresi eksik kayıt getiriyor | `DATE` saat de taşıyor | Gün aralığı ile filtrele | [5](05-tarih-ve-zaman.md) |
| MSSQL "gün sonu" filtresi ertesi günü getiriyor | `datetime` `.999`'u yuvarlıyor | `< ertesi gün` | [5](05-tarih-ve-zaman.md) |
| Taksit tarihleri Şubat'tan sonra farklı | Oracle `ADD_MONTHS` ay sonu kuralı | Ay sonunu açıkça kodla | [5](05-tarih-ve-zaman.md) |
| `9999-12-31` hata veriyor veya boş oluyor | pandas tarih aralığı | Saniye birimi / boş değere çevir | [5](05-tarih-ve-zaman.md) |
| Müşteriler 126 yaşında görünüyor | `1900-01-01` "bilinmiyor" anlamında kullanılmış | Boş değere çevir | [5](05-tarih-ve-zaman.md) |

### Boş değer (NULL)

| Belirti | Sebep | Çözüm | Bölüm |
|---|---|---|---|
| `NOT IN` sorgusu hiç sonuç döndürmüyor | Alt sorguda NULL var | `NOT EXISTS` | [6](06-null.md) |
| Oracle'dan taşınan veride boş yazılar kayboldu | Oracle'da `''` = NULL | Boş yazıları NULL'a çevir | [6](06-null.md) |
| "En riskli 100" listesi ortamlara göre farklı | NULL'ların sıralamadaki yeri farklı | Sıralamada NULL yerini belirt | [6](06-null.md) |
| MSSQL ikinci boş vergi no'da hata veriyor | MSSQL benzersizlik kuralında tek NULL'a izin verir | Filtreli indeks | [6](06-null.md) |
| `ISNULL(kod, 'BILINMIYOR')` → `'BI'` | MSSQL `ISNULL` uzunluğu keser | `COALESCE` | [6](06-null.md) |
| Python birleştirmesi SQL'den fazla satır verdi | pandas boş anahtarları eşleştirir | Önce boş anahtarları ayır | [6](06-null.md) |
| Raporda "sektörü bilinmeyen" grubu yok | pandas gruplamada boşu siler | `dropna=False` | [6](06-null.md) |
| Veri olmayan ayın hacmi 0 görünüyor | pandas tümü boş toplamı 0 verir | `min_count=1` | [6](06-null.md) |
| Fiyatı eksik günlerde getiri %−100 | Boş fiyat 0 ile dolduruldu | Son geçerli fiyat | [6](06-null.md) |

## Kontrol listesi

Yeni bir tablo tasarlarken veya bir veriyi analize alırken bu listeyi gözden geçirin.

### Her kolon için
- [ ] Hangi değerleri alabileceğini biliyorum (en küçük, en büyük, kaç ondalık).
- [ ] Boş olduğunda ne anlama geldiğini veri sözlüğüne yazdım.
- [ ] Kimlik mi, sayı mı, kod mu, serbest yazı mı? Karar verdim.

### Kimlikler ve yazılar
- [ ] Kimlikler sayı gibi kullanılmıyor ve modele girmiyor.
- [ ] Analiz ortamında açık TC kimlik no, vergi no, kart no yok (KVKK, PCI-DSS).
- [ ] Birleştirme anahtarları tüm tablolarda aynı tipte.
- [ ] Oracle'da `VARCHAR2(n CHAR)`; MSSQL'de Türkçe karakter destekli kurallar.
- [ ] Tekrar eden açıklamalar (sektör adı gibi) ayrı kod tablosunda.

### Sayılar
- [ ] Tam sayı tipleri olabilecek en büyük değere göre seçildi.
- [ ] Para ve tutarlar veritabanında kesin ondalık tipte, yanında döviz kodu var.
- [ ] Kur, fiyat ve faiz için ondalık sayısı yeterli; faiz oranının birimi yazılı.
- [ ] `float32` sadece model özelliklerinde kullanıldı.
- [ ] Tam sayı bölmesi ve MSSQL toplam/ortalama davranışı kontrol edildi.

### Tarihler
- [ ] İş tarihleri saatsiz; olay zamanları saat dilimli.
- [ ] Oracle `DATE` kolonlarının saati sıfır.
- [ ] Dataiku'ya alınan tarihlerde bir gün kayması olup olmadığı kontrol edildi.
- [ ] Ay ekleme ve gün sayımı tek yerde, açıkça kodlandı.
- [ ] Özel anlamlı tarihler (`9999-12-31`, `1900-01-01`) tanımlı ve analizde temizleniyor.

### Boş değerler
- [ ] Boş yazılar NULL'a çevrildi.
- [ ] `NOT IN` yerine `NOT EXISTS` kullanıldı.
- [ ] Sıralamalarda NULL'ların yeri açıkça belirtildi.
- [ ] pandas'ta gruplama ve birleştirme SQL ile aynı sonucu veriyor.
- [ ] Toptan `fillna(0)` yapılmadı.

### Ölçek
- [ ] Tablonun bellekteki boyutu ölçüldü; `object` kolon kalmadı.
- [ ] Sadece gereken kolonlar okunuyor, tipler okuma anında veriliyor.
- [ ] Ara çıktılar Parquet formatında.
- [ ] 10 milyon satırı aşan tablolarda hesaplama veritabanında yapılıyor.

---

[← Önceki: Python ile Uygulama](11-python-ile-uygulama.md) · [İçindekiler](../README.md) · [Sözlük →](sozluk.md)

---
title: "Tablo Bölümlendirme"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 12, 2020
summary: "PostgreSQL Tablo Bölümlendirme"
sidebar: mydoc_sidebar
permalink: mydoc_tablo_bolumlendirme.html
folder: mydoc
---

## Tablo Bölümlendirme (Partitioning)

- PostgreSQL bölümleme (partitioning), mantıksal olarak büyük tabloların daha küçük fiziksel parçalara bölünmesini ifade eder.
- Devide and concure yönteminin Postgres'e yansıması denilebilir.
- Özellikle tablonun yoğun erişilen satırlarının çoğu tek bir bölümde veya az sayıda bölümde olduğu durumlarda büyük tablolar belli bir locige göre küçük tablolara bölünerek daha kolay scale edilebilir.
- PostgreSQL 10 sürümüyle gelen Bildirimli Bölümleme (Declerative Partitioning) özelliği ile birlikte performansı ve sağladığı key wordler ile kullanımı kolaylaşmıştır.
- Bölümleme, indeks boyutunu azaltır ve yoğun olarak kullanılan tablo bölümlerinin belleğe sığmasını sağlayarak sorgu performansını önemli ölçüde etkiler. Sorgular veya güncellemeler tek bir bölümün büyük bir yüzdesine eriştiğinde, indeks veya tüm tabloya dağılmış rastgele erişim okumalarını kullanmak yerine ilgili bölümün sıralı taramasını yaparak daha iyi bir performans gösterir.
- Sadece ilgili bölüme index atarak hem index'lerimizin boyutunu küçültüp hem de index fragmantasyonlarımızın azalmasına olanak sağlar.
- Tablo üzerinde toplu yükleme ve silme işlemleri planlanıyorsa bölümler eklenerek veya kaldırılarak bu işlemler daha verimli şekilde gerçekleştirilebilir. ``ALTER TABLE DETACH PARTITION`` yapmak veya ``DROP TABLE`` kullanarak bir bölümü bırakmak, toplu işlemden çok daha hızlıdır. Bu komutlar, ``DELETE`` nedeniyle oluşacak **VACUUM** yükünü de tamamen önler.
- Bölümlendirmeyle beraber nadiren kullanılan veriler daha ucuz ve yavaş bir depolama ortamına geçirilebilir.
- Üst tabloya atılan index alt tablolar içinde geçerli olur aynı zamanda alt tablolar için de ayrı ayrı index tanımlanabilir.

{% include note.html content="Bir tablo üzerindeki bölümlendirmenin avantajlı durumlar uygulanma noktasına bağlıdır, ancak genel kural tablonun boyutunun veri tabanı sunucusunun fiziksel belleğini aşması durumudur." %}

### Terminoloji

- **Partitioned Table (Bölümlenmiş Tablo)**: Ana büyük table'ı ifade eder.
- **Partition**: Partition edilmiş tablonun her bir bölümünü ifade eder.
- **Partitioning Method (Bölümleme Yöntemi)**: Partition formunu ifade eder. (range, list, hash)
- **Partition Keys (Bölüm Anahtarı)**: Table'ları ayırdığımız column veya column'ları ifade eder.
- **Partition Bounds**: Partitionları ayıran sınırlara denilir.
- **Partition Pruning**: Normalde bizim dışardan ana tabloya attığımız bir query'nin içerde iligili partitiona yönlendirilme işlemi Partition Pruning olarak adlandırılır.

### Bölümlendirme Yöntemleri

**List Partitioning**: PARTITION KEY değeri neyse sadece o değerlere ait veriler parçalanır. Örneğin status alanı *ACTIVE*, *PASIVE* olan değerleri içeren tüm satırlar için ayrıca partition oluşturulması için LIST kullanılır.

**Range Partitioning**: Belirlenen kriterlere göre tablo bölümlenmesi gerçekleşir. Tarih aralığına bölünebileceği gibi belirli objelere göre partition yapılabilir. Örneğin, belirli tarih aralığı için RANGE kullanılabilir.

**Hash Partitioning**: Partition Key’in ait hash değerine göre satır bazlı verilerin partition tablolarına aktarılmasıdır. RANGE ve LIST partitionlara benzer şekilde oluşturulur. Partition tablolarının oluşturulması sırasında **MODULUS** ve **REMAINDER** kavramları önemlidir. MODULUS değeri bizim kaç adet partition tablomuzun olduğunu belirtir ve tüm partition tabloları için sabittir ve değişmez. REMAINDER değeri HASH partition için kullanılan hash key‘in, yani tablomuzu bölmek için kullanacağımız sütun, hash değerini REMAINDER’a böler ve kalan değer ne ise o REMAINDER’in bulunduğu partition tablosuna veriyi aktarır. Örneğin, MODULUS 10 dediğinizde partitioned tablosu 10 adet partition tablosuna sahip demektir ve tüm partition tablolarında sabittir. REMAINDER 0 dan 9 a kadar tüm değer alır. Böylece tablomuzu hangi sütuna göre böleceksek, o sütunun hash değerini MODULUS değerine böler ve örneğin kalan 3 ise REMAINDER 3 olan partition tablosuna veriyi aktarır.

## Bildirimli Bölümleme (Declerative Partitioning)

- PostgreSQL, bölümleme işlemi için bölümleme yönteminden ve bölüm anahtarı olarak kullanılacak sütunların veya ifadelerin listesinden oluşan bir spesifikasyon sunar. Bölümlenmiş bir tabloya eklenen tüm satırlar, bölümleme anahtarının değerine göre bölümlerden birine yönlendirilir.
- Herbir bölüm, bölüm sınırları tarafından tanımlanan verilerden olusan bir alt küme belirtir ve kendi indeksleri, kısıtlamaları ve varsayılan değerlere sahip olabilir. İndeksler her bölüm için ayrı ayrı oluşturulmalıdır.
- Normal bir tabloyu bölümlenmiş bir tabloya dönüştürmek ya da tam tersi mümkün değildir. Ancak, bölümlenmiş bir tablonun bir bölümü normal tabloya dönüştürmek veya bu bölümü, bölümlenmiş bir tablo eklemek ya da bölümlenmiş tablodan bağımsız olacak şekilde ayırmak mümkündür.

Örnekler üzerinde anlatacak olursak; büyük bir dondurma şirketi için bir veritabanı oluşturduğumuzu varsayalım. Şirket her gün en yüksek sıcaklıkları ve her bölgedeki dondurma satışlarını saklıyor. Kavramsal olarak, aşağıdaki gibi bir tablo istiyoruz:

```sql
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
);
```

Çoğu sorgunun yalnızca geçen haftanın, ayın veya çeyreğin verilerine erişeceğini biliyoruz, çünkü bu tablonun ana kullanımı yönetim için çevrimiçi raporlar hazırlamak olacaktır. Saklanması gereken eski veri miktarını azaltmak için, yalnızca en son 3 yıllık veriyi tutmaya karar verdiğinizi varsayalım ve her ayın başında, en eski ayın verilerini sileceksiniz. Bu senaryoda, *measurement* tablosunda üzerinde gereksinimlerimizi karşılamamıza yardımcı olması için bölümlemeyi kullanabiliriz.

Bu senaryoda bildirimsel bölümlemeyi kullanmak için aşağıdaki adımlar uygulanır.

**1**.Bölümleme yöntemini (bu senaryoda RANGE) ve bölüm anahtarı olarak kullanılacak sütun(lar) listesini içeren ``PARTITION BY`` yan tümcesini belirterek bölümlenmiş *measurement* tablosunu oluşturun.

```sql
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (logdate);
```

**2**. Bölümler oluşturun. Her bölüm, üst öğenin bölümleme yöntemine ve bölüm anahtarına karşılık gelen sınırları içermelidir. Varolan bir veya daha fazla bölümdeki değerlerle çakışacak şekilde sınır belirtilerek oluşturulacak yeni bölümünün hataya neden olacağını unutmayın.

{% include warning.html content="Varolan bölümlerden biriyle eşleşmeyen üst tabloya veri eklemek hataya neden olur; bu problem default bir bölüm eklenerek çözülür." %}

Her bölüm için ayrı bir tablespace ve depolama parametresi belirtmek mümkündür. Ayrıca bölümler için bölüm sınırı koşulunu açıklayan tablo kısıtlamaları oluşturmak gerekli değildir. Bölüm kısıtlamaları, bölüme bağlı belirtimden örtük olarak oluşturulur.

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
    FOR VALUES FROM ('2006-02-01') TO ('2006-03-01');

CREATE TABLE measurement_y2006m03 PARTITION OF measurement
    FOR VALUES FROM ('2006-03-01') TO ('2006-04-01');

...
CREATE TABLE measurement_y2007m11 PARTITION OF measurement
    FOR VALUES FROM ('2007-11-01') TO ('2007-12-01');

CREATE TABLE measurement_y2007m12 PARTITION OF measurement
    FOR VALUES FROM ('2007-12-01') TO ('2008-01-01')
    TABLESPACE fasttablespace;

CREATE TABLE measurement_y2008m01 PARTITION OF measurement
    FOR VALUES FROM ('2008-01-01') TO ('2008-02-01')
    WITH (parallel_workers = 4)
    TABLESPACE fasttablespace;
```

Alt bölümlemeyi (sub-partitioning) uygulamak için, bölümler oluşturmak için kullanılan komutlara ``PARTITION BY`` yan tümcesini ekleyin:

```sql
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
    FOR VALUES FROM ('2006-02-01') TO ('2006-03-01')
    PARTITION BY RANGE (peaktemp);
```

Bu işlemden sonra **measurement_y2006m02** bölümüne eklenen her veri *peaktemp* kolonuna göre alt bölümlere yönlendirilecektir. Belirtilen bölüm anahtarı, üst bölümün anahtarı ile çakışabilir, ancak bir alt bölümün sınırlarını belirtirken kabul ettiği veri kümesi, bölümün kendi sınırlarının izin verdiğinin alt kümesini oluşturduğuna dikkat edilmelidir; sistem durumun böyle olup olmadığını kontrol etmez.

**3**.Anahtar kolon ve kolonlar üzerinde bir indeks oluşturun.

```sql
CREATE INDEX ON measurement_y2006m02 (logdate);
CREATE INDEX ON measurement_y2006m03 (logdate);
...
CREATE INDEX ON measurement_y2007m11 (logdate);
CREATE INDEX ON measurement_y2007m12 (logdate);
CREATE INDEX ON measurement_y2008m01 (logdate);
```

Periyodik olarak eski veri bölümlerini kaldırmak veya yeni veriler için bölümler eklemek istemek yaygındır. Bölümlemenin bir diğer avantajı ise büyük miktarda veriyi fiziksel olarak taşımak yerine bölüm yapısını manipüle ederek daha verimli yönetilmesini sağlamaktadır.

Eski verileri kaldırmak için en basit yöntem artık gerekli olmayan bölümü bırakmaktır. Böylece, tablo üzerindeki kayıtları tek tek silmek zorunda olmadığından milyonlarca kaydı çok hızlı bir şekilde silebilir.

```sql
DROP TABLE measurement_y2006m02;
```

İhtiyaç durumunda tercih edilen bir diğer yöntem ise bölümü bölümlenmiş tablodan ayırmak ve kendi başına bir tablo olarak erişimini korumaktır.

```sql
CREATE TABLE measurement_y2008m02 PARTITION OF measurement
    FOR VALUES FROM ('2008-02-01') TO ('2008-03-01')
    TABLESPACE fasttablespace;
```

## Tablo Bölümlendirme Limitleri

Bölümlenmiş tablolar için aşağıdaki sınırlamalar geçerlidir:

- Tüm bölümlerde uygun indeksleri otomatik olarak oluşturmak için kullanılabilir bir yöntem yoktur. Indeksler her bölüme ayrı komutlarla eklenmelidir. Bu ayrıca tüm bölümleri kapsayan bir birincil anahtar veya kısıtlama (constraint) oluşturmanın bir yolu olmadığı anlamına gelir. Her bir partition için ayrı ayrı constraint yaratmak gerekir.
- Bölümlendirilmiş tablolardaki benzersiz kısıtlamalar (Unique constraints), tüm bölüm anahtarı sütunlarını içermelidir. Bir çözüm, bölümlenmiş tablo yerine her bölümde benzersiz kısıtlamalar oluşturmaktır.
- Bölüm, bölümlenmiş tablolardaki BEFORE ROW tetikleyicilerini desteklemez. Gerekliyse; bölümlenmiş tabloda değil, alt bölümlerde ayrı ayrı tanımlanmaları gereklidir.
- Range partition NULL değerlere izin vermez.
- Tabloda bir PK varsa bu partition key içerisinde olmak zorunda.

{% include note.html content="Sürekli gelişen PostgreSQL'de en güncel tablo bölümlendirme limitleri için [](https://www.postgresql.org/docs/current/ddl-partitioning.html)" %}

{% include links.html %}

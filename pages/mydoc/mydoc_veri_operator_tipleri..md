---
title: "PostgreSQL'e Özgü Veri ve Operatör Tipleri"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 3, 2020
summary: "PostgreSQL'e Özgü Veri ve Operatör Tipleri"
sidebar: mydoc_sidebar
permalink: mydoc_veri_operator_tipleri.html
folder: mydoc
---

## PostgreSQL Veri Tipleri

PostgreSQL’deki veri tipleri aşağıda sunulmuştur. PostgreSQL’de kullanılabilen veri tiplerinden bazıları PostgreSQL’e özgüdür, diğer veritabanlarında yer almaz. Bazı veri tipleri ise alt türlere bölünerek farklı kullanım alanlarına uyum sağlamayı kolaylaştırarak performans getirileri sunar. Veri tipleri ve alt türleri bu bölümde incelenecektir.

|Veri Tipi| Veri Tipi | Veri Tipi | Veri Tipi |
|-------|--------|-------|-------|
| bigint | box | circle | lseg |
| bigserial | bytea |date | line |
| bit [ (n) ] | character [ (n) ] | double precision | jsonb |
| bit varying [ (n) ] | character varying [ (n) ] | inet | json |
| boolean | cidr | integer | interval [ fields ] [ (p) ] |
| macaddr | macaddr8 | money | numeric [ (p, s) ] |
| path | pg_lsn | point** | polygon |
| real | smallint | smallserial | serial |
| text | time [ (p) ] | time [ (p) ] w/t TZ | timestamp [ (p) ] |
| timestamp [ (p) ] w/t TZ | tsquery | tsvector | txid_snapshot |
| uuid | xml || |

### Sayısal Veri Tipleri

* Sayısal veri tipleri, tam ya da rasyonel sayılar gibi farklı büyüklükte (magnitude) ve hassasiyette (precision) veri türlerini saklamaya yararlar.
* Sayısal veri tiplerini tam sayı, virgül sonrası hassasiyeti sabit veya değişken rasyonel sayılar ile seri tipindeki sayılar olarak sınıflandırabiliriz. Bu veri tiplerinde saklanan sayılar virgüllü ya da tam sayı şeklinde, virgülden sonraki hassasiyetine karar verebildiğimiz, negatif ya da pozitif sayılar olabileceği gibi, indeks kolonlarında kullanabileceğimiz otomatik artış verebileceğimiz seri tipinde de olabilir.
* Tam sayı türündeki veri tipleri smallint, integer ve bigint olarak ayrılmıştır. Bunların birim boyutları farklıdır ve negatif - pozitif tam sayılar uzayında değer alabilecekleri aralıklar değişkenlik gösterir.

|Veri Tipi| Boyut | Tanım | Değer aralığı |
|-------|--------|-------|-------|
| smallint (int2) | 2 byte | small-range integer | -32768 ile +32767 aralığındaki tam sayılar |
| integer (int4) | 4 byte | typical choice for intege | -2147483648 ile +2147483647 aralığındaki tam sayılar |
| bigint (int8) | 8 byte | large-range integer | -9223372036854775808 ile +9223372036854775807 aralığındaki tam sayılar |

PostgreSQL’de rasyonel sayıları da saklamak için birkaç veri tipi vardır. Bunları virgülden sonraki hassasiyet seviyesini, veri tipinin tanımlanması aşamasında belirlememize veya serbest bırakmamıza göre sınıflandırabiliriz.

{% include note.html content="Decimal ve Numeric tamamen birbirinin aynı özelliklere sahip veri tipleri olup SQL standartlarını sağlayabilmek için ayrı ayrı tanımlanmışlardır. Decimal ve numeric veri tiplerinde saklanacak toplam hane sayısına karar veririz. Bunun için veri tipini tanımlarken **precision** ve **scale** tanımlamalarını kullanırız." %}

|Veri Tipi| Boyut | Tanım | Değer aralığı |
|-------|--------|-------|-------|
| decimal | değişken | Kullanıcı tanımlı hassasiyet, mutlak | Ondalık ayracı öncesinde 131072 haneye kadar sonrasında 16383 haneye kadar |
| numeric | değişken |Kullanıcı tanımlı hassasiyet, mutlak | Ondalık ayracı öncesinde 131072 haneye kadar sonrasında 16383 haneye kadar |

Aşağıdaki şekilde tanımlamalar yapılabilir:

```sql
NUMERIC(precision, scale)
NUMERIC(precision)
NUMERIC
```

Son tanımlamada kolonda azami saklama boyutlarına kadar istenen uzunlukta veri saklanabilir. Yani ondalık ayracının bulunduğu yer değişken olabilir. Örneğin aynı alanda 12.345 ve 1234.5 saklayabilmek gibi. İkinci tanımlamada ondalık toplam hane sayısı es geçilerek sıfır tanımlanmış olmakta yani hane sayısı belirlenmiş tam sayı tanımlanmış olmaktadır.  İlkinde ise toplam haneye precision’la ve her değerin içinde olabilecek ondalık hane sayısına scale ile karar verilir. Bu alana düşen sayısal olmayan değerlerin tanımlanabilmesi için NaN (Not a Number: Sayı değil) değeri de kullanılabilir. NaN değeri kullanılarak yapılan tüm işlemler NaN’la sonuçlanır.

|Veri Tipi| Boyut | Tanım | Değer aralığı |
|-------|--------|-------|-------|
| real | 4 byte | değişken ondalık hassasiyeti, mutlak değil | 6 ondalık hane |
| double precision | 4 byte | değişken ondalık hassasiyeti, mutlak değil | 15 ondalık hane |

Real ve double veri tipleri, tanımı gereği rasyonel olarak tam olarak ifade edilemeyen bunun yerine değerine en yakınsanmış değer olarak kaydedilebilen sayılar için kullanılır. Bu sebeple kesin hesap gerektiren veri tiplerinde bunlar yerine numeric veri tipinin kullanılması tavsiye edilir. Bununla birlikte bu veri tipinde numeric tipine ilave olarak sonsuzluk değerleri de NaN’la birlikte saklanabilmektedir. Hem real hem double veri tipi “Infinity”, “-Infinity” ve “NaN” değerlerini alabilmektedir.

PostgreSQL, IEEE754 standardı uyumluluğundan dolayı float tipini de desteklemektedir. Buna göre aslında float (1) ve float(24) aralığındaki değerler real’a; float(25) ve float(53) aralığındaki değerler de double’a karşılık gelmektedir.

Sayısal değerlerde round fonksiyonu kullanıldığında yuvarlama işlemine verdikleri tepkiler farklıdır. Aşağıdaki örnekte görüleceği gibi numeric bir kolon yuvarlandığında sıfırdan en uzak tam sayıya (yani yukarı) yuvarlanırken, double türünde bir sayı yuvarlandığında (aşağı veya yukarı olduğuna bakılmaksızın) en yakın çift sayıya yuvarlanır.

Bir kolon eğer ID kolonu olarak kullanılacaksa SERIAL tipinde tanımlanabilir. Bu durumda kolonu integer tanımlayıp bu kolon üzerinde bir **SEQUENCE** oluşturmakla aynı işlem yapılmış olunur. Yine integer (ya da int4) tipinde olduğu gibi serial tipi de aslında serial4’e denktir. Daha az yer kaplayan **serial2** (veya smallserial) ve daha geniş değer aralığında çalışabilen **serial8** (veya bigserial) de kullanılabilir.

```sql
SELECT x,
  round(x::numeric) AS num_round,
  round(x::double precision) AS dbl_round
FROM generate_series(-3.5, 3.5, 1) as x;
  x   | num_round | dbl_round
------+-----------+-----------
 -3.5 |        -4 |        -4
 -2.5 |        -3 |        -2
 -1.5 |        -2 |        -2
 -0.5 |        -1 |        -0
  0.5 |         1 |         0
  1.5 |         2 |         2
  2.5 |         3 |         2
  3.5 |         4 |         4
(8 rows)
```

Bu noktada SEQUENCE’lere değinmek gerekirse, Sequence’ler bir tablodaki kolonun otomatik olarak değer almasını sağlayan nesnelerdir. Sequence’ler içinde yaratıldıkları (veya belirtilirse bir başka) şema tarafından sahiplenilirler. Bir sequence’in ismi kendisiyle aynı şema içindeki diğer sequence’ler, tablolar, indeksler, view’lar ve yabancı tablolar (foreign table) arasında benzersiz olmalıdır.

Bir sequence oluşturulma anında veri türüne de karar verilebilir. Varsayılan olarak bigint olan sequence’ler smallint ve int türlerinde de seçilebilir. Bunun için oluşturma anında data_type parametresine istenen veri türü belirtilir. Ayrıca yine oluşturulma anında sequence’in başlayacağı değer, alabileceği en büyük değer veya artış değeri gibi birkaç parametreye karar verilerek işlevsellik kazandırılabilir. Artış değeri için kullanılan increment değişkeninin negatif bir değer alması halinde sequence azalarak da ilerleyebilir. Ayrıca sequence tanımı sırasında minvalue ve maxvalue verilebiliyor olmasına rağmen başlangıç değerine START sözcüğü kullanılarak spesifik olarak da karar verilebilmektedir.

Örneğin varsayılan özelliklerle bir sequence yaratıp START ile başlangıç değeri verdiğimizde, ilk sequence talebimizde ```(nextval('serial')``` ile talep ediyoruz) başlangıç değeri 101’in sonra da bir artarak 102’nin geldiğini görüyoruz.

```sql
CREATE SEQUENCE serial START 101;

SELECT nextval('serial');

 nextval
---------
     101
SELECT nextval('serial');

 nextval
---------
     102
```

Bir sequence yaratıldıktan sonra, nextval, currval, lastval ve setval fonksiyonları ile işlerlik kazanabilmektedir. Ayrıca cache özelliği ile sequence üretimi önbelleklenerek hızlı hale de getirilebilir. Cache değişkeninin varsayılan değeri 1 olduğu için her seferinde bir otomatik değer üretimi yapılabilir. Aşağıdaki örnekteki gibi bir tabloya satır eklerken indeks atadığımız ID kolonu için ``nextval()`` fonksiyonunu kullanabiliriz.

```sql
INSERT INTO distributors VALUES (nextval('serial'), 'nothing');
```

### Metinsel Veri Tipleri

Hem char, hem de varchar metinsel verileri saklamak için kullanılan veri tipleridir. Varchar ve char tanımlanma aşamasında verilen n karakteri saklarken, n’den daha uzun bir metin girişi olduğunda hata döndürerek metni n karaktere kırparlar. Varchar(n) tanımlı bir kolonda kolon uzunluğundan kısa metinler girildiğinde, girilen metin kadar fiziksel yer harcarken, char(n) tanımlı kolonda metnin kalanı boşluk karakteri ile doldurularak n karaktere tamamlanır ve n karakterlik fiziksel yer harcar. Diğer veri tipi olan text için bir karakter sınırı tanımlaması yapılmaz. Değişken boyutlu metinler hata vermeksizin kabul edilebilir.

|Veri Tipi | Tanım |
|-------|--------|-------|-------|
| character varying(n), varchar(n) | değişken uzunluklu sabit boyutlu |
| character(n), char(n) | sabit uzunluklu, boş alanlar boşlukla doldurulur |
| text | değişken sınırsız uzunluklu|

Yine char(n) için n boyut tanımı yapılmazsa 1 karakter uzunluğunda alınır. Yani char = char(1) olur.

{% include note.html content="char(n) ve varchar(n) arasında ciddi bir performans farkında ziyade diskte kaplanan alanda farklılık vardır. Bununla birlikte char(n) üç metin tipi içinde en maliyetli olan olarak varsayılabilir." %}

char ve varchar alanlarda tanımlanan boyuttan daha uzun bir string girişi (örneğin insert veya cast ile) olduğunda boyut aşımı hatası alınır ve veri azami boyuta kırpılarak girilir.

```sql
CREATE TABLE test1 (a character(4));
INSERT INTO test1 VALUES ('ok');
SELECT a, char_length(a) FROM test1; -- (1)

  a   | char_length
------+-------------
 ok   |           2


CREATE TABLE test2 (b varchar(5));
INSERT INTO test2 VALUES ('ok');
INSERT INTO test2 VALUES ('good      ');
INSERT INTO test2 VALUES ('too long');
ERROR:  value too long for type character varying(5)
INSERT INTO test2 VALUES ('too long'::varchar(5)); -- explicit truncation
SELECT b, char_length(b) FROM test2;

   b   | char_length
-------+-------------
 ok    |           2
 good  |           5
 too l |           5
```

### Binary Veri Tipi

Binary tipindeki verileri saklamak için PostgreSQL bytea veri tipini kullanır. Bytea veri tipi input ve output için iki formatı destekler: hex ve escape. Her iki format da daima input olarak kabul edilirken output için varsayılan format hex’tir.

SQL standardı BLOB veya BINARY LARGE OBJECT olarak adlandırılmış farklı bir binary veri formatını tanımlamaktadır. Input formatı bytea’den farklıdır fakat sunulan fonksiyon ve operatörler çoğunlukla aynıdır.

Hex veri formatında binary veriler bit başına 2 hexadecimal hane olarak saklanır ve veri stringinin önünde ‘\x’ ifadesi konularak escape formatından ayırt edilmesi sağlanır. Bu format çoğu uygulama ve protokol tarafından kabul edilen bir format olduğu ve bu sebeple dönüşümü daha hızlı olduğu için kullanımda tercih edilmesi tavsiye edilir.

Escape formatı ise geleneksel PostgreSQL bytea tipi formatıdır. Bu format, binary veri stringini ascii karakter dizisi olarak işlerken ascii olarak temsil edilemeyen karakterler için kaçış karakteri karşılıklarını kullanır. Böylece bu formatta da dönüşüm sağlanabilmiş olur. Ascii olarak ifade edilmeyen octetlere örnek aşağıda verilmiştir.

![Escape formatı](/images/veri_operator_tipleri_sekil1.png)

Burada tırnak ve backslash karakterlerinin kullanımına dikkat edilmelidir.

```sql
SET bytea_output = 'escape';

SELECT 'abc \153\154\155 \052\251\124'::bytea;
     bytea
----------------
 abc klm *\251T
```

### Parasal Veri Tipleri

Parasal bilgileri saklamak için PostgreSQL’de MONEY veri tipi kullanılabilir. Bu veri tipinde geçerli olan para tipi bilgisi ve saklanacak verinin ondalık hassasiyeti veritabanının lc_monetary ayarında belirlenir.  Kuruş hassasiyeti de diyebileceğimiz bu değer varsayılan olarak virgül sonrası 2 hanedir.

Bu veri tipini kullanırken dikkatli olunması gereken durumlar, özellikle farklı para birimindeki değerlerin aynı tabloya girilmesi halinde ortaya çıkar. Farklı para birimindeki bir tablo yedeği farklı yerel para birimi ayarına sahip bir veritabanına atılırken ya da veritabanında ayarlanandan farklı para biriminde bir nesneyi çalışılan tabloya yazarken dikkat edilmelidir.

Money türünde kaydedilen veriler farklı veri tiplerine dönüştürülebilir. Bununla birlikte içinde parasal varlıkları kaydettiğimiz başka veri kolonlarının money tipine dönüşümlerinde dikkat edilmelidir.

```sql
SELECT '12.34'::float8::numeric::money;
```

Örneğin yukarıda istenen veri float’tan numeric’e dönüşürken yuvarlama hatalarıyla karşılaşma ihtimalimiz yüksektir. Bunu önlemek için parasal varlıklar float tipinde kolonlarda saklanmamalıdır. Money tipini benzer dönüşüme soktuğumuzda ise böyle bir sorun yaşamayız.

```sql
SELECT '52093.89'::money::numeric::float8;
```

Son olarak money tipinde bir parasal varlıkla yapılan çarpma / bölme işlemleridir. Money tipindeki veriler integer tipinde bir veriye bölündüğünde ondalık kısımdaki veri kaybedilir ve tam sayı sonuç elde edilir. Oysa aynı tam sayının real ya da double karşılığı kullanılırsa kuruş bilgisi kaybedilmeden hesaplama tamamlanır. Yine money cinsinden bir veri money cinsinden başka bir veriye bölünürse sonuç double olarak elde edilir, yani sonuç verimiz artık money cinsinde değildir, numerik bir değere dönüşür. Bunun sebebi para birimlerinin birbirlerini götürmesidir.

### Tarih / Zaman Veri Tipleri

PostgreSQL çok geniş bir tarih - zaman veri tipi kütüphanesi sunmaktadır. Bu sayede tarih ve zaman fonksiyonları çok etkin bir şekilde kullanılabilmekte ve birbirine dönüştürülerek çok etkin hesaplamalar yapılabilmektedir.
Bu amaçla kullanılan veri tipleri arasında timestamp, time, date ve interval sayılabilir. Aşağıda tabloda bu veri tipleri görülmekte ve alabilecekleri en küçük ve en büyük değerler de gösterilmektedir. Ayrıca zamansal çözünürlük kolonunda seçilen zaman biriminin ondalık hassasiyeti de görünmektedir. Bu değer veri tipi tanımlama aşamasında girilmelidir ve 0 ile 6 arasında değerler alabilir.

|Veri Tipi | Depolama Alanı | Tanım | Asgari Değer | Azami Değer |Zamansal çözünürlük |
|-------|--------|-------|-------|-------|-------|
| timestamp [ (p) ] [ without time zone ] | 8 byte | Tarih ve saat (zaman dilimi bilgisi hariç) | 4713 MÖ | 294276 MS | 1 mikrosaniye / 14 hane |
| timestamp [ (p) ] with time zone | 8 byte | Tarih ve saat (zaman dilimi bilgisi dahil) | 4713 MÖ | 294276 MS | 1 mikrosaniye / 14 hane |
| date | 4 byte | Tarih | 4713 MÖ | 5874897 MS | 1 gün |
| time [ (p) ] [ without time zone ] | 8 byte | Saat (zaman dilimi bilgisi hariç) | 00:00:00 | 24:00:00 | 1 mikrosaniye / 14 hane |
| time [ (p) ] with time zone | 12 byte | Saat (zaman dilimi bilgisi dahil) | 00:00:00 +1459 | 24:00:00-1459 | 1 mikrosaniye / 14 hane |
| interval [ fields ] [ (p) ] | 16 byte | Zaman araligi | -178000000 yıl | 178000000 yıl | 1 mikrosaniye / 14 hane |

Tarih zaman veri tipleri arasında INTERVAL diye özel bir veri tipi vardır. Bu belli bir zamandan ziyade zamansal hesaplamalarda zaman aralığı olarak kullanılabilir ve aşağıdaki listede sunulan zaman aralıklarını saklayabilir.

```sql
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND
```

Aşağıdaki tablo tarih tipindeki veriler için bazı örnek gösterimler sunmaktadır.

![Tarih tipindeki veriler için örnek](/images/veri_operator_tipleri_sekil2.png)

Benzer şekilde aşağıdaki tabloda da saat cinsinden veri tiplerinin gösterimine örnekler verilmiştir.

![Saat tipindeki veriler için örnek](/images/veri_operator_tipleri_sekil3.png)

Timestamp with time zone ve time with time zone türündeki veri tipleri zaman dilimi bilgisini de saklamaktadır. Eğer veri tipi “without time zone” şeklinde ise ve girilen veri zaman dilimi bilgisi içeriyorsa, hata verilmez fakat zaman dilimi bilgisi veriden silinir. Zaman dilimi tutan veri türlerinde zaman dilimi bilgisi aşağıdaki şekillerde gösterilebilir.

![Saat tipindeki veriler için örnek](/images/veri_operator_tipleri_sekil4.png)

Timestamp veri tipi tarihi, zamanı ve (istenirse) zaman dilimi bilgisini saklayabilir.  Ayrıca timestamp verisinde milattan önceki yıllar için BC, milattan sonraki yıllar için AD ifadesi de saklanabilir. aşağıda timestamp türünde farklı örnekler bulunmaktadır.

```sql
1999-01-08 04:05:06
January 8 04:05:06 1999 CET
TIMESTAMP '2004-10-19 10:23:54+02'
```

Timestamp with time zone türünde bir veri, her zaman tabloda UTC (Universal Coordinated Time) ya da diğer bilinen ismiyle GMT (Greenwich Mean Time) olarak saklanır. Bu sebeple veritabanına giren bir veride açıkça zaman dilimi belirtildiyse bu değer saklanırken UTC’ye çevrilir. Zaman dilimi belirtilmeden girildiyse, varsayılan olarak sistemin zaman dilimi kullanılır ve UTC’ye çevrilerek kaydedilir. Veritabanında bu şekilde kaydedilmiş veriler gösterilirken ise UTC kaydedilmiş bu veriler yerel zaman diliminde gösterilir. Eğer veritabanındaki değerlerin başka bir zaman diliminde gösterilmesi isteniyorsa ya zaman dilimi değiştirilir ya da AT TIME ZONE yapısı kullanılır.

```sql
SELECT TIMESTAMP '2001-02-16 20:38:40' AT TIME ZONE 'America/Denver';
Result: 2001-02-16 19:38:40-08

SELECT TIMESTAMP WITH TIME ZONE '2001-02-16 20:38:40-05' AT TIME ZONE 'America/Denver';
Result: 2001-02-16 18:38:40

SELECT TIMESTAMP '2001-02-16 20:38:40-05' AT TIME ZONE 'Asia/Tokyo' AT TIME ZONE 'America/Chicago';
Result: 2001-02-16 05:38:40
```

PostgreSQL bünyesinde zamansal anlamda bazı özel ifadeler bulunmaktadır. Bunlar da aşağıdaki tabloda özetlenmiştir. Sistem zamanını almak için ayrıca ``CURRENT_DATE``, ``CURRENT_TIME``, ``CURRENT_TIMESTAMP``, ``LOCALTIME``, ``LOCALTIMESTAMP`` fonksiyonları da mevcuttur.

![zamansal anlamda bazı özel ifadeler](/images/veri_operator_tipleri_sekil5.png)

Zaman bilgisinin nasıl göründüğü ile ilgili farklı görünüm konvansiyonları vardır. PostgreSQL bu konvansiyonlar arasında geçiş yapabilmek için postgresql.conf içinde bulunan ``“SET datestyle”`` komutunu kullanmamıza izin verir. Ayrıca **PGDATESTYLE** çevre değişkeni de aynı amaçla kullanılabilir. Yine benzer şekilde zaman dilimindeki oturum bazlı değişimler için ``“SET TIME ZONE”``  fonksiyonu kullanılabilir.

Özellikle tarih saat hesaplamalarında ya da veritabanında yapılacak güncellemelerde kullanmak üzere veya tarihsel farklılıklarının sonucunu sunmak için ``INTERVAL`` denilen ve tarih - saat farkı olarak açıklanabilecek bir veri türü de vardır. Bu veri türünde zaman birimi olarak *microsecond, millisecond, second, minute, hour, day, week, month, year, decade, century, millennium* ile bunların (sonuna s, es veya ies eklenerek yapılmış çoğulları) ya da kısaltmaları kullanılabilir.  

Örneğin ``'1 12:59:10'`` ve ``'1 day 12 hours 59 min 10 sec'`` ifadeleri birbirinin aynısıdır ve herhangi bir tarih - zaman türü veri üzerinde yapılacak işlemlere dahil edilebilir. Aşağıdaki tabloda zamanı veya zaman aralığını ifade ederken kısaltma olarak kullanılabilecek tanımlar bulunmaktadır.

![zamanı veya zaman aralığını ifade ederken kısaltma olarak kullanılabilecek tanımlar](/images/veri_operator_tipleri_sekil6.png)

### Boolean Veri Tipleri

{% include links.html %}

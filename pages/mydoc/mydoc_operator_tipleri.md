---
title: "PostgreSQL Fonksiyon ve Operatör Tipleri"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 5, 2020
summary: "PostgreSQL Fonksiyon ve Operatör Tipleri"
sidebar: mydoc_sidebar
permalink: mydoc_operator_tipleri.html
folder: mydoc
---

## PostgreSQL Fonksiyon ve Operatör Tipleri

PostgreSQL, çok sayıda fonksiyon ve operatör sunmaktadır. Kullanıcılar, bu gömülü fonksiyon ve operatörlere ilave olarak kendileri de fonksiyon ve operatör tanımı yapabilirler. Sistemde tanımlı tüm fonksiyon ve operatörleri psql’de \df ve \do komutlarıyla listeleyebilirler.

### Mantıksal Operatörler

PostgreSQL’de kullanılan mantıksal operatörler ``AND``, ``OR`` ve ``NOT`` olarak listelenebilir. Örneğin TRUE, FALSE veya NULL değerler alabilecek a ve b gibi iki boolean tipinde değişkenin aldığı değerlere göre AND, OR ve NOT mantıksal operatörlerinin sonuçlar üzerindeki etkileri aşağıdaki tablolarda görülmektedir.

{% include image.html file="veri_operator_tipleri_sekil10.png" url="/pg-gelistirici/images/veri_operator_tipleri_sekil10.png" alt="mantıksal operatörler"%}

AND ve OR operatörlerinin solundaki ve sağındaki değişkenlerin yer değiştirmesi, sonucu değiştirmez. Fakat bu operatörlerin bir yanında alt ifadelerin olması farklı sonuçlara sebep olabilir.

### Karşılaştırma Fonsksiyon ve Operatörler

PostgreSQL’de kullanılan karşılaştırma operatörleri aşağıdaki tabloda verilmiştir.

| Operatör | Açıklama |
|-------|--------|
| < | Küçüktür |
| > | Büyüktür |
| <= | Küçük eşit |
| >= | Büyük eşit |
| = | Eşit |
| <> veya != | Eşit değildir |

Bu karşılaştırma operatörleri ilgili tüm veri tipleri için kullanılabilir. Bu operatörlerle yapılan karşılaştırmaların tamamında boolean bir ifade geri döner.

{% include note.html content="Birden fazla karşılaştırma operatörü yanyana kullanılamaz. Örneğin 1<2<3 ifadesi hata döndürür. Çünkü 1<2 den dönen TRUE ifadesi TRUE<3 operasyonunda hata verecektir." %}

Anlatılan bu operatörlere benzer karşılaştırma işlerini yapan ifadeler de bulunmaktadır. Bunlar aşağıda listelenmiştir.

| İfade | Açıklama |
|-------|--------|
| **a** BETWEEN **x** AND **y** | arasında |
| **a** NOT BETWEEN **x** AND **y** | arasında değil (aralık dışında) |
| **a** BETWEEN SYMMETRIC **x** AND **y** | karşılaştırma değerleri sıralanırsa arasında |
| **a** NOT BETWEEN SYMMETRIC **x** AND **y** | karşılaştırma değerleri sıralanırsa arasında değil (aralık dışında) |
| **a** IS DISTINCT FROM **b** | a, b’den farklı (eşit değil) |
| **a** IS NOT DISTINCT FROM **b** | a, b’nin aynısı (eşit) |
| **expression** IS NULL | “null" |
| **expression** IS NOT NULL | “not null” |
| **expression** ISNULL | “is null” mu? |
| **expression** NOTNULL | “is not null” mu? |
| **boolean_expression** IS TRUE | “is true” mu? |
| **boolean_expression** IS NOT TRUE| “is false” mu yoksa “is unknown” mu? (bilinmiyor mu?) |
| **boolean_expression** IS FALSE | “is false” mu? |
| **boolean_expression** IS NOT FALSE| “is true” mu yoksa “is unknown” mu? (bilinmiyor mu?) |
| **boolean_expression** IS UNKNOWN | “is unknown” mu? (bilinmiyor mu?) |
| **boolean_expression** IS NOT UNKNOWN | “is unknown” mu? (bilinmiyor mu?) |

Bunlara ilave olarak NULL olan ve olmayan ifadelerin bulunması için iki tane de fonksiyon bulunmaktadır. Bunlar ``num_nulls()`` ve ``num_nonnulls()`` fonksiyonlarıdır.

```sql
num_nonnulls (1, NULL, 2) ⇒ 2
num_nulls (1, NULL, 2) ⇒ 1
```

### Matematiksel Fonksiyonlar ve Operatörler

Çoğu PostgreSQL tipi için matematiksel operatörler sunulmuştur. Bu operatörler ve örnekleri aşağıdaki tabloda verilmiştir. En alttaki bitwise operatörler bit ve bit varying tipindeki verilerle integral veri tiplerinde geçerlidir.

| Operatör | Tanım |
|-------|--------|
| `+` | toplama |
| `-` | çıkarma |
| `*` | çarpma |
| `/` | bölme |
| `%` | mod alma (bölmenin kalanını verir, ör: 5 % 4=1) |
| `^` | üs alma (soldan sağa işler, ör: 2^3 = 8 |
| `|/` | karekök (ör:``|/4 = 2``) |
| `||/` | küpkök (ör:``||/27 = 3``) |
| `!` | faktöriyel (sayının sonrasında kullanılırsa, ör: 5! = 120) |
| `!!` | faktöriyel (sayının öncesinde kullanılırsa, ör: !!5 = 120) |
| `@` | mutlak değer (ör: @-5 = 5) |
| `&` | bitwise AND |
| `|` | bitwise OR |
| `#` | bitwise XOR |
| `~` | bitwise NOT |
| `<<` | bitwise shift left |
| `>>` | bitwise shift right |

Aşağıdaki tablo ise PostgreSQL’deki matematiksel fonksiyonları göstermektedir.

| Fonksiyon | Dönen verinin tipi | Tanım |
|-------|--------|--------|
| ``abs(x)`` | Giren veri tipinin aynısı | mutlak değer |
| ``cbrt(double)`` | double | küp kök |
| ``ceil(double veya numeric)`` | Giren veri tipinin aynısı | yukarı yuvarlar |
| ``ceiling(double veya numeric)`` | Giren veri tipinin aynısı | yukarı yuvarlar (ceil ile aynı) |
| ``degrees(double)`` | double | radyanı dereceye çevirir |
| ``div(y numeric, x numeric)`` | numeric | y/x işleminin tam (bölüm) kısmı |
| ``exp(double veya  numeric)`` | Giren veri tipinin aynısı | eksponansiyel |
| ``floor(double veya numeric)`` | Giren veri tipinin aynısı | aşağı yuvarlar |
| ``ln(double veya  numeric)`` | Giren veri tipinin aynısı | doğal (e tabanlı) logaritma |
| ``log(double veya numeric)`` | Giren veri tipinin aynısı | baz (10 tabanlı) logaritma |
| ``log(b numeric, x numeric)`` | numeric | b tabanlı logaritma |
| ``mod(y, x)`` | Giren argümanların tiplerinin aynı | y/x işleminin kalan kısmı (modu) |
| ``pi()`` | double | pi sabiti |
| ``power(a double, b double)`` | double | a üssü b |
| ``power(a numeric, b numeric)`` | numeric | a üssü b |
| ``radians(double)`` | double | dereceden radyana çevirme |
| ``round(double veya numeric)`` | Giren veri tipinin aynısı | en yakın tam sayıya (aşağı veya yukarı) yuvarlama |
| ``round(v numeric, s int)`` | numeric | s tane ondalık haneyi yuvarlar |
| ``scale(numeric)`` | int | bir rasyonel sayıdaki ondalık hane sayısı |
| ``sign(double veya numeric)`` | Giren veri tipinin aynısı | argümanın işareti (-1, 0, +1) |
| ``sqrt(double veya numeric``) | Giren veri tipinin aynısı | karekök |
| ``trunc(double veya numeric)`` | Giren veri tipinin aynısı | sıfıra doğru yuvarlar (küsüratı atar) |
| ``trunc(v numeric, s int)`` | numeric | sıfıra doğru s haneyi yuvarlar (s hanenin küsüratını sıfıra yakınsayacak şekilde atar) |
| ``width_bucket(operanddouble, b1 double, b2 double, c int)`` veya ``(operandnumeric, b1 numeric,b2 numeric, count int)`` veya ``(operandanyelement,thresholds anyarray)`` | int | operand kısmında tanımlanan kolondaki verileri normal dağılıma göre dağıtarak b1 ve b2 aralığını c eşit aralığa böler. operand kolonundaki her değerin c+2 aralıktan hangisine düştüğünü gösterir. |

Ayrıca rastgele sayılar üretmek için `random()` ve `setseed(double)` fonksiyonları bulunmaktadır. Setseed, random’dan hemen önce kullanılır.

Trigonometrik işlemler içinse `sin(x)`, `cos(x)`, `tan(x)`, `cot(x)`, `asin(x)`, `acos(x)`, `atan(x)` ve `atan2(y,x)` fonksiyonları bulunmaktadır. Bu fonksiyonlarda kullanılan x ve y değerleri radyan yerine derece cinsinden girilecekse sind(x), cosd(x), tand(x), cotd(x), asind(x), acosd(x), atand(x) ve atan2d(y,x) fonksiyonları kullanılır. Derece ve radyan fonksiyonları için uygun olanların seçilmesi yerine radians() ve degrees() fonksiyonlarının kullanılarak dönüşüm yapılması da bir seçenektir.

### Metinsel Fonsksiyon ve Operatörler

Bu kısımda anlatılacak operatör ve fonksiyonlar char, varchar ve text tipindeki verilerde kullanılabilir. Bazı fonksiyonlar ise bit-string tipindeki verilerde de kullanılabilir.

| Fonksiyon | Dönen Veri Tipi | Açıklama |
|-------|--------|--------|
| ``string || string`` | text | İki metni birbirine bitiştirir |
| ``string || non-string`` veya ``non-string || string`` | text | metinsel ve metinsel olmayan verileri birbirine birleştirir |
| ``bit_length(string)`` | int | metindeki bit sayısı |
| ``char_length(string)`` veya ``character_length(string)`` | int | metindeki karakter sayısı |
| ``lower(string)`` | text | metni küçük karaktere çevirir |
| ``octet_length(string)`` | int | metindeki byte sayısı |
| ``overlay(string placing string from int [forint])`` | text | metin içinde istenen bir parçayı başka bir metin parçasıyla değiştirir |
| ``position(substring instring)`` | int | metin içinde aranan metin parçasının ilk bulunduğu konum |
| ``substring(string [fromint] [for int])`` | text | metinden bir metin parçası alır |
| ``substring(string from pattern)`` | text | metinden, POSIX regex ifadesine uyan bir metin parçası alır |
| ``substring(string from pattern for escape)`` | text | metinden, SQL regex ifadesine uyan bir metin parçası alır |
| ``trim([leading | trailing | both] [characters] from string)`` veya ``trim([leading | trailing | both] [from] string [,characters] )`` | text |metnin başında, sonunda veya her iki tarafında istenen karakteri (varsayılan olarak boşluk karakterini) siler |
| ``upper(string)`` | text | metni büyük karaktere çevirir |

### Binary String Fonsksiyon ve Operatörleri

Bu fonksiyonlar bytea tipindeki kolonlar üzerinde işlevseldir. Bu fonksiyonlar, bazı string fonksiyonlarının bytea veri tipinde uygulanabilir halidir.

| Fonksiyon | Dönen Veri Tipi | Açıklama |
|-------|--------|--------|
| ``string || string`` | bytea | İki metni birbirine bitiştirir |
| ``octet_length(string)`` | int | binarydeki byte sayısı |
| ``overlay(string placing string from int [for int])`` | bytea | metinde istenen yerdeki karakterleri yeni metin parçasıyla değiştirir |
| ``position(substringin string)`` | int | metin içinde aranan metin parçasının ilk bulunduğu konum |
| ``substring(string[from int] [forint])`` | bytea | metinden bir metin parçası alır |
| ``trim([both] bytesfrom string)`` | bytea | metnin başında, sonunda veya her iki tarafında istenen byte’ı siler |

Bunlar haricinde binary string tipi verilerin manipülasyonu için ``btrim()``, ``decode()``, ``encode()``, ``get_bit()``, ``get_byte()``, ``length()``, ``md5()``, ``set_bit()``, ``set_byte()``, ``sha224()``, ``sha256()``, ``sha384()``, ``sha512()`` fonksiyonları da bulunmaktadır.

### Desen Eşleştirme

PostgreSQL’de veri içinde belli bir desene uyan kayıtların getirilmesi önemlidir. Bunu sağlamak için PostgreSQL’de ``LIKE``, ``SIMILAR TO`` ve ``POSIX regex`` ifadeler kullanılmaktadır. Bu ifadelere örnekler de aşağıda sunulmuştur.

**LIKE**, kendisinden önce gelen ifade içinde kendinden sonra gelen ifadenin bulunup bulunmadığı hallere bakarken, **NOT LIKE** tam tersini yapar. Eğer bu operatörler bir SQL sorgusunda WHERE’den sonra kullanılıyorsa arama yapılan kolonda LIKE sonrasında gelen ifadenin doğruluğu test edilir. Örnekleri aşağıda bulunmaktadır.

```sql
'abc' LIKE 'abc'    true
'abc' LIKE 'a%'     true
'abc' LIKE '_b_'    true
'abc' LIKE 'c'      false
```

LIKE yerine ``ILIKE`` operatörünü kullanırsak aramada büyük - küçük harf duyarlılığı devre dışı bırakılır. ILIKE, SQL standartında olmayıp PostgreSQL’e özgü bir özelliktir. Desen eşleşmesi için istenen deseni belirlerken herhangi tek karakter için alt çizgi ``_`` karakteri ve herhangi belirsiz / değişken sayıda karakter için yüzde ``%`` sembolleri kullanılabilir.

LIKE’a benzer şekilde **SIMILAR TO** ifadesi de desen eşleşme amaçlı kullanılmaktadır. Bu ikisi arasındaki fark, SIMILAR TO operatörünün POSIX regex ifadelerinde kullanılan çeşitli operatörlerin de desen eşleşme amacıyla kullanıma katılmış olmasıdır. Aşağıdaki ifadeler SIMILAR TO için kullanılabilir.

| Operatör | Açıklama |
|-------|--------|
| ``a|b``| a ya da b’den herhangi birisi |
| ``*`` | bir önceki karakterin 0 veya daha fazla sefer tekrar etmesi |
| ``+`` | bir önceki karakterin 1 veya daha fazla sefer tekrar etmesi |
| ``?`` | bir önceki karakterin 0 veya 1 sefer tekrar etmesi |
| ``{m}`` | bir önceki karakterin tam m sefer tekrar etmesi |
| ``{m,}`` | bir önceki karakterin m veya daha fazla sefer tekrar etmesi |
| ``{m, n}`` | bir önceki karakterin en az m, en çok n sefer tekrar etmesi |
| ``( )`` | gruplanmış birden fazla seçim kriteri öbeğini tanımlar |
| ``[...]`` | POSIX regex ifadelerindeki gibi bir karakter sınıfını tanımlar |

Aşağıda bazı örnekler verilmiştir:

```sql
'abc' SIMILAR TO 'abc'      true
'abc' SIMILAR TO 'a'        false
'abc' SIMILAR TO '%(b|d)%'  true
'abc' SIMILAR TO '(b|c)%'   false
```

POSIX regex tipi ifadelerin oluşturulabilmesi için aşağıdaki operatörler kullanılabilir. Bunlar için de bazı örnekler verilmiştir.

| Operatör | Açıklama | Açıklama |
|-------|--------|--------|
| ``~`` | regex’e uymalı, büyük - küçük harf duyarlılığı var | 'thomas' ~ '.*thomas.*' |
| ``~*`` | regex’e uymalı, büyük - küçük harf duyarlılığı yok| 'thomas' ~* '.*Thomas.*' |
| ``!~`` | regex’e uymamalı, büyük - küçük harf duyarlılığı var | 'thomas' !~ '.*Thomas.*' |
| ``!~*`` | regex’e uymamalı, büyük - küçük harf duyarlılığı yok | 'thomas' !~* '.*vadim.*' |

POSIX ifadeler LIKE ve SIMILAR TO araçlarından çok daha güçlü araçlar içerdiği için bunları kullanarak çok daha kapsamlı desen sorguları oluşturulabilir.

```sql
'abc' ~ 'abc'    true
'abc' ~ '^a'     true
'abc' ~ '(b|d)'  true
'abc' ~ '^(b|c)' false
```

### Tarih / Saat Fonsksiyon ve Operatörleri

Tarih ve saat tipi veriler, çok sayıda PostgreSQL operatör ve fonksiyonu kullanılarak birbirine ya da numeric, double, string gibi tiplere dönüştürülebilir. Aşağıda tarih - saat formatlama fonksiyonları vardır. Bu fonksiyonlarda ilk argüman formatlanacak zaman, ikinci argüman ise format ifadesidir.

| Fonksiyon | Dönen Veri Tipi | Açıklama | Örnek |
|-------|--------|--------|--------|
|``to_char(timestamp,text)`` | text | time stamp’i string’e çevirir | to_char(current_timestamp, 'HH12:MI:SS') |
|``to_char(interval,text)`` | text | interval’i string’e çevirir | to_char(interval '15h 2m 12s', 'HH24:MI:SS') |
|``to_char(int, text)`` | text | integer’ı string’e çevirir | to_char(125, '999') |
|``to_char(double, text)`` | text | real/double’ı string’e çevirir | to_char(125.8::real, '999D9') |
|`to_char(numeric, text)`| text | numeric’i string’e çevirir | to_char(-125.8, '999D99S') |
|``to_date(text, text)``| date | string’i date’e çevirir | to_date('05 Dec 2000', 'DD Mon YYYY') |
|``to_number(text, text)``| numeric | string'i numeric’e çevirir | to_number('12,454.8-', '99G999D9S') |
|``to_timestamp(text,text)`` | timestamp with time zone | string'i time stamp’a çevirir | to_timestamp('05 Dec 2000', 'DD Mon YYYY') |

Aşağıda bazı örnekler verilmiştir.

| İfade | Sonuç |
|-------|--------|
| ``to_char(current_timestamp, 'Day, DD  HH12:MI:SS')`` | 'Tuesday  , 06  05:39:18' |
| ``to_char(current_timestamp, 'FMDay, FMDD  HH12:MI:SS')`` | 'Tuesday, 6  05:39:18' |
| ``to_char(-0.1, '99.99')`` | '  -.10' |
| ``to_char(-0.1, 'FM9.99')`` | '-.1' |
| ``to_char(-0.1, 'FM90.99')`` | '-0.1' |
| ``to_char(0.1, '0.9')`` | ' 0.1' |
| ``to_char(12, '9990999.9')`` | '    0012.0' |
| ``to_char(12, 'FM9990999.9')`` | '0012.' |
| ``to_char(485, '999')`` | ' 485' |
| ``to_char(-485, '999')`` | '-485' |
| ``to_char(485, '9 9 9')`` | ' 4 8 5' |
| ``to_char(1485, '9,999')`` | ' 1,485' |
| ``to_char(1485, '9G999')`` | ' 1 485' |
| ``to_char(148.5, '999.999')`` | ' 148.500' |
| ``to_char(148.5, 'FM999.999')`` | '148.5' |
| ``to_char(148.5, 'FM999.990')`` | '148.500' |
| ``to_char(148.5, '999D999')`` | ' 148,500' |
| ``to_char(3148.5, '9G999D999')`` | ' 3 148,500' |
| ``to_char(-485, '999S')`` | '485-' |
| ``to_char(-485, '999MI')`` | '485-' |
| ``to_char(485, '999MI')`` | '485 ' |
| ``to_char(485, 'FM999MI')`` | '485' |
| ``to_char(485, 'PL999')`` | '+485' |
| ``to_char(485, 'SG999')`` | '+485' |
| ``to_char(-485, 'SG999')`` | '-485' |
| ``to_char(-485, '9SG99')`` | '4-85' |
| ``to_char(-485, '999PR')`` | '<485>' |
| ``to_char(485, 'L999')`` | 'DM 485' |
| ``to_char(485, 'RN')`` | '        CDLXXXV' |
| ``to_char(485, 'FMRN')`` | 'CDLXXXV' |
| ``to_char(5.2, 'FMRN')`` | 'V' |
| ``to_char(482, '999th')`` | ' 482nd' |
| ``to_char(485, '"Good number:"999')`` | 'Good number: 485' |
| ``to_char(485.8, '"Pre:"999" Post:" .999')`` | 'Pre: 485 Post: .800' |
| ``to_char(12, '99V999')`` | ' 12000' |
| ``to_char(12.4, '99V999')`` | ' 12400' |
| ``to_char(12.45, '99V9')`` | ' 125' |
| ``to_char(0.0004859, '9.99EEEE')`` | ' 4.86e-04' |

Tarih - zaman fonksiyonları ile dört işlem yapılabilir. Burada tarih - zaman tipinde veriler (tarih, zaman, zaman aralığı, saat vs)  bileşenleri birbirleriyle dört işleme sokulabilir. Aşağıda örnekleri vardır.

| Operatör | Örnek | Sonuç |
|-------|--------|--------|
| + | ``date '2001-09-28' + integer '7'`` | date '2001-10-05' |
| + | ``date '2001-09-28' + interval '1 hour'``| timestamp '2001-09-28 01:00:00' |
| + | ``date '2001-09-28' + time '03:00'``| timestamp '2001-09-28 03:00:00' |
| + | ``interval '1 day' + interval '1 hour'`` | interval '1 day 01:00:00' |
| + | ``timestamp '2001-09-28 01:00' + interval '23 hours'`` | timestamp '2001-09-29 00:00:00' |
| + | ``time '01:00' + interval '3 hours'`` | time '04:00:00' |
| - | ``- interval '23 hours'`` | interval '-23:00:00' |
| - | ``date '2001-10-01' - date '2001-09-28'`` | integer '3' (days) |
| - | ``date '2001-10-01' - integer '7'`` | date '2001-09-24' |
| - | ``date '2001-09-28' - interval '1 hour'`` | timestamp '2001-09-27 23:00:00' |
| - | ``time '05:00' - time '03:00'`` | interval '02:00:00' |
| - | ``time '05:00' - interval '2 hours'`` | time '03:00:00' |
| - | ``timestamp '2001-09-28 23:00' - interval '23 hours'`` | timestamp '2001-09-28 00:00:00' |
| - | ``interval '1 day' - interval '1 hour'`` | interval '1 day -01:00:00' |
| - | ``interval '1 day' - interval '1 hour'`` | interval '1 day 15:00:00' |
| *| ``900* interval '1 second'`` | interval '00:15:00' |
| *| ``21* interval '1 day'`` | interval '21 days' |
| *| ``double precision '3.5'* interval '1 hour'`` | interval '03:30:00' |
| / | ``interval '1 hour' / double precision '1.5'`` | interval '00:40:00' |

Bunların haricinde çok sayıda tarih / zaman fonksiyonu da bulunmaktadır. Bunlar da aşağıdaki tabloda sunulmuştur.

| Fonksiyon | Dönen Veri Tipi | Açıklama | Örnek | Sonuç |
|-------|--------|-------|--------|--------|
|``age (timestamp, timestamp)``| interval | iki argümanı birbirinden çıkartarak bir zaman aralığı elde eder.| `age(timestamp '2001-04-10', timestamp '1957-06-13')` | 43 years 9 mons 27 days|
|``age (timestamp)`` | interval | Bir zaman ile o anki tarihin farkını alır| ``age(timestamp '1957-06-13')``| 43 years 8 mons 3 days |
|``clock_timestamp()``| timestamp with time zone | Sorgu çalıştırma anındaki timestamp'i getirir |||
| ``current_date`` | date | Tarihi getirir |  |  |
| ``current_time`` | time with time zone | Saati getirir |  |  |
| ``current_timestamp`` | timestamp with time zone | Transaction başlangıcındaki timestampi getirir | | |
| ``date_part (text, timestamp)`` | double precision | bir tarih değerinin belirtilen kesimini alır | ``date_part('hour', timestamp '2001-02-16 20:38:40')`` | 20 |
| ``date_part (text, interval)`` | double precision | bir tarih değerinin belirtilen zaman birimindeki kısmını alır | `date_part('month', interval '2 years 3 months')` | 3 |
| ``date_trunc (text, timestamp)`` | timestamp | bir tarih değerinin belirtilen zaman biriminden sonraki kısmını atar | `date_trunc('hour', timestamp '2001-02-16 20:38:40')` | 2001-02-16 20:00:00 |
| ``date_trunc (text, interval)`` | interval | bir tarih değerini belirtilen zaman aralıklarına böldüğünde sonra kalan artık kısmı atar | `date_trunc('hour', interval '2 days 3 hours 40 minutes')` | 2 days 03:00:00 |
| ``extract (field from timestamp)`` | double precision | bir tarih değerinin belirtilen kesimini alır | `extract(hour from timestamp '2001-02-16 20:38:40')` | 20 |
| ``extract (field from interval)`` | double precision | bir tarih değerinin belirtilen zaman birimindeki kısmını alır | `extract(month from interval '2 years 3 months')` | 3 |
| ``isfinite (date)`` | boolean | bir tarihin sonlu değerde olup olmadığını test eder (tarih +/- sonsuz mu) | `isfinite(date '2001-02-16')` | true |
| ``isfinite (timestamp)`` | boolean | bir timestamp’in sonlu değerde olup olmadığını test eder (timestamp +/- sonsuz mu) | isfinite(timestamp '2001-02-16 21:28:30') | true |
| ``isfinite (interval)`` | boolean | bir zaman aralığının sonlu değerde olup olmadığını test eder (interval +/- sonsuz mu) | `isfinite(interval '4 hours')` | true |
| ``justify_days (interval)`` | interval | bir zaman aralığını temel zaman birimi ay olacak şekilde ifade eder | `justify_days(interval '35 days')` | 1 mon 5 days |
| ``justify_hours (interval)`` | interval | bir zaman aralığını temel zaman birimi gün olacak şekilde ifade eder | `bir zaman aralığını temel zaman birimi gün olacak şekilde ifade eder` | 1 day 03:00:00 |
| ``justify_interval (interval)`` | interval | bir zaman aralığını justify_days ve justify_hours, fonksiyonlarını bir arada kullanacak şekilde ayarlar | `justify_interval(interval '1 mon -1 hour')` | 29 days 23:00:00 |
| ``localtime`` | time | yerel saati verir | | |
| ``localtimestamp`` | timestamp | yerel saate göre timestamp üretir. | | |
| ``make_date (year int, monthint, day int)`` | date | yıl, ay ve gün alanlarını kullanarak tarih üretir | `make_date(2013, 7, 15)` | 2013-07-15 |
| ``make_interval(years intDEFAULT 0, months int DEFAULT 0, weeks int DEFAULT 0, daysint DEFAULT 0, hours intDEFAULT 0, mins int DEFAULT 0, secs double precision DEFAULT 0.0)`` | interval | yıl, ay, hafta, gün, saat, dakika ve saniye alanlarını kullanarak interval üretir | `make_interval(days => 10)` | 10 days |
| ``make_time (hour int, min int,sec double precision)`` | time | saat, dakika ve saniye alanlarını kullanarak zaman  üretir | `make_time(8, 15, 23.5)` | 08:15:23.5 |
| ``make_ timestamp (year int,month int, day int, hour int,min int, sec double precision)`` | timestamp | yıl, ay, gün, saat, dakika ve saniye alanlarını kullanarak timestamp üretir | `make_timestamp(2013, 7, 15, 8, 15, 23.5)` | 2013-07-15 08:15:23.5 |
| ``make_ timestamptz (year int,month int, day int, hour int,min int, sec double precision, [ timezone text ])`` | timestamp with time zone | yıl, ay, gün, saat, dakika ve saniye alanlarını kullanarak timestamp with time zone üretir. | `make_timestamptz(2013, 7, 15, 8, 15, 23.5)` | 2013-07-15 08:15:23.5+01 |
| ``now()`` | timestamp with time zone | transaction başlangıcındaki timestamp with timezone bilgisi | | |
| ``statement_ timestamp()`` | timestamp with time zone | sorgu başlangıcındaki timestamp with timezone bilgisi | | |
| ``timeofday()`` | text | Sorgu çalıştırma anındaki timestampi text olarak getirir |  |  |
| ``transaction_ timestamp()`` | timestamp with time zone | transaction başlangıcındaki timestampi getirir |  |  |
| ``to_timestamp (double precision)`` | timestamp with time zone | Unix epokunu (1970-01-01 00:00:00+00 anından beri geçen saniyeler) timestamp’e dönüştürür | `to_timestamp(1284352323)` | 2010-09-13 04:32:03+00 |

### XML Fonsksiyon ve Operatörleri

Bu bölümde anlatılacak fonksiyonlar ve fonksiyon benzeri ifadeler xml türünde kaydedilmiş veriler üzerinde işlevseldir. Fonksiyon benzeri ifadeler arasında *xmlparse* ve *xmlserialize* bulunmakta olup bunlar başka türlerden xml’e ve xml’den başka türlere tip dönüşümü yapmakta kullanılır. Bu fonksiyonların birçoğunun kullanılabilmesi için kurulumun ``-configure --with-libxml`` ile konfigüre edilmiş olması gerekmektedir.

SQL verilerinden XML içerik üretmek amacıyla kullanılan bazı fonksiyonlar arasında ``xmlcomment( )``, ``xmlconcat( )``, ``xmlelement( )``, ``xmlforest( )``, ``xmlpi( )``, ``xmlroot( )``, ``xmlagg( )`` sayılabilir. Bu fonksiyonların tamamı, ya çeşitli xml elementlerini ya da bütün bir xml dökümanını oluşturmak için kullanılabilir. Bu fonksiyonlar için bazı örnekler aşağıda verilmiştir.

Örneğin ``xmlconcat( )`` fonksiyonu bir dizi xml değerini birleştirerek tek bir xml içeren tek bir içeriğe dönüştürür.

```sql
SELECT xmlconcat('<abc/>', '<bar>foo</bar>');

      xmlconcat
----------------------
 <abc/><bar>foo</bar>
```

``xmlelement( )``, istenen bir xml elementini, özniteliğini ve içeriğini yaratır.

```sql
SELECT xmlelement(name foo);

 xmlelement
------------
 <foo/>

SELECT xmlelement(name foo, xmlattributes('xyz' as bar));

    xmlelement
------------------
 <foo bar="xyz"/>

SELECT xmlelement(name foo, xmlattributes(current_date as bar), 'cont', 'ent');

             xmlelement
-------------------------------------
 <foo bar="2007-01-26">content</foo>
```

``xmlforest( )``, liste olarak verilen bir dizi xml elementini ve onların içeriklerini xml olarak oluşturur.

```sql
SELECT xmlforest('abc' AS foo, 123 AS bar);

          xmlforest
------------------------------
 <foo>abc</foo><bar>123</bar>


SELECT xmlforest(table_name, column_name)
FROM information_schema.columns
WHERE table_schema = 'pg_catalog';

                                         xmlforest
-------------------------------------------------------------------------------------------
 <table_name>pg_authid</table_name><column_name>rolname</column_name>
 <table_name>pg_authid</table_name><column_name>rolsuper</column_name>
```

``xmlpi( )`` fonksiyonu bir xml işleme komutu oluşturur. Bu fonksiyonun kullanımında içerik üzerinde bazı kontroller yapılmalıdır, yoksa oluşan xml’de hatalar ortaya çıkabilir.

```sql
SELECT xmlpi(name php, 'echo "hello world";');

            xmlpi
-----------------------------
 <?php echo "hello world";?>

```

``xmlroot( )`` fonksiyonu ise bir xml içeriğinin kök tag’indeki bazı öznitelikleri düzenlemeye yarar.

```sql
SELECT xmlroot(xmlparse(document '<?xml version="1.1"?><content>abc</content>'),
               version '1.0', standalone yes);

                xmlroot
----------------------------------------
 <?xml version="1.0" standalone="yes"?>
 <content>abc</content>
```

XML fonksiyonları arasında xml dosya kontrol ifadeleri de bulunmaktadır. Bunlar aşağıda listelenmiştir. Bu ifadeler sorgu içinde kullanılarak boolean değerler döndürürler.

```sql
IS DOCUMENT
IS NOT DOCUMENT
XMLEXISTS
xml_is_well_formed
```

Son olarak XML fonksiyonları içinde XML işleme fonksiyonları sayılabilir. Bunlar ``xpath( )``, ``xpath_exists( )``, ``xmltable( )`` ile ``map`` fonksiyonları ``table_to_xml( )``, ``query_to_xml( )`` ve ``cursor_to_xml( )``’dir.

{% include links.html %}

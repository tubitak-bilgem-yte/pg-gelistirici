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

![mantıksal operatörler](/images/veri_operator_tipleri_sekil9.png)

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

### Matematiksel Fonsksiyon ve Operatörler

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

{% include links.html %}

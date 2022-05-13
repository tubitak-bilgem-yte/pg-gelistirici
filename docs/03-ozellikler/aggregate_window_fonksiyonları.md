---
layout: default
title: Özetleme ve Bölümleme Fonksiyonları
parent: Öne Çıkan Özellikleri
nav_order: 6
---

## Özetleme ve Bölümleme Fonksiyonları

### Özetleme (AGGREGATE) Fonksiyonları

Birçok başka veritabanı ürününde de olduğu gibi, PostgreSQL de özetleme fonksiyonlarını desteklemektedir. Özetleme fonksiyonları çok sayıda satırdan gelen bir kolona ait verileri derleyerek tek bir sonuç hesaplar ve bunu sunar. Yapılabilecek hesaplama seçenekleri arasında satır sayısını saymak (COUNT), gelen satırlardaki değerlerin toplamını (SUM), ortalamasını (AVG), standart sapmasını (STDDEV), en büyük (MAX) ve en küçük (MIN) değerlerini bulmak seçenekleri sayılabilir. Örnek olarak, farklı yerlerin sıcaklık okumalarının tutulduğu bir meteoroloji tablosunda en yüksek sıcaklık okunduğu girdiyi bulabiliriz.

```sql
SELECT max(temp_lo) FROM weather;

max
-----
  46
(1 row)
```

Eğer bu sıcaklık değerinin okunduğu şehri görmek isteseydik ve şu sorguyu deneseydik ne olurdu?

```sql
SELECT city FROM weather WHERE temp_lo = max(temp_lo);
```

Bu sorgu çalışmayacaktır. Bunun sebebi özetleme fonksiyonları (``min``, ``max``, ``avg``, ``sum``, ``stddev``) sorgunun **WHERE** kısmında kullanılamaz. Bunun yerine aşağıdaki gibi bir alt sorgu kullanmak sorulan soruya cevap olacaktır.

```sql
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);

    city
---------------
 San Francisco
(1 row)
```

Bunun çalışabiliyor olmasının sebebi, alt sorgunun kendi içinde önce çalışarak sonuç döndürmesi ve WHERE cümlesine bu sonucun iletilmesidir. Alt sorgu kendi dışında dönen sorgudan bağımsız çalışmaktadır.

Özetleme fonksiyonları ``GROUP BY`` ifadesinin içinde de oldukça işlevseldir. Örneğin, her şehir için ölçülmüş en yüksek sıcaklığı bulabiliriz.

```sql
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city;

    city      | max
---------------+-----
 Hayward       |  37
 San Francisco |  46
(2 rows)
```

Bu sayede her şehir için tek satırlık çıktı almış oluruz, ya da diğer bir deyişler her şehir için ölçülmüş tüm değerler GROUP BY ifadesiyle ayrı ayrı alt kümelere ayrılır ve ``max(temp_lo)`` ifadesiyle özetlenerek en büyük değerleri getirilir.

Eğer her alt küme için de bir filtreleme yaparsak ``HAVING`` ifadesini kullanabiliriz. Aşağıdaki örnekte tüm şehirler için aynı kümelemeyi yaparken, max sıcaklığı 40 C’den düşük olanları görüntülemekte.

```sql
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;

 city   | max
---------+-----
 Hayward |  37
(1 row)
```

Sorguyu son defa özelleştirerek bu şartlara "isimleri S ile başlayan" şehirlerden bu seçimin yapılması talebini ekleyelim. Bu durumda aşağıdaki sorguyu yazmalıyız.

```sql
SELECT city, max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%'
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

Bu sefer WHERE ifadesi kullandık. Burada SQL’in WHERE ve HAVING ifadelerine bakmak ve özetleme fonksiyonlarıyla nasıl etkileştiklerini anlamak gerekiyor. WHERE ve HAVING arasındaki temel fark şudur: WHERE gruplamadan önceki girdi satırları seçer ve yapılacak özetleme işlemi öyle hesaplanır. Bunun yanında HAVING grupları, GROUP BY ile gruplama yapıldıktan sonra süzer ve özetleme fonksiyonları bundan da sonra uygulanır. Tam da bu sebepten dolayı WHERE ifadesi özetleme fonksiyonu bulunduramaz. Daha da ötesi HAVING ise izin verilmesine rağmen, neredeyse özetleme fonksiyonu olmadan bir işe yaramaz. Yani özetleme fonksiyonu bulunmayan bir ``GROUP BY..HAVING`` ifadesi yazılabilir ama gerçekten çok az durum için fonksiyoneldir.

### Window (Bölümleme) Fonksiyonları

Window (bölümleme) fonksiyonları ise, bir tabloda birbiriyle bir şekilde ilişkili satırları seçerek oluşturduğu alt gruplar üzerinde çeşitli hesaplamalar yapabilir. Bu kabiliyet, aggregate (özetleme) fonksiyonlarının yaptığı işle kıyaslanabilir. Fakat özetleme fonksiyonları ilişkili satırları tek satıra indirgeyerek özetlerken, bölümleme fonksiyonları birbiriyle ilişkisi tanımlanan satırları, içeriklerinin tamamını koruyacak şekilde gösterir ve ilave olarak ilişki grubuna dair bilgileri de sunar.

Açıklamak gerekirse özetlemek fonksiyonları ile kullandığımız **AVG** fonksiyonu örneğin her departman içindeki toplam ortalama maaşı gösterirken, ``PARTITION BY`` kullanarak departmanlarına göre bölümlediğimiz bir sorguda herkesin maaşının yanında departman ortalamasını görebiliriz. Bu sayede herkesin ortalamadan sapmasını ya da yüzdesel oranını bulmamız mümkün hale gelir. Örneklerini aşağıda görelim.

```sql
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;


 depname   | empno | salary |          avg
-----------+-------+--------+-----------------------
 develop   |    11 |   5200 | 5020.0000000000000000
 develop   |     7 |   4200 | 5020.0000000000000000
 develop   |     9 |   4500 | 5020.0000000000000000
 develop   |     8 |   6000 | 5020.0000000000000000
 develop   |    10 |   5200 | 5020.0000000000000000
 personnel |     5 |   3500 | 3700.0000000000000000
 personnel |     2 |   3900 | 3700.0000000000000000
 sales     |     3 |   4800 | 4866.6666666666666667
 sales     |     1 |   5000 | 4866.6666666666666667
 sales     |     4 |   4800 | 4866.6666666666666667
(10 rows)
```

Eğer biz sorguda ``OVER PARTITION BY`` yerine ``GROUP BY`` kullansaydık karşımıza 3 tane departmanın ortalama maaşının olduğu özet bir tablo gelecekti. Fakat OVER sayesinde *depname* kullanılarak bölümlere ayrılan PARTITION’lar bazında bir avg fonksiyon hesaplaması yapılarak orjinal tablodan çağrılan üç kolondaki her satıra bu bilgi ilave edildi. Dolayısıyla burada bölümleme işini PARTITION BY ifadesi, bölümlenmiş birimlere avg()’nin window fonksiyon olarak uygulanması işini ise OVER ifadesi sağlamış oldu.

Tabi oluşturulmuş bu bölümlerdeki her satırın bölümü içindeki sıralamasını da merak edebiliriz. Sıralama yapmak için kullandığımız ORDER BY’ı partition ifadesinin içine ekleyerek ve kolonlar arasına da bir ``rank()`` fonksiyonu ilave ederek aşağıdaki gibi bir çıktı elde ederiz.

```sql
SELECT depname, empno, salary,
       rank() OVER (PARTITION BY depname ORDER BY salary DESC)
FROM empsalary;
 depname   | empno | salary | rank
-----------+-------+--------+------
 develop   |     8 |   6000 |    1
 develop   |    10 |   5200 |    2
 develop   |    11 |   5200 |    2
 develop   |     9 |   4500 |    4
 develop   |     7 |   4200 |    5
 personnel |     2 |   3900 |    1
 personnel |     5 |   3500 |    2
 sales     |     1 |   5000 |    1
 sales     |     4 |   4800 |    2
 sales     |     3 |   4800 |    2
(10 rows)
```

Burada *develop* ve *sales* departmanlarında ikinci sıra paylaşıldığı için üçüncülük bulunmuyor. Eğer sorgu WHERE, GROUP BY ya da HAVING gibi ilave anahtar kelime kullanımından dolayı üretilen sonuçları filtreleseydi, filtrelenmiş tablo kullanılarak üretilen sonuç tablosu girdi olarak düşünülecek ve bölümleme fonksiyonları giriş tablosu olarak kullanılacak bu sanal tablo haricindeki satırları hesabına katmayacaktı.

Eğer bir sıralama ihtiyacı yoksa sorguda ORDER BY’ın yer almasına gerek yoktu. Buna ilave olarak sorguda OVER’ı bırakıp PARTITION BY’ı düşürmek de mümkün olabilir. Eğer bir bölümleme yapmadan window fonksiyonu kullanmak istiyorsak bu seçenek düşünülebilir. Aşağıdaki örnekteki sonuç elde edilecektir.

```sql
SELECT salary, sum(salary) OVER () FROM empsalary;

salary |  sum  
--------+-------
   5200 | 47100
   5000 | 47100
   3500 | 47100
   4800 | 47100
   3900 | 47100
   4200 | 47100
   4500 | 47100
   4800 | 47100
   6000 | 47100
   5200 | 47100
(10 rows)
```

Bölümleme fonksiyonlarıyla ilgili bir başka önemli kavram vardır. Sözü geçen her satır için, içinde bulunduğu bölüm (partition) tarafından kapsanan bir veri alt kümesi vardır ve buna çerçeve (window frame) denir. Bazı bölümleme fonksiyonları sadece çerçevelere (bölümler, partition) ait satırlar üzerinde iş yaparlar. Varsayılan olarak ORDER BY ifadesi kullanıldıysa çerçeve, bölümün başından sonuna kadar tüm satırları kapsar. Bu kapsamda geçerli satır ile bölümde öncesindeki ve sonrasındaki tüm satırlar bulunmaktadır.

Yukarıdaki örnek düşünüldüğünde sum fonksiyonu, sorguda bir bölümleme yapılmadan OVER uygulandığı için bütün satırlara, bütün tablonun toplamını ekleyecektir. Peki OVER grubunun içine PARTITION BY kullanmadan (bölümleme yapmadan) ORDER BY kullanırsak ne olacak? Bu durumda OVER (ve kendisine bağlı çalışan sum() bölümleme fonksiyonu) kümülatif toplama işlemi oluşturacaktır. Örnek aşağıdadır.

```sql
SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary;

salary |  sum  
--------+-------
   3500 |  3500
   3900 |  7400
   4200 | 11600
   4500 | 16100
   4800 | 25700
   4800 | 25700
   5000 | 30700
   5200 | 41100
   5200 | 41100
   6000 | 47100
(10 rows)
```

Burada dikkat edilmesi gereken önemli bir nokta OVER’ın çalışma şeklidir. Aslında her satırda yeni bir bölüm oluşturmakta ve hesaplama sonucunu her seferinde yeniden oluşan bölüm için yapmaktadır. Bu nokta tekrarlanan satırlarda kendini net bir şekilde göstermektedir. Tekrarlanan iki maaş değeri 4800 ve 5200’ün olduğu satırlarda sum hesabı değişmemektedir. Bunun sebebi tekrarın bölümü değiştirmiyor oluşudur.

Eğer sorgunun içinde birden fazla bölümleme fonksiyonu kullanılacaksa bunların her biri teorik olarak ayrı OVER ifadeleri kullanılarak yazılabilir. Fakat bu uygulama hata üretmeye açık, tekrarlı sonuç vermeye meyilli sonuçlar döndürebilir. Bunun yerine her çerçeve, ayrı bir WINDOW olarak adlandırılabilir ve bunun üzerinde OVER kullanılabilir. Örneğin;

```sql
SELECT sum(salary) OVER w, avg(salary) OVER w
  FROM empsalary
  WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

Bölümleme sorgularında rank( ) ile birlikte kullanabileceğimiz ilave birkaç fonksiyon daha bulunmaktadır. Bunlar arasında satır numaralarını veren ``row_number( )``, her satırın içinde bulunduğu bölüme oranını veren ``percent_rank( )``, her bölüm içindeki sıralamaya göre ilk, son ya da n’inci değeri veren ``first_value``, ``last_value`` ya da ``nth_value`` fonksiyonları sayılabilir.  

Aşağıdaki tablo çeşitli bölümleme fonksiyonlarının listesini gösterir. Bu fonksiyonlar, OVER gibi bölümleme fonksiyonları için geçerli bir söz dizimine uyumlu bir şekilde çağrılmalıdır. Bunlara ilave olarak kullanıcıların tanımladığı fonksiyonlar, genel amaçlı özetleme fonksiyonları (aggregate functions) da bölümleme fonksiyonu gibi kullanılabilir. Daha önce de bahsedildiği üzere, özetleme fonksiyonları OVER ile birlikte kullanıldığında bölümleme fonksiyonları gibi kullanılabilirler.

| Fonksiyon | Geri Dönen Veri Tipi | Açıklama |
|-------|--------|--------|
| ``row_number()`` | bigint | bir satırın bulunduğu bölüm içindeki satır numarası, 1’den başlar. |
| ``rank()`` | bigint | her satırın içinde bulunduğu bölüm (partition) içindeki sırasını tutar. her bölümde bu sıralama 1’den başlar, fakat kontrol edilen değere göre birbirinin aynı olan veya bazı değerlerin atlandığı bir atama görülebilir. |
| ``dense_rank()`` | bigint | her satırın içinde bulunduğu bölüm (partition) içindeki sırasını tutar. rank’tan farkı tüm değerler ardışıktır. |
| ``percent_rank()`` | double precision | Satırın bağıl (yüzdesel) mertebe (rank) değeri : (rank() - 1) / (bölümdeki toplam satır sayısı - 1) |
| ``cume_dist()`` | double precision | kümülatif dağılım: (bir bölümde satırdan sonra gelen ilişkili satır sayısı) / (bölümdeki toplam satır sayısı) |
| ``ntile(num_buckets integer)`` | integer | bölümü argüman sayısı bölerek içine elemanları eşit olarak dağıtır. |
| ``first_value(value any)`` | same type as **value** | İçinde bulunulan bölümün ‘value’ kolonundaki ilk değerini getirir |
| ``last_value(value any)`` | same type as **value** | İçinde bulunulan bölümün ‘value’ kolonundaki son değerini getirir |
| ``nth_value(value any, nth integer)`` | same type as **value** | İçinde bulunulan bölümün ‘value’ kolonundaki n.inci değerini getirir |

{% include links.html %}

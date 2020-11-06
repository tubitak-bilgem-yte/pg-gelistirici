---
title: "PostgreSQL İndeks Türleri ve Kullanımları"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 6, 2020
summary: "PostgreSQL İndeks Türleri ve Kullanımları"
sidebar: mydoc_sidebar
permalink: mydoc_index_turleri.html
folder: mydoc
---

## İndeks Türleri ve Kullanımları

İndeksler veritabanı performansını geliştirmenin önemli yollarından birisidir. Bir indeks sayesinde veritabanı, arama sonucunda istenen satırları, indeksler olmadan yapabileceğinden daha hızlı getirmektedir. Fakat tabi ki veritabanına belli bir maliyetleri de vardır. Bu iki bilgiyi göz önüne aldığımızda, indeksler mantıklı kullanılmaları sonucunda performans getirileri büyük olacaktır. Bir indeks aşağıdaki gibi tanımlanabilir.

```sql
CREATE INDEX test1_id_index ON test1 (id);
```

İndeks tanımlandığında ilave bir müdahale gerekmez. Sistem, tabloda bir güncelleme olduğunda indeksi de güncelleyerek kullanıma hazır hale getirir. Sonrasında bir sorguda indeksi kullanmanın tablo üzerinde ardışık tarama (Sequential Table Scan) yapmaktan daha performanslı olduğunu düşündüğünde de kullanır. Bu noktada PostgreSQL’in sorgu planlamasına yardımcı olabiliriz. Periyodik olarak çalıştıracağımız ``ANALYZE`` komutu, sorgulara yönelik performans istatistiklerinin güncellenmesinde kullanılır ve ve sorgu planlayıcısının daha akılcı tercihler yapmasını sağlar.

Büyük tablolarda indeks oluşturma işlemi uzun sürecektir. PostgreSQL indeksi oluştururken SELECT sorgularına izin verir, fakat INSERT, UPDATE, DELETE tipi veri manipülasyon komutlarını indeksin oluşturulması işi tamamlanana kadar bekletir. PostgreSQL bu noktada sistem yöneticisine indeksi oluştururken INSERT, UPDATE, DELETE yapabilmesine izin veren ``CONCURRENTLY`` anahtar sözcüğünü sunar. ``CREATE INDEX CONCURRENTLY`` ifadesini kullanırken dikkat edilecek birkaç husus olsa da bu imkanı verebilmektedir.

Indeks tamamen oluşturulduktan sonra veritabanı üzerinde bakım masrafı oluşturacaktır. Çünkü, tablo her değiştiğinde indeksin de arayı kapatması, kendini tabloyla senkronize etmesi gerekecektir. Bu her data manipülasyonu işlemine ek bir süre olarak yansıyacaktır. Tam da bu sebeple indeks seçimi doğru yapılmalı, sorgularda nadiren kullanılan ya da kullanılmayan indeksler tablolardan çıkarılmalıdır.

İndeksler tek kolon üzerine uygulanabileceği gibi, aynı anda birden fazla kolona da tanımlanabilir. Bu türde bir tanımlamaya sadece **B-tree**, **BRIN**, **GiST** ve **GIN** indeksleri izin vermektedir. Bunun örneği şu şekildedir.

```sql
CREATE TABLE test2 (
  major int,
  minor int,
  name varchar
);

CREATE INDEX test2_mm_idx ON test2 (major, minor);
```

{% include tip.html content="Genellikle tek bir kolon üzerinde indeks uygulamak performans arttırma açısından bakıldığında yeterli sonuç verir. Üç kolondan daha fazlasında oluşturulan indeksler ise tam tersine, tablo özel olarak bu indekslerin kullanımı için tasarlanmamışsa performans kaybına sebep olacaktır." %}

PostgreSQL çok sayıda indekse izin verir: **B-tree**, **Hash**, **GiST**, **SP-GiST**, **GIN** ve **BRIN**. Her indeks türü, farklı sorgu ihtiyaçlarına uygun başka algoritmalarla çalışır. Varsayılan olarak ``CREATE INDEX`` dediğimizde B-tree indeksi yaratılır.

### B-tree

B-tree’ler eşitlik ve değer aralığı tipindeki sorgular için verinin tasnif edilmesi mantığına dayalıdır ve birçok duruma uygun bir performans sunar. Bunun yanı sıra, PostgreSQL de eşitlik ve değer aralığı tipindeki sorgularda ``<``, ``>``, ``<=``, ``=>``, ``=`` gibi operatörleri görür görmez **B-tree indeksi** kullanacağını göz önüne alır.

`BETWEEN` ile `IN` çifti gibi operatör kombinasyonlarına denk yapılar ayrıca B-tree indeks araması ile de gerçekleştirilebilir. Aynı şekilde, bir indeks kolonu üzerindeki ``IS NULL`` veya ``IS NOT NULL`` şartında da B-tree indeksi kullanılabilir. Sorgu planlayıcısı B-tree indeksi ``LIKE`` operatörünün kullanımında da devreye alır. B-tree indeksler ayrıca veriyi sıralı bir şekilde almakta da kullanılır. Her zaman basit bir arama ve sıralama algoritmasından daha hızlı olmasa da çoğu zaman tercih edilecek kadar faydalıdır.

Diğer indekslerden farklı olarak B-tree indeksi döndürdüğü sonuç kümesi üzerinde sıralama kabiliyetine sahiptir. Yani B-tree kullanarak indekslediğimiz kolonlar zaten ``ORDER BY … ASC`` ile sıralanmış gibi geleceği için ekstra bir sıralama işlemine tabi tutulmasına gerek kalmaz. B-tree haricindeki indeksler ise döndürecekleri sonuç kümesi üzerinde tanımsız, uygulama özelinde değişkenlik gösterebilecek bir sıralama düzeni üretirler. Eğer tablo çok büyükse, ya da indeksli kolon geniş bir değer dağılımına sahipse ``ORDER BY`` kullanmak çok daha iyi bir performans sergileyecektir. Zira indeks, tüm tablo içinde belli bir kriteri karşılayan satırların döndürülmesi konusunda yüksek performansa sahiptir. Buna istisna oluşturacak bir durum ise ``ORDER BY … LIMIT n`` kombinasyonudur. Zira bu sorgu türü hem sıralama hem de en üst n satırı ayırma adımlarını içerir. Bu durumda indeksli kolon ORDER BY ile istenilene eşitse, indeks mekanizması ilk n satırı direk getirecek ve sıralama işlemini aradan çıkararak önemli bir performans kazancı sağlayacaktır.

Varsayılan olarak B-tree indeksler oluşturulurken indeksleme kolonundaki değerleri artan sıralamaya tabi tutar ve NULL değerleri de sıralamanın en sonuna atar. Fakat farklı ihtiyaçlar için indekslemenin varsayılan kaydı bunun tam tersi olacak şekilde de ayarlanabilir. Aşağıdaki şekilde bir indeks oluşturulduğunda B-tree indeksi sıralamanın en başına NULL değerleri koyabilir hatta *x* kolonu üzerinde azalan bir sıralamayla değer dizisi saklayabilir. Bu davranışı düzenleyebilmek için ``NULLS FIRST``, ``NULLS LAST``, ``DESC``, ``ASC`` ifadeleri kullanılır.

```sql
CREATE INDEX test2_info_nulls_low ON test2 (info NULLS FIRST);
CREATE INDEX test3_desc_index ON test3 (id DESC NULLS LAST);
```

Yine B-tree indekse özel bir durum olarak kolonda tekillik sağlaması özelliği kullanılabilir. Bunun için ``UNIQUE INDEX`` kullanılır ve bu indeks sadece B-tree indeksler için uygulanabilir. PostgreSQL, bir PRIMARY KEY tanımlandığında ya da UNIQUE CONSTRAINT yaratıldığında otomatik olarak **UNIQUE INDEX** oluşturur ve tanımlama yapılan kolona atar. Bu sebeple manuel olarak bu tür kolonlar üzerinde UNIQUE INDEX yaratmak, indeksin duplikasyonuna yol açacağından gereksizdir.

```sql
CREATE UNIQUE INDEX name ON table (column [, ...]);
```

### Hash İndeks

Bu indeks türü sadece basit eşitlik karşılaştırmalarında işe yarar. Sorgu planlayıcı, hash ile indekslenmiş bir kolonda ``=`` operatörüyle yapılmak istenen bir sorgu gördüğü yerde, bu işlem için hash indeksini kullanacaktır. Sadece **eşitlik** durumlarında kullanılacak hash indeksi aşağıdaki şekilde oluşturulur.

```sql
CREATE INDEX name ON table USING HASH (column);
```

### GiST (Generalized Search Tree: Genelleştirilmiş Arama Ağacı) İndeks

GiST indeksi, tek türde bir indeks olmayıp daha ziyade bir çok farklı indeksleme stratejisinin uygulanabileceği bir altyapı sunar. Hatta B-tree ve R-tree gibi başka indeksleme mekanizmaları da GiST içinde uygulanabilir. Bu indeksleme stratejilerinin her biri için uygun operatörler (ya da operatör grupları) vardır. Örnek olarak PostgreSQL’deki 2 boyutlu geometriler için kullanılan bazı geometri veri türleri vardır. Bu tür veriler üzerinde GiST indeksi kullanılarak ``<<``, ``&<``, ``&>``, ``>>``, ``<<|``, ``&<|``, ``|&>``, ``|>>``, ``@>``, ``<@``, ``~=``, ``&&`` gibi geometrik ilişki sorgulama operatörleriyle başarılı bir şekilde çalışması sağlanabilir.

GiST’in bir avantajı da, uygun erişim metodları seçildiğinde (çalışılan veri tipi konusunda uzman bir kullanıcının da yardımıyla) özel veri tiplerinde de indeksleme kabiliyetini geliştirebilmesidir.

{% include tip.html content="GiST indekslerinin verimli kullanımı ile ilgili ilave bilgiler internette çeşitli kaynaklarda mevcuttur [](http://gist.cs.berkeley.edu/) [](http://www.sai.msu.su/~megera/postgres/gist/) . Farklı kullanımları için **postgresql-contrib** paketinin içeriği de incelenmelidir." %}

### SP-GiST (Space Partitioned Generalized Search Trees: Disk Bölümlemeli Genelleştirilmiş Arama Ağaçları) İndeks

GiST indekslerine benzer şekilde SP-GiST (Space Partitioned Generalized Search Trees: Disk Bölümlemeli Genelleştirilmiş Arama Ağaçları) indeksleri ise, çeşitli aramaları içeren bir diğer altyapıyı sunar. SP-GiST, dörtlü ağaçlar, k-d ağaçları, radix ağaçları gibi birbirinden farklı dengesiz, disk bazlı veri yapılarının geliştirilmesine ve tasnifine imkan tanır. Örnek olarak, PostgreSQL’in standart sürümü iki boyutlu nokta veriler için ``<<``, ``>>``, ``~=``, ``<@``, ``<^``, ``>^`` operatörlerinin kullanıldığı sorgularda **SP-GiST** indeksiyle sonuç getirmeyi destekler. Bu indeks metodu, veri kümesini sürekli bölümlere ayırır ve arama kriteri SP-GiST indeksine uygun olan sorgularda çok süratli sonuç döndürebilir. SP-GiST de, aynı GiST gibi özel veri tipleri üzerinde kolayca uygulanabilir.

{% include tip.html content="Bu indeks ile ilgili çeşitli araştırmalar internette incelenebilir [](https://www.cs.purdue.edu/spgist/)." %}

### GIN İndeks

GIN indeksi de son iki indeksimize benzer şekilde farklı indeksleme stratejilerini desteklemektedir. Ayrıca PostgreSQL’in standart sürümünde GIN indeksiyle birlikte kullanılacak ``<@``, ``@>``, ``=``, ``&&`` operatörleri bulunmaktadır. Standart dağıtımla gelen bu operatörlerin yanında **postgresql-contrib** paketiyle gelen ilave çok sayıda GIN destekli operatör de vardır.

### BRIN (Block Range INdex) indeks

BRIN indeksi, indeksleme kolonundaki değer aralığı çok geniş değilse ve çizgisel bir değişim grafiği ortaya koyuyorsa çok hızlı çalışır. Lineer bir artış düzeni içeren veri tipleri üzerinde BRIN uygulandığında, her blok aralığı için minimum ve maksimum değerler depolanır. Hatta örneğin BRIN geometrik / coğrafi alanlar üzerinde uygulandığında değişim aralığı yerine geometrik dağılan nesnelerin dış çerçevesi (bounding box) saklanır. Bu sebeple BRIN indeksinin boyutu çok küçüktür.

Özel bir durum olarak kolonlara değil ifadelere de indeks oluşturulabilir. Örneğin iki kolonun birleşimine, bir kolondaki ifadelerin küçük harfe çevrilmiş haline ya da PostgreSQL’in sunduğu çeşitli hesaplama operatörlerinin kullanılarak üretilmiş veri setlerine indeks uygulamak mümkündür. Aşağıda birkaç örnek sunulmaktadır.

```sql
CREATE INDEX test1_lower_col1_idx ON test1 (lower(col1));

CREATE INDEX people_names ON people ((first_name || ' ' || last_name));
```

İndeskler hesaplanan veri setlerinde tüm kolona uygulanabileceği gibi bir kolonun (belli bir kritere tabi tutularak ayrımlanan) bir alt kümesine de (yani bazı satırlarına da) uygulanabilir. Kısmi indeksleme, özellikle çok tekrar eden değerleri olan kolonlarda, olağan satırların tekrar tekrar indekslenmesi gibi bir durumda işe yarayabilir. Eğer ilgili kolonda bazı değerler tablonun içinde diğer satırlardan çok daha fazla miktarda tekrar ediyorsa zaten indeksi teoride kullanamayacağı için indekse dahil edilmesinin anlamı da olmayacaktır. Bu sebeple bir kriter oluşturularak bu satırlar indeksin kapsamından çıkarılabilir. Böylece hem indeksin boyutu küçülür, hem de bu satırlar dışında yapılacak sorgularda indeks daha iyi çalışabilir. Performans artışı bu tabloda yapılacak UPDATE işlemlerine de yansıyacaktır. Bu gibi durumlarda tablo üzerindeki indeksin kısmi indeks olduğunun tabloda sorgu oluşturacak kişiler tarafından bilinmesi önemlidir. Aşağıda kısmi indeks oluşturulmasına bir örnek verilmiştir.

```sql
CREATE INDEX access_log_client_ip_ix ON
access_log (client_ip)
WHERE
NOT (client_ip > inet '192.168.100.0'
AND client_ip < inet '192.168.100.255');
```

{% include links.html %}

---
layout: default
title: Özyinelemi Sorgular
parent: Öne Çıkan Özellikleri
nav_order: 8
---

## Özyinelemi Sorgular (Recursive Queries)

**WITH**, kapsamlı sorgularda, sorguyu yazmayı kolaylaştıran yardımcı ifadeler yazmanın bir yoludur. Bu cümlecikler (ya da çokça ifade edildiği şekliyle Ortak Tablo İfadeleri - Common Table Expressions) sadece bir sorgu için oluşturulup kullanılıp imha olan geçici tablolar oluşturan ifadelermiş gibi düşünülebilirler. SELECT, INSERT, UPDATE ya da DELETE ile oluşturulmuş ifadeler ``WITH`` yardımcı cümleciğinin içinde olabileceği gibi WITH’le başlayan ana cümlecikler de olabilirler. Aşağıda bunların her biri için örnekler verilmiştir.

```sql
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
), top_regions AS (
    SELECT region
    FROM regional_sales
    WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)

SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

- Bu sorgu sadece en önemli satış bölgelerindeki ürün bazlı satış toplamlarını gösterir. Burada WITH cümleciği *regional_sales* ve *top_regions* isminde iki yardımcı ifade oluşturur. Bu sayede *regional_sales*'in çıktısı *top_regions* tarafından; ve *top_regions*’ın çıktısı da ana SELECT sorgusu tarafından kullanılabilir. WITH sözcüğünün kullanılmadığı alternatif bir yazım da verilmiştir. Ancak bu versiyonda iç içe iki sorgu kullanılmıştır.

Opsiyonel ``RECURSIVE`` anahtar kelimesi ise sıradan WITH sözcüğünü değiştirerek, sıradan SQL diliyle yazılması mümkün olmayan bir güç kazandırır. RECURSIVE kullanıldığında, WITH sorgusu, istenildiğinde kendi çıktısını dahi kullanabilir hale gelir. Aşağıda 1’den 100’e kadar sayıların toplamını bulan bir sorgu örneği verilmiştir.

```sql
WITH RECURSIVE t(n) AS (
    VALUES (1)
  UNION ALL
    SELECT n+1 FROM t WHERE n < 100
)
SELECT sum(n) FROM t;
```

- Bu sorgu normalde özyineleme özelliği olmayan ``WITH RECURSIVE`` ifadesinin ``UNION`` (veya UNION ALL) kullanarak kendi çıktısından faydalanabilmesine bir örnektir. Bu tür bir sorgu aşağıdaki gibi işletilir.

```sql
WITH RECURSIVE gecici_tablo(
    gecici_tablo_sorgusu -- özyinelemesiz sorgu
    UNION [ALL]
    gecici_tablo_sorgusu  -- özyinelemeli sorgu
) SELECT * FROM gecici_tablo;
```

Bütün sorgu analiz edildiğinde özyineleme (tekrarlama) kısmı ve sıradan sorgu kısmı olarak ikiye ayırdığımızda:

1. Özyinelemesiz olan kısım ilk önce çalıştırılır ve ilk baz sonuç seti elde edilir.
2. Özyinelemeli kısım çalıştırılır ve tablodaki kalan tüm satırları geçene kadar sonuç çıktıları üretilir. Bunlar geçici bir çalışma tablosuna konulur.
3. Tüm sonuçlar UNION veya UNION ALL birleştirilir. Geçici tablo boşaltılır ve silinir.

Buna daha detaylı bir örnek vermek gerekirse her çalışanın id’si, ismi ve yöneticisinin id’si kolonlarını içeren bir çalışan tablosu olduğunu varsayalım. ``WITH RECURSIVE`` ile astlar adında bir geçici tablo oluşturuluyor. Sonrasında özyinelemesiz kısımdan yönetici id’si 2 olan çalışanlar bulunuyor. Sonrasında ise bu 2 id’li çalışana bağlı çalışanlar ve onlara da bağlı çalışanlar sırayla sorgulanarak UNION ile ilk tabloya eklenir. Bu sayede 2 id’li çalışana bağlı tüm çalışanlar yöneticisi ile birlikte elde edilmiş olur.

```sql
WITH RECURSIVE astlar AS (
   SELECT
      calisan_id,
      yonetici_id,
      adi_soyadi
   FROM calisan
   WHERE calisan_id = 2
   UNION
   SELECT
      a.calisan_id,
      a.yonetici_id,
      a.adi_soyadi
   FROM calisan a
   INNER JOIN astlar b ON b.calisan_id = a.yonetici_id
) SELECT * FROM astlar;
```

Özyinelemeli sorgularla çalışırken, özyinelemeli kısmın en sonunda hiç satır döndürmemesini garanti altına almak önem arz etmektedir. Zira döngüye giren sorgu eninde sonunda işleyeceği satırları tüketmiyorsa, tanımsız bir şekilde döngülenmeye devam edecektir. Bazen bu durum gerçekleştiğinde ``UNION ALL`` yerine ``UNION`` kullanmak bu sorunu çözmeye yeterli olacaktır. UNION, önceki çıktı satırları çoklayan satırları dikkate almadan işleyeceği için döngü eninde sonunda sonlanacaktır. Fakat bazen, tablo çoklanmış satır içermez. Bu durumda bir ya da birkaç kolonda çoklanmaya başlamış aynı değerin olup olmadığını kontrol etmek sorunu anlamaya yardımcı olabilir. Bu tür durumları tahlil edebilmek için daha önce ziyaret edilmiş değerlerden oluşan bir dizi hesaplatmaktır. Örneğin aşağıda graph isimli bir tabloyu, link alanını kullanarak tarayan bir sorgu ile bu yapılmaya çalışılmıştır.

```sql
WITH RECURSIVE search_graph(id, link, data, depth) AS (
    SELECT g.id, g.link, g.data, 1
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1
    FROM graph g, search_graph sg
    WHERE g.id = sg.link
)
SELECT * FROM search_graph;
```

Bu sorgu, eğer link ilişkisinin cycle içermesi durumunda döngüye girecektir. Çünkü burada bizden istenen *depth* çıktısını sunabilmek için UNION ALL’u UNION’a çevirmek yetersiz kalacak ve bu değişiklik sonsuz döngüden kurtulmaya yetmeyecektir. Bunun yerine linkin path’ini tutan bir kolon eklenir ve aynı satıra ulaşıp ulaşılmayacağını yani cycle oluşup oluşmayacağının kontrolü yapılır. Bu sebeple path ve *cycle* isminde iki kolon ekleyerek aşağıdaki gibi sorgu oluşturulur.

```sql
WITH RECURSIVE search_graph(id, link, data, depth, path, cycle) AS (
    SELECT g.id, g.link, g.data, 1,
      ARRAY[g.id],
      false
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1,
      path || g.id,
      g.id = ANY(path)
    FROM graph g, search_graph sg
    WHERE g.id = sg.link AND NOT cycle
)
SELECT * FROM search_graph;
```

Eğer sorgumuzun döngüye girip girmediğini anlayamıyorsak bunu kontrol etmek için sonuna bir ``LIMIT`` ifadesi ekleyebiliriz. Örneğin aşağıdaki sorgu ``LIMIT 100`` ifadesi eklenmeseydi sonsuz döngüye girecekti. Bu tür bir kontrolün üretim ortamlarında denenmesi de çok tavsiye edilmez. Zira bu tür denemeler diğer sistemlerin farklı çalışmasına yol açabilir.

```sql
WITH RECURSIVE t(n) AS (
    SELECT 1
  UNION ALL
    SELECT n+1 FROM t
)
SELECT n FROM t LIMIT 100;
```

WITH ifadesi SELECT sorgularında kullanılabileceği gibi ``INSERT``, ``UPDATE`` ya da ``DELETE`` gibi verinin düzenlendiği / değiştirildiği sorgu türlerinde de kullanılabilir. Bu sayede çok sayıda veri modifikasyonu aynı sorgu içinde yerine getirilebilir. Aşağıda basit bir örnek sunulmuştur.

```sql
WITH moved_rows AS (
    DELETE FROM products
    WHERE
        "date" >= '2010-10-01' AND
        "date" < '2010-11-01'
    RETURNING *
)
INSERT INTO products_log
SELECT * FROM moved_rows;
```

- Bu sorgu sayesinde istenen tarih aralığındaki *products* tablosunda yer alan ürünlerin *products_log* tablosuna taşınması (aynı anda products’tan silinmesi ve *products_log* tablosuna yazılması) sağlanacaktır. WITH içerisindeki kısımda yer alan *products*’tan silinen satırlar *moved_rows* geçici tablosuna adım adım alınırken en sonunda topluca products_log tablosuna atılacaktır. Silinmekte olan içeriklerin geçici tabloya ve oradan da *products_log* tablosuna yazılabilmesi için ise ``RETURNING`` cümleciği kullanılmış, bu sayede içeriğin silinmeden önce okunarak iletilmesi de sağlanmıştır.

Burada dikkat edilecek noktalardan birisi WITH ifadesinin, INSERT’ün altındaki alt SELECT cümleciğine değil doğrudan INSERT’e eklenmesidir. Çünkü, veri düzenleyen ifadelerin WITH cümleciğinde sadece WITH’in en üst kademesine eklenmesine izin verilir. Fakat normal WITH görünürlük kuralları geçerli olduğundan dolayı WITH cümleciğinin çıktılarına INSERT’ün altında yer alan SELECT ifadesinden erişebilmek de mümkündür.

WITH ile kullanılan veri düzenleme ifadelerinde RETURNING ifadesi de genelde görülür. Bu RETURNING ifadesinin çıktısıdır, verisi düzenlenmiş hedef tablonun değil. Eğer bu tür veri düzenleyen bir WITH ifadesinde RETURNING yer almazsa geçici bir tablo oluşturamayacağı için sorgunun kalanına referanslanamayacak ve istenen iş gerçekleşemeyecektir. Tabi bu ifade her halükarda işletilecektir. Örneğin aşağıda buna verilen örnek çok da işlevsel değildir.

```sql
WITH t AS (
    DELETE FROM foo
)
DELETE FROM bar;
```

- Bu örnek hem *foo* hem de *bar* tablolarının tüm satırlarını siler. Fakat kullanıcıya sadece bar tablosunda silinen satırların sayısına dair geri dönüş yapılır.

{% include links.html %}

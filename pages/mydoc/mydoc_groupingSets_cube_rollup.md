---
title: "PostgreSQL Küme, Küp ve Katlama Tipi Yapılar (GROUPING SETS, CUBE, ROLLUP)"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 6, 2020
summary: "PostgreSQL GROUPING SETS, CUBE, ROLLUP"
sidebar: mydoc_sidebar
permalink: mydoc_groupingSets_cube_rollup.html
folder: mydoc
---

## Küme, Küp ve Katlama Tipi Yapılar (GROUPING SETS, CUBE, ROLLUP)

GROUP BY’dan daha karmaşık gruplama ihtiyaçları için PostgreSQL ``GROUPING SETS``, ``CUBE`` ve ``ROLLUP`` seçeneklerini sunar. `FROM` ve ``WHERE`` ifadeleriyle seçilen veriler, istenen gruplama setinin algoritmasına göre gruplanır ve aynı GROUP BY ifadelerinde olduğu gibi özet istatistikleri hesaplanabilir. Burada gruplama kümelerinin GROUP BY ifadesinden farkı, daha farklı gruplamaları GROUPING SETS, CUBE ve ROLLUP ifadelerini kullanarak yapabiliyor olmasıdır.

Aşağıdaki veri setini içeren bir tablo oluşturarak gruplama kümelerini inceleyelim.

```sql
SELECT * FROM items_sold;

 brand | size | sales
-------+------+-------
 Foo   | L    |  10
 Foo   | M    |  20
 Bar   | M    |  15
 Bar   | L    |  5
(4 rows)
```

Yukarıdaki tabloda önce *brand* sonra size kolonu ile yapılan GROUP BY gruplandırmalarına ait toplam satışlar tek tabloda görülebildiği gibi en son satırda da tüm satırlardaki satış toplamları eklenmiştir. Dolayısıyla aslında üç ayrı GROUP BY çıktı sonucunun birleştirilerek bir arada gösterilmesinden oluşan bir tablo oluşturulabilmiştir.

```sql
SELECT brand, size, sum(sales) FROM items_sold
GROUP BY GROUPING SETS ((brand), (size), ());

 brand | size | sum
-------+------+-----
 Foo   |      |  30
 Bar   |      |  20
       | L    |  15
       | M    |  35
       |      |  50
(5 rows)
```

ROLLUP ve CUBE ise GROUPING SETS’in farklı kombinasyonlarıdır. Örneğin **CUBE** tüm kombinasyonlara göre gruplama kümeleri oluştururken **ROLLUP**, hiyerarşik seviyelere göre gruplama kümeleri oluşturur. Aşağıda ROLLUP ve CUBE kullanılarak verilen örnekler bulunmaktadır. ROLLUP ve CUBE yerine GROUPING SETS kullanılarak yazıldığında ise söz diziminin ne kadar uzayabildiği her iki örneğin de altlarında gösterilmiştir.

```sql
ROLLUP ( e1, e2, e3, ... )              CUBE ( a, b, c )
GROUPING SETS (                         GROUPING SETS (
    ( e1, e2, e3, ... ),                    ( a, b, c ),
    ...                                     ( a, b    ),
    ( e1, e2 ),                             ( a,    c ),
    ( e1 ),                                 ( a       ),
    ( )                                     (    b, c ),
)                                           (    b    ),
                                            (       c ),
                                            (         )
                                        )
```

- Sol kolonda yer alan ROLLUP hemen altındaki GROUPING SETS ifadesine denk olup aynı ihtiyacı daha kısa bir yazımla yerine getirir. Özellikle hiyerarşik verilerin tutulduğu bir tabloda hiyerarşi bazlı (örneğin departman bazlı toplam maaş) gruplamalarda çok sık bir şekilde kullanılır.

- CUBE ise sağ kolonda üstte yer aldığı gibi yazılırken bir listenin tüm kombinasyonlarını verir. Bu sebeple ``CUBE (a,b,c)`` aslında altında yer alan ``GROUPING SETS (...)`` ifadesine denktir ve kısa yazılışıdır.

CUBE ve ROLLUP ifadelerini oluşturan birim elemanlar da ayrı benzer ifadelerden ya da parantez içindeki elemanların altlistelerinden oluşmuş olabilir. Özellikle ikinci durumda altlisteler daha çok bireysel gruplama kümeleri elde etmek için kullanılan tek birimler gibi işlem görürler. Örneğin;

```sql
CUBE ( (a, b), (c, d) )
```

ifadesinin muadili şu gruplama kümesidir.

```sql
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b       ),
    (       c, d ),
    (            )
)
```

Benzer şekilde;

```sql
ROLLUP ( a, (b, c), d )
```

ifadesinin de muadili şu gruplama kümesidir.

```sql
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b, c    ),
    ( a          ),
    (            )
)
```

CUBE ve ROLLUP arasındaki farklılık ROLLUP’ın kolonlar arasında bir hiyerarşi olduğu varsayımından kaynaklanır. CUBE parantez içindeki tüm kolonların farklı sayılarda kombinasyonlarını oluşturur. n elemanlı bir listede CUBE ile 2n kadar satır üretir. ROLLUP ise n eleman arasında soldan sağa bir hiyerarşi oluşturur ve her hiyerarşik seviyeden bir satır üretir. Sonunda her ikisi de SELECT ifadesinde yer alan özetleme fonksiyonunu oluşan satırlar için işletir. Fakat fark, oluşan satır gruplamalarında görülür.

{% include note.html content="CUBE ve ROLLUP ifadeleri doğrudan GROUP BY ifadesi bünyesine gömülebileceği gibi iç içe bir GROUPING SETS ifadesi içinde de yer alabilir." %}

Eğer birden fazla gruplama ifadeleri tek bir GROUP BY cümleciğinde yer alırsa sonunda elde edilecek gruplama kümesi bireysel tüm bileşenlerin çapraz çarpımı olarak döner. Örnek olarak

```sql
GROUP BY a, CUBE (b, c), GROUPING SETS ((d), (e))
```

ifadesi şuna denktir.

```sql
GROUP BY GROUPING SETS (
    (a, b, c, d), (a, b, c, e),
    (a, b, d),    (a, b, e),
    (a, c, d),    (a, c, e),
    (a, d),       (a, e)
)
```

{% include links.html %}

---
layout: default
title: View / Materialized View
parent: Öne Çıkan Özellikleri
nav_order: 4
---

## View / Materialized View

PostgreSQL’de oluşturulan bir view’da herhangi bir karmaşıklık düzeyindeki sorgu view olarak kaydedilip isimlendirilebilir. Örneğin aşağıdaki view, iki tablodan ihtiyaç duyulan kolonları getirir.

```sql
CREATE VIEW myview AS
    SELECT city, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

SELECT * FROM myview;
```

View’lar, özellikle çok kullanılan sorguların kullanıcı tarafından yeniden yazılmak zorunda kalınmadan sorgulama yapmasına yarar. View çeşitli tablolardaki çeşitli kolonlardan oluşabileceği gibi başka viewlardan da kolonlar içerebilir. Bir view, select kullanarak her çağrıldığında tanımındaki tablolarda bulunan veriyi çağrı anında yeniden sorgular.

Verilerin çok nadir değiştiği (ve kullanım amacı gereği sorgulama sonucunun dönmesi çok uzun süren) tablolardan derlenmiş view’ların daha hızlı çalışabilmesi için diske kaydedilmesi ihtiyacını karşılamak amacıyla özelleştirilmiş bir view türü olan **MATERIALIZED VIEW** bulunmaktadır. Bir materialized view, yaratılma anında bir kere sorgulanır ve her çağrılmada bu snapshot gösterilir, bu sebeple diskte saklanır, ama çok büyük sonuç tablolarının çok hızlı çağrılmasını sağlar.

```sql
CREATE MATERIALIZED VIEW mymatview AS SELECT * FROM mytab;
```

Eğer kaynak tablolardan gelen verilerde değişiklik varsa **REFRESH** edilerek yeniden derlenir ve güncellenir. REFRESH ifadesi materialized view’ın içeriğini değiştirir. Refresh işlemini sadece view’ın sahibi tarafından yapabilir.

```sql
REFRESH MATERIALIZED VIEW mymatview;
```

{% include note.html content="Bu arada, herhangi bir kullanıcı, view üzerinde refresh işlemi devam ederken bu view’a sorgu gönderirse, kaynak tablolara gidilerek materialized view’ın en güncel hali yeniden oluşturularak diske kaydedilir. Bununla birlikte güncelleme işlemi sırasında view’a erişimler view üzerinde lock oluşturularak engellenir. Refresh işlemi bittiğinde view tekrar hizmet vermeye başlar. Bunun önüne geçebilmek ve materialized view güncellenirken kullanıcıların hata almadan verileri görebilmesini sağlayabilmek için ``CONCURRENTLY`` anahtar kelimesi kullanılır. Öte yandan refresh işleminin süresi de uzar." %}

Bir materialized view üzerinde concurrently anahtar kelimesini kullanabilmek için view içindeki kolonları kullanan en az bir UNIQUE index oluşturulmuş olmalıdır. Bu index o kolondaki tüm satırları içermeli, bir ifade (expression) ya da WHERE bloğu ile süzülmüş bir veri seti üzerinde bulunmamalıdır.

Diğer bir seçenek de ``REFRESH MATERIALIZED VIEW … WITH NO DATA`` ifadesidir. Bir materialized view ilk oluşturulma anında verileri tablolardan çekerek diske yazılır. Eğer materialized view’ın oluşturulma anında diske yazılması istenmiyorsa sonuna ``WITH NO DATA`` eklenir.

```sql
REFRESH MATERIALIZED VIEW [ CONCURRENTLY ] name
    [ WITH [ NO ] DATA ]
```

{% include note.html content="Eğer belli aralıklarla Materialized View’ın güncellenmesi işi yapılabiliyorsa, sorgulama anında büyük avantaj getiren bir özellik olduğu göz önüne alınmalıdır." %}

{% include links.html %}

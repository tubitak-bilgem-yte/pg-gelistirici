---
layout: default
title: JSON
parent: Öne Çıkan Özellikleri
nav_order: 9
---

## JSON

Aşağıdaki operatörler PostgreSQL içinde JSON tipi veriler olan **json** ve **jsonb** ile kullanılmaktadır.

| Operatör | Operatörün sağındaki veri tipi | Açıklama | Örnek | Sonuç |
|-------|--------|-------|--------|--------|
| ``->`` | int | JSON dizisinden eleman alır (0 ilk indeksi, negatif elemanlar sondan geriye indeksleri gösterir) | ``'[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2`` | {"c":"baz"} |
| ``->`` | text | JSON objesini key üzerinden alır | ``'{"a": {"b":"foo"}}'::json->'a'`` | {"b":"foo"} |
| ``->>`` | int | JSON dizi elemanını text olarak alır | ``'[1,2,3]'::json->>2`` | 3 |
| ``->>`` | text | JSON’ın istenen obje alanını text olarak alır | ``'{"a":1,"b":2}' ::json->>'b'`` | 2 |
| ``#>`` | text[] | JSON objesini alır | ``'{"a": {"b":{"c":  "foo"}}}'::json#> '{a,b}'`` | {"c":"foo"} |
| ``#>>`` | text[] | JSON objesini text olarak alır | ``'{"a":[1,2,3],"b": [4,5,6]}'::json#>> '{a,2}'`` | 3 |

PostgreSQL’de kullanılan standart karşılaştırma operatörleri (``<``, ``>``, ``<=``, ``=>`` ,``=`` ,``<>`` ve ``!=``) jsonb için çalışırken json veri tiplerinde çalışmaz. Bunun sebebi json tipindeki verilerin B-Tree sıralama operasyonlarına göre dizilmesidir ve büyüklük, küçüklük, eşitlik gibi kıyaslamalara farklı reaksiyon gösterirler.

Yine aşağıdaki operatörler sadece jsonb’de çalışırlar.

| Operatör | Açıklama | Operatör | Açıklama |
|-------|--------|-------|--------|
| ``@>`` | jsonb | En üst düzeyde, soldaki JSON değeri sağdakini kapsıyor mu? | ``'{"a":1, "b":2}' ::jsonb @> '{"b":2}'::jsonb`` |
| ``<@`` | jsonb | En üst düzeyde, soldaki JSON değeri sağdaki tarafından kapsanıyor mu? | ``'{"b":2}' ::jsonb <@ '{"a":1, "b":2}' ::jsonb`` |
| ``?`` | text | String JSON’daki en üst düzeydeki anahtarın değeri mi? | ``'{"a":1, "b":2}' ::jsonb ? 'b'`` |
| ``?|`` | text[] | Dizideki stringlerden herhangi birisi en üst düzeydeki anahtar mı? | ``'{"a":1, "b":2, "c":3}’::jsonb ?| array['b', 'c']`` |
| ``?&`` | text[] | Dizideki stringlerin tümü en üst düzeydeki anahtar mı? | ``'["a", "b"]' ::jsonb ?& array['a', 'b']`` |
| ``||`` | jsonb | İki jsonb değerini birleştirip tek bir jsonb değeri üretir | ``'["a", "b"]' ::jsonb || '["c", "d"]' ::jsonb`` |
| ``-`` | text | Operatörün solunda istenen anahtar:değer çiftini veya string elemanı siler. | ``'{"a": "b"}' ::jsonb - 'a'`` |
| ``-`` | text[] | Operatörün solunda istenen bir ya da birkaç anahtar:değer çiftini veya string elemanı siler. | ``'{"a": "b", "c": "d"}'::jsonb - '{a,c}'::text[]`` |
| ``-`` | integer | Indeks numarası (konumu) verilen dizi elemanını siler. En üst seviye konteyner, dizi değilse hata döndürür. | ``'["a", "b"]' ::jsonb - 1`` |
| ``#-`` | text[] | istenen konumdaki JSON alanını veya elemanını siler. | ``'["a", {"b":1}]' ::jsonb #- '{1,b}'`` |

{% include note.html content="PostgreSQL’deki verileri kullanarak JSON objesi oluşturmak için de çok sayıda fonksiyon vardır." %}

{% include links.html %}

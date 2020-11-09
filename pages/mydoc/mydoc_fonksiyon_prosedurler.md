---
title: "Fonksiyonlar ve Prosedürler"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 9, 2020
summary: "Fonksiyonlar ve Prosedürler"
sidebar: mydoc_sidebar
permalink: mydoc_fonksiyon_prosedurler.html
folder: mydoc
---

## Fonksiyonlar ve Prosedürler

PostgreSQL, bünyesinde çok uzun zamandır bulunan fonksiyon tanımlama özelliğine ek olarak 11. sürümle birlikte PROCEDURE özelliğini de getirdi. Birbirine oldukça yakın kavramlar olan FUNCTION ve PROCEDURE kavramları sadece çıktı noktasında farklılık gösterirler. **PROCEDURE** içindeki komutları uygulayarak sonlanırken; **FUNCTION** uyguladığı komutlar için geriye bir değer döndürebilir ve bu sonuç değeri SELECT tipi sorgulara girdi olarak kullandırılabilir. Benzer şekilde prosedürler bir ifade (expression) içinde kullanılamazken fonksiyonlar sorguların, ifadelerin ya da DML komutlarının içinde kullanılabilirler. Prosedürler ise ancak `CALL` komutuyla çağrılabilirler. Fonksiyonlar transaction içermezler, sadece transaction’lar içinde kontrol noktası gibi çalışan **savepoint**’lere benzer hatalar (exceptions) atabilirler. Bir fonksiyon içinde bir transaction COMMIT edilemez, yenisi başlatılamazken yeni gelen PROCEDURE içinde transaction’lar COMMIT ve ROLLBACK yapılabilir.

Bir prosedür `CREATE (OR REPLACE) PROCEDURE` ile tanımlanır ve bu tanımlama açıkça belirtilmediği sürece içinde çalışılan şemada oluşur. Başka bir kullanıcının, yaratılmış bir prosedürü kullanabilmesi için **GRANT** ile **USAGE** yetkisinin kendisine tanımlanmış olması gerekir.

```sql
CREATE [ OR REPLACE ] PROCEDURE
    name ( [ [ argmode ] [ argname ] argtype [ { DEFAULT | = } default_expr ] [, ...] ] )
  { LANGUAGE lang_name
    | TRANSFORM { FOR TYPE type_name } [, ... ]
    | [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
    | SET configuration_parameter { TO value | = value | FROM CURRENT }
    | AS 'definition'
    | AS 'obj_file', 'link_symbol'
  }
```

Benzer sekilde fonksiyon ise `CREATE (OR REPLACE) FUNCTION` ile tanımlanır. Fonksiyon tanımında `RETURNS` ifadesi ve geri dönen çıktının veri tipi bilgisi yer alırken prosedürlerde bu olmaz.

```sql
CREATE [ OR REPLACE ] FUNCTION
    name ( [ [ argmode ] [ argname ] argtype [ { DEFAULT | = } default_expr ] [, ...] ] )
    [ RETURNS rettype
      | RETURNS TABLE ( column_name column_type [, ...] ) ]
  { LANGUAGE lang_name
    | TRANSFORM { FOR TYPE type_name } [, ... ]
    | WINDOW
    | IMMUTABLE | STABLE | VOLATILE | [ NOT ] LEAKPROOF
    | CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT
    | [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
    | PARALLEL { UNSAFE | RESTRICTED | SAFE }
    | COST execution_cost
    | ROWS result_rows
    | SET configuration_parameter { TO value | = value | FROM CURRENT }
    | AS 'definition'
    | AS 'obj_file', 'link_symbol'
  } ...
```

Fonksiyonun ismine müteakip girdi olarak kullanılacak argümanlar ve bunların veri tipi belirtilir, RETURNS ile ise çıktının veri tipi belirtilir. Prosedürün ismi, kendisiyle aynı isme ve giriş argümanına sahip bir başka prosedür veya fonksiyonla aynı olamaz. Fakat giriş argümanları değiştiği müddetçe prosedür ve fonksiyonlar aynı isimlerle kaydedilebilir ( diğer bir deyişle PostgreSQL’de fonksiyon ve prosedürlerde overloading’e izin verilmektedir ).

Eğer varolan bir prosedür `CREATE OR REPLACE PROCEDURE` ile yeniden tanımlanıyorsa, prosedürü ilk yaratan sahip ve izinler değişmez. Sadece prosedür tanımı değişir. Prosedürü hangi kullanıcı ilk defa yaratıyorsa, sahibi de o olur.

Tanımlamada kullanılacak dil olan SQL, prosedürün tanımlanması sırasında açıkça belirtilir. PostgreSQL varsayılan olarak SQL, PL/pgSQL ve C’yi desteklese de Perl, Python, TCL gibi diller PostgreSQL üzerine sonradan eklenti olarak eklenerek kullanılabilir. Tanımlama sonrasında ise prosedür tanımı başlangıçta ve bitişte yer alacak `$$` işaretleri arasına yerleştirilir. Prosedür kullanılacağı zaman ise `CALL` ile çağırılır. Aşağıda bir örnek verilmiştir.

```sql
CREATE PROCEDURE insert_data(a integer, b integer)
LANGUAGE SQL
AS $$
INSERT INTO tbl VALUES (a);
INSERT INTO tbl VALUES (b);
$$;

CALL insert_data(1, 2);
```

Fonksiyonlarda transaction’lara müdahale izni olmazken prosedürlerde transaction kontrol komutlarının tamamı kullanılabilir denmişti. Aşağıda tüm transaction komutlarının kullanıldığı bir prosedür örneği de verilmiştir. Bu örnekte iki tablo oluşturup içine veri atmaya çalışan bir prosedür tanımlanmıştır. Bu prosedür ilk oluşturduğu tabloyu COMMIT etmiş, sonrasında yarattığı ikinci tabloyu ise ROLLBACK ile diske yazmaktan vazgeçmiştir. Bu sebeple bu prosedürün çağrılması sonrasında veritabanındaki tabloların kontrolü ile sadece ilk tablonun yazıldığı ve 1 satır veri içerdiği görülebilir. Bir prosedür tanımlanarak ve çağrılarak bu sonuç elde edilebilmiştir.

Aşağıda birkaç fonksiyon örneği verilmiştir. Bunlardan ilki girdi olarak aldığı iki sayıyı toplayarak sonucunu döndürürken diğeri girdi olarak aldığı sayıyı bir arttırır. Toplama fonksiyonu SQL dilinde, artırım fonksiyonu ise plpgsql dilinde yazılmıştır.

```sql
CREATE FUNCTION add(integer, integer) RETURNS integer
    AS 'select $1 + $2;'
    LANGUAGE SQL
    IMMUTABLE
    RETURNS NULL ON NULL INPUT;
```

Toplama fonksiyonunda girdi değişkenlerine isim verilmemiş, bu sebeple fonksiyon tanımında `$1` ve `$2` olarak kullanılmıştır. Bunun yerine ikinci fonksiyonda görülebileceği gibi i isminde bir değişken adı kullanılabilir ve fonksiyon tanımının geri kalanında da bu şekilde devam edilebilirdi. Ayrıca toplama fonksiyonunda dönen çıktı değer için NULL giriş olursa NULL döndür kuralı da verilmiştir.

```sql
CREATE OR REPLACE FUNCTION increment(i integer) RETURNS integer AS $$
        BEGIN
                RETURN i + 1;
        END;
$$ LANGUAGE plpgsql;
```

Aşağıdaki fonksiyonlarda ise birden fazla değer döndürülmektedir. İlk fonksiyon hemen altında alternatif olarak yeniden yazılmıştır.

```sql
CREATE FUNCTION dup(in int, out f1 int, out f2 text)
    AS $$ SELECT $1, CAST($1 AS text) || ' is text' $$
    LANGUAGE SQL;

SELECT * FROM dup(42);
```

Bu fonksiyonun alternatif hali de şöyle olabilir.

```sql
CREATE TYPE dup_result AS (f1 int, f2 text);

CREATE FUNCTION dup(int) RETURNS dup_result
    AS $$ SELECT $1, CAST($1 AS text) || ' is text' $$
    LANGUAGE SQL;

SELECT * FROM dup(42);
```

İkinci fonksiyonda çıktılar `RETURNS TABLE` kullanımı sayesinde döndürülmüştür. RETURNS TABLE kullanıldığında bir değer değil bir satır dizisi döndürülebilir.

```sql
CREATE FUNCTION dup(int) RETURNS TABLE(f1 int, f2 text)
    AS $$ SELECT $1, CAST($1 AS text) || ' is text' $$
    LANGUAGE SQL;

SELECT * FROM dup(42);
```

Eğer oluşturulmuş bir prosedür üzerinde `ALTER PROCEDURE` kullanarak değişiklik yapılmak isteniyorsa prosedürün sahibi olmak gerekmektedir. Eğer prosedürün şemasını değiştirmek istenirse söz konusu prosedür üzerinde CREATE hakkına da sahip olmak gerekir. Prosedürün ismini değiştirmek için `RENAME` anahtar kelimesi kullanılabilir. Prosedür için geçerli olan ALTER uygulaması fonksiyon için de `ALTER FUNCTION` şeklinde geçerlidir.

```sql
ALTER PROCEDURE insert_data(integer, integer) RENAME TO insert_record;
```

Bir *superuser*, herhangi bir prosedürün sahipliğini ya da bulunduğu şemayı değiştirebilir. Bu sayede prosedür üzerinde çok sayıda değişiklik yapmak da mümkün hale gelebilir. Bunun için şöyle bir atama kullanılmalıdır.

```sql
ALTER PROCEDURE insert_data(integer, integer) OWNER TO teoman SET SCHEMA bilgi_islem;
```

{% include note.html content="Fonksiyon ve prosedürler de DROP FUNCTION / DROP PROCEDURE şeklinde silinebilirler." %}

{% include links.html %}

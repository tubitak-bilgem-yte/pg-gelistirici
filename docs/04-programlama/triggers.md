---
layout: default
title: Tetikleyiciler
parent: Veritabanı Programlama
nav_order: 3
---

## Tetikleyiciler (Triggers)

Tetikleyiciler, veritabanlarında belli olayların gerçekleşmesi halinde çalışması istenen, veritabanının daha önceden belirlenmiş bir fonksiyonu otomatik işletme uygulamasıdır. Tetikleyiciler tanımlandığında, çalışacakları olay ve bu olay gerçekleştiğinde çalıştırılacak fonksiyon bilgisiyle birlikte tanımlanırlar. Dolayısıyla bir tetikleyici tanımlanmadan hemen önce fonksiyonunun tanımlanmış olması gerekmektedir. Trigger’da kullanılacak fonksiyon da PL/pgSQL, PL/Tcl, PL/Perl, PL/Python gibi çok sayıda dilden herhangi birisinde yazılmış olabilir.

- Bir tetikleyici (trigger) oluşturulmadan önce tetikleyicinin fonksiyonu oluşturulmalıdır. Tetikleyici fonksiyonları tanımında argüman olmamalı ve dönen çıktı veri tipi trigger olmalıdır. Tetikleyici fonksiyonunda giriş argümanı tanımlanmamasının sebebi ise, fonksiyonun giriş argümanını **TriggerData** yapısı denilen özel bir şekilde kendisine iletilen veriyi kullanarak almasıdır. TriggerData türü argümanlar, sıradan fonksiyon argümanlarından farklıdır. Fonksiyon tanımından sonra ise ``CREATE TRIGGER`` argümanı kullanılarak tetikleyicinin tanımlanması sağlanır. Aynı fonksiyon birden fazla tetikleyici için kullanılabilir.

- PostgreSQL’de hem satır bazlı işletilebilen hem de ifade / cümle bazlı işletilebilen tetikleyicilere destek verilmektedir. Satır bazlı tetikleyiciyle, tetikleyici, ifadeden etkilenen her satır için birer kere işletilir. Bunun aksine cümle / ifade bazlı işleyen tetikleyiciler, cümlenin uygulanabildiği satırlar için bir kere işletilir. Burada tetikleyicinin uygulanacağı satır sayısı değil, tetikleyicinin sadece bir kere işletilmesi durumu önemlidir.

- Tetikleyiciler için önemli bir kullanım alanı olarak ilişkili tablolar gösterilebilir. Data manipülasyonu türünde bir operasyonunda (INSERT, UPDATE, DELETE), işlem gerçekleşmeden önce veya gerçekleştikten sonra işlemeye başlayacak ve düzenlenen satır bazında ya da SQL cümleciğinin tamamında çalışacak bir tetikleyici oluşturabiliriz. Hatta UPDATE tetikleyicileri, sorgudaki SET’e karşılık gelen satırlar bazında dahi çalışabilir. Tetikleyiciler, belli şartların gerçekleşmesi öncesinde/sonrasında, TRUNCATE komutunu kullanmakta da kullanılabilir.

- Tetikleyiciler view’lardaki INSERT, UPDATE veya DELETE operasyonları yerine işletilmek için de tanımlanabilir. Bu tür INSTEAD OF tetikleyicileri view içinde düzenlenmesi gereken her satır için bir kere çalıştırılabilir. View’a veri sağlayan tablolardaki verinin güncellenmesi ise tetikleyici fonksiyonun sorumluluğundadır. Önemli bir not olarak belirtmek gerekirse, ifadeden etkilenen satır sayısı sıfır satır dahi olsa, ifade bazında tanımlanan tetikleyici işletilecektir. TRUNCATE fonksiyonunu kullanacak tetikleyiciler sadece ifade bazında çalıştırılabilir, satır bazında işlem yapamaz.

- Tetikleyicileri işletilme anına göre **öncesinde** (BEFORE), **sonrasında** (AFTER), **yerine** (INSTEAD OF) işletilme şeklinde sınıflandırmak mümkündür. BEFORE ve AFTER türündeki tetikleyicilerin isimleri aslında zamanlama olarak ne şekilde çalıştıklarını açıklar. Fakat öte yandan bu tür tetikleyicilerin sadece partitioning yapılmamış tablolarda ve hem satır hem de  ifade / cümle seviyesinde uygulanabildiği bilinmelidir. View’larda ve partitioning uygulanmış tablolarda çalışmayacaklardır. Öte yandan INSTEAD OF tipi tetikleyiciler sadece view’larda ve sadece satır seviyesinde çalışabilirler. Bir view’ın işlenmesi gereken satırları tespit edilir edilmez INSTEAD OF türü tetikleyiciler çalıştırılır.

- Tetikleyici fonksiyon tanımı içinde INSERT veya UPDATE cümleciğiyle eklenmesi / güncellenmesi istenen satırlar ``NEW``, ``DELETE`` cümleciğiyle silinmesi istenen satırlar ise ``OLD`` olarak çağırılır. Ayrıca INSERT ve UPDATE tipi tetikleyiciler NEW satırını düzenledikten sonra döndürebilir. Bu durum, ``INSERT RETURNING`` veya ``UPDATE RETURNING`` kullanarak geri döndürülen satırı değiştirecektir. Bu tanımlar satır bazlı tetikleyicilerde geçerliyken, ifade bazlı tetikleyicilerde kullanım alanları yoktur.

- Eğer aynı olay anı için birden fazla tetikleyici ayarlandıysa, tüm tetikleyiciler isimlerine göre alfabetik sırayla çalıştırılacaktır. BEFORE ve INSTEAD OF tipi tetikleyicilerde ise her tetikleyicinin döndürdüğü değiştirilmiş satırın bir sonrakinin girdisi olması muhtemeldir. Eğer bir şekilde BEFORE veya INSTEAD OF tetikleyicileri NULL değer döndürürse tüm operasyon duracağı için (çalışılan tablonun bu satırı özelinde) sonraki tetikleyiciler de çalışmayacaktır.

- Eğer bir tetikleyici fonksiyon içinde SQL cümlecikleri varsa, bu cümleciğin işletilmesi sonucu bir başka tetikleyici harekete geçebilir. Bu türde çalışan tetikleyicilere ardışık tetikleyici (cascading triggers) denir. Ardarda işletilebilecek tetikleme kademesi üzerinde herhangi bir sınırlama olmadığı için bu tür tetikleyiciler sınırsız bir şekilde kurulabilir / çalışabilir. Hatta bu şekilde bir ardışık tetikleme mekanizmasının içinde bulunduğu tetikleyiciyi de ardarda tetiklemesi mümkündür. Örnek vermek gerekirse bir INSERT tetikleyicisi, aynı tabloya yeni bir satır ekleme komutunu içeriyor olabilir. Tetikleyiciyi kodlayan programcının (koddaki şartları) gözden kaçırması sonucu, tetikleyici bir kere tabloya satır eklemeye başladığında sonsuz döngüye girerek tabloya sürekli yeni satırlar ekleyebilir. Bu tamamen tetikleyiciyi programlayan kişinin sorumluluğundadır.

- Varsayılan olarak ifade bazlı tetikleyicilerin ifade tarafından düzenlenen bireysel satırları araştırma kabiliyeti yoktur. Fakat bir ``AFTER STATEMENT tetikleyicisi, etkilenen satırlardan oluşan geçiş tabloları oluşturulmasını isteyebilir. AFTER ROW tetikleyicileri de bu tür geçiş tabloları oluşturabilirler. Bu sayede **AFTER ROW** türü tetikleyiciler de hem tablodaki toplam değişikliği hem işletilmelerine yol açan satırdaki değişikliği görebilir. Bu tür geçiş tablolarını incelemekte kullanılan metod, programlamada kullanılan dile bağlı olarak değişkenlik gösterir. Fakat genel uygulama, geçiş tablolarının tetikleyici fonksiyonu içerisinden SQL komutlarının erişebildiği sadece okunur (read-only) tablolar olduğu varsayımı ile hareket etmektir.  

- Tetikleyici fonksiyonları içinde SQL komutları kullanabiliriz. Bu komutlar ise tetikleyicinin varlık sebebi olan tabloya erişmektedirler. Bizim bu noktada göz önüne almamız gereken en önemli nokta SQL komutlarının hangi satırlara erişebileceği, hangi satırların çalışma anında tetikleyiciye *görünür* olduğudur. Kısaca bu durumları özetlemek gerekirse:

  - İfade bazlı tetikleyiciler basit görünürlük kuralları izlerler: bir SQL cümleciğinin sebep olduğu değişikliklerin hiçbiri BEFORE tetikleyicisine görünmezken, tamamı AFTER tetikleyicisine görünür durumdadır.
  - Satır düzeyinde BEFORE türü bir tetikleyicide, tetikleyicinin işletilmesine yol açan veri değişikliği (INSERT - UPDATE - DELETE) SQL komutu tarafından görünür değildir, çünkü değişiklik henüz gerçekleşmemiştir.
  - Fakat satır düzeyinde bir BEFORE tetikleyici fonksiyonundaki SQL komutları, bir adım öncesinde işlenmiş satır değişikliklerinin etkilerini     görecektir. Komutların yol açabileceği değişiklikler genel olarak tahmin edilemez olduğu için bu tür değişikliklere dikkatle yaklaşmak gerekmektedir.   Birden fazla satırı etkileyen SQL komutları satırları rastgele bir düzende ziyaret edebilir.
  - Benzer şekilde satır düzeyinde bir INSTEAD OF tetikleyicisi bir üst düzeydeki komutun INSTEAD OF tetikleyicisinin sebep olduğu değişiklikleri      görecektir.
  - Satır düzeyinde bir AFTER tetikleyicisi işletildiğinde ise dış komut düzeyinde zaten tamamlanmış olan tüm veri değişiklikleri, bahsi geçen iç      tetikleyici fonksiyonda görünürdür.

- Eğer tetikleyici fonksiyonumuz standart prosedürel dillerden herhangi biriyle yazılmışsa, yukarıda sayılan maddeler sadece fonksiyon VOLATILE olarak deklare edildiğinde işleyecektir. STABLE ya da IMMUTABLE olarak deklare edilmiş fonksiyonlar ise hiçbir durumda çağıran komut tarafından yapılmış değişiklikleri görmeyecektir.

Aşağıda veri değişiklikleri ile ilgili yazılmış bir tetikleyici örneği bulunmaktadır. Bu tetikleyicinin yazımında PL/pgSQL dili kullanılmıştır. Tetikleyici fonksiyonun tanımlanması için CREATE FUNCTION komutu, hiçbir argüman almayacak ve (veri değişikliklerinde işleyen tetikleyiciler için) trigger ya da (olay bazlı tetikleyiciler için) event_trigger tipinde çıktı döndürecek şekilde  kullanılmıştır. **TG_** ile başlayan özel yerel değişkenlerse, çağrıyı tetikleyen şartları belirlemek amacıyla otomatik olarak tanımlanmaktadır.

Eğer bir PL/pgSQL fonksiyonu tetikleyici tarafından çağrılırsa aşağıdaki değişkenler otomatik olarak en üst seviyede tanımlanırlar:

- **NEW**: RECORD türünde bir değişken olup veritabanındaki INSERT / UPDATE ile gelen yeni satırı tutarak satır bazlı tetikleyicilerde kullanılabilmesini sağlar. Bu değişkenin değeri DELETE işlemlerinde ve ifade bazlı tetikleyicilerde NULL’dur.
- **OLD**: RECORD türünde bir değişken olup veritabanındaki DELETE ile silinen satırı tutarak satır bazlı tetikleyicilerde kullanılabilmesini sağlar. Bu değişkenin değeri INSERT işlemlerinde ve ifade bazlı tetikleyicilerde NULL’dur.
- **TG_NAME**: Veri tipi name’dir. Bu değişken işletilmekte olan tetikleyicinin ismini tutar.
- **TG_WHEN**: Veri tipi text’tir. Tetikleyici tanımına bağlı olarak `BEFORE`, ``AFTER`` veya `INSTEAD OF` ifadelerini tutar.
- **TG_LEVEL**: Veri tipi text’tir. Tetikleyici tanımına bağlı olarak `ROW` veya `STATEMENT` ifadelerini tutar.
- **TG_OP**: Veri tipi text’tir. Tetikleyicinin işlemeye başladığı operasyona bağlı olarak `INSERT`, `UPDATE`, `DELETE` veya `TRUNCATE` ifadelerini tutar.
- **TG_TABLE_NAME**: Veri tipi name’dir. Tetikleyicinin çağrılmasına yol açan tablonun adını tutar. Eskiden kullanılan **TG_RELNAME** değişkeninin yerini almıştır.
- **TG_TABLE_SCHEMA**: Veri tipi name’dir. Tetikleyicinin çağrılmasına yol açan tablonun şemasının adını tutar.
- **TG_NARGS**: Veri tipi integer’dır. ``CREATE TRIGGER`` cümleciği içinde tetikleyici fonksiyonua girdi olarak tanımlanan argüman sayısıdır.
- **TG_ARGV [ ]**: Veri tipi array of text(metin dizisi)’dir. `CREATE TRIGGER` cümlesinde tanımlanan argümanları tutar. Dizi indeksi 0’dan başlar. Geçersiz (indeksi 0’dan küçük, TG_NARGS’a eşit veya daha büyük) indeksler NULL değer döndürür.

Aşağıdaki gibi bir tablomuz olsun.

```sql
CREATE TABLE emp (
    empname text,
    salary integer,
    last_date timestamp,
    last_user text
);
```

Oluşturacağımız tetikleyici (trigger) tabloda ne zaman bir INSERT ya da UPDATE işlemi yapılsa kullanıcının adını, gerçek adını ve o anki zaman bilgisini ayrı bir tabloya ekliyor. Ayrıca bu işçinin adını da kontrol edip maaşının pozitif değerde olduğunu doğruluyor. Tabi önce tetikleyici fonksiyonu oluşturuyoruz.

```sql
CREATE FUNCTION emp_stamp() RETURNS trigger AS $emp_stamp$
    BEGIN
-- INSERT veya UPDATE yaparken empname
-- ve salary kolonları dolu olmalı, değilse hata verecek
        IF NEW.empname IS NULL THEN
            RAISE EXCEPTION 'empname cannot be null';
        END IF;
        IF NEW.salary IS NULL THEN
            RAISE EXCEPTION '% cannot have null salary', NEW.empname;
        END IF;
        -- Maaş kolonuna girilen değer pozitif olmalı
        IF NEW.salary < 0 THEN
            RAISE EXCEPTION '% cannot have a negative salary', NEW.empname;
        END IF;
-- last_date ve last_user kolonlarını otomatik doldurur
-- ve tabloya yazdığı tüm satırı gösterir.
        NEW.last_date := current_timestamp;
        NEW.last_user := current_user;
        RETURN NEW;
    END;
$emp_stamp$ LANGUAGE plpgsql;
```

Sonrasında ise ``CREATE TRIGGER`` ile tetikleyicinin tanımını yapıyor ve oluşturduğumuz fonksiyonu bu tetikleyiciye atıyoruz.

```sql
CREATE TRIGGER emp_stamp BEFORE INSERT OR UPDATE ON emp
    FOR EACH ROW EXECUTE FUNCTION emp_stamp();
```

Tabloda kontrol yapmak yerine loglama yapmak için de aşağıdaki gibi bir yol izleyebiliriz. Bu sefer tabloda yapılan herhangi bir INSERT, UPDATE veya DELETE operasyonunu kaydedeceğimiz aşağıdaki tablo yapısını kuralım.

```sql
CREATE TABLE emp (
    empname           text NOT NULL,
    salary            integer
);

CREATE TABLE emp_audit(
    operation         char(1)   NOT NULL,
    stamp             timestamp NOT NULL,
    userid            text      NOT NULL,
    empname           text      NOT NULL,
    salary integer
);
```

Yukarıdaki yapıda emp tablosuna yapılacak INSERT, UPDATE ya da DELETE işlemlerinde *emp_audit* tablosuna yapılan işlemin türü ve değişiklik girilecek. Bu sayede tablo üzerinde *t=0* anından itibaren yapılan tüm değişikliklerin bir kaydı ayrı bir tabloda tutulmuş olacak.

```sql
CREATE OR REPLACE FUNCTION process_emp_audit() RETURNS TRIGGER AS $emp_audit$
    BEGIN
-- emp_audit tablosunda emp tablosundaki değişikliği
-- yansıtacak yeni bir satır oluşturmak için TG_OP
-- değişkenini kullanıyoruz.
        IF (TG_OP = 'DELETE') THEN
            INSERT INTO emp_audit SELECT 'D', now(), user, OLD.*;
        ELSIF (TG_OP = 'UPDATE') THEN
            INSERT INTO emp_audit SELECT 'U', now(), user, NEW.*;
        ELSIF (TG_OP = 'INSERT') THEN
            INSERT INTO emp_audit SELECT 'I', now(), user, NEW.*;
        END IF;
        RETURN NULL;
-- bu bir AFTER tetikleyicisi olduğu için sonucu döndürmemize
-- gerek yok zira amacımız bir denetim tablosu oluşturmak
    END;
$emp_audit$ LANGUAGE plpgsql;

CREATE TRIGGER emp_audit
AFTER INSERT OR UPDATE OR DELETE ON emp
    FOR EACH ROW EXECUTE FUNCTION process_emp_audit();
```

Bazen tetikleyiciler sayesinde bir tablodan özet bir başka tablo da elde edebiliriz. Belli sorgular için ana sorgu yerine otomatik oluşturulan bu özet tabloyu kullanabiliriz. Kapsamlı bir tablo yerine elde edilmiş bu özet tabloyu sorgulamak bize zamansal kazanç dahi sağlayabilir. Data Warehousing olarak adlandırılan bu teknik daha çok ölçüm sonuçlarından oluşan aşırı büyük tablolar üzerinde uygulanmaktadır.

Aşağıdaki örnekte zamana bağlı satış bilgisinin saklandığı bir veritabanı örneklenmiştir. Bu ortamda *sales_summary_bytime* tablosu *time_dimension* ve *sales_fact* tablolarından elde edilmiş özet bilginin saklandığı özet tablodur.

```sql
CREATE TABLE time_dimension (
    time_key                    integer NOT NULL,
    day_of_week                 integer NOT NULL,
    day_of_month                integer NOT NULL,
    month                       integer NOT NULL,
    quarter                     integer NOT NULL,
    year                        integer NOT NULL
);
CREATE UNIQUE INDEX time_dimension_key ON time_dimension(time_key);

CREATE TABLE sales_fact (
    time_key                    integer NOT NULL,
    product_key                 integer NOT NULL,
    store_key                   integer NOT NULL,
    amount_sold                 numeric(12,2) NOT NULL,
    units_sold                  integer NOT NULL,
    amount_cost                 numeric(12,2) NOT NULL
);
CREATE INDEX sales_fact_time ON sales_fact(time_key);

CREATE TABLE sales_summary_bytime (
    time_key                    integer NOT NULL,
    amount_sold                 numeric(15,2) NOT NULL,
    units_sold                  numeric(12) NOT NULL,
    amount_cost                 numeric(15,2) NOT NULL
);
CREATE UNIQUE INDEX sales_summary_bytime_key ON sales_summary_bytime(time_key);
```

Bu tabloları esas alarak öncelikle tetikleyici fonksiyonumuzu oluşturuyoruz.

```sql
CREATE OR REPLACE FUNCTION maint_sales_summary_bytime() RETURNS TRIGGER
AS $maint_sales_summary_bytime$
    DECLARE
        delta_time_key          integer;
        delta_amount_sold       numeric(15,2);
        delta_units_sold        numeric(12);
        delta_amount_cost       numeric(15,2);
    BEGIN
        IF (TG_OP = 'DELETE') THEN

            delta_time_key = OLD.time_key;
            delta_amount_sold = -1 * OLD.amount_sold;
            delta_units_sold = -1 * OLD.units_sold;
            delta_amount_cost = -1 * OLD.amount_cost;
        ELSIF (TG_OP = 'UPDATE') THEN
            IF ( OLD.time_key != NEW.time_key) THEN
                RAISE EXCEPTION 'Update of time_key : % -> % not allowed',OLD.time_key, NEW.time_key;
            END IF;
            delta_time_key = OLD.time_key;
            delta_amount_sold = NEW.amount_sold - OLD.amount_sold;
            delta_units_sold = NEW.units_sold - OLD.units_sold;
            delta_amount_cost = NEW.amount_cost - OLD.amount_cost;

        ELSIF (TG_OP = 'INSERT') THEN
            delta_time_key = NEW.time_key;
            delta_amount_sold = NEW.amount_sold;
            delta_units_sold = NEW.units_sold;
            delta_amount_cost = NEW.amount_cost;

        END IF;

        LOOP
            UPDATE sales_summary_bytime
                SET amount_sold = amount_sold + delta_amount_sold,
                    units_sold = units_sold + delta_units_sold,
                    amount_cost = amount_cost + delta_amount_cost
                WHERE time_key = delta_time_key;

            EXIT insert_update WHEN found;

            BEGIN
                INSERT INTO sales_summary_bytime (
                            time_key,
                            amount_sold,
                            units_sold,
                            amount_cost)
                    VALUES (
                            delta_time_key,
                            delta_amount_sold,
                            delta_units_sold,
                            delta_amount_cost
                           );

                EXIT insert_update;

            EXCEPTION
                WHEN UNIQUE_VIOLATION THEN
            END;
        END LOOP insert_update;

        RETURN NULL;

    END;
$maint_sales_summary_bytime$ LANGUAGE plpgsql;

CREATE TRIGGER maint_sales_summary_bytime
AFTER INSERT OR UPDATE OR DELETE ON sales_fact
    FOR EACH ROW EXECUTE FUNCTION maint_sales_summary_bytime();
```

Aşağıdaki satırları *sales_fact* tablosuna atarak tetikleyicinin nasıl çalıştığını gözlemleyebiliriz.

```sql
INSERT INTO sales_fact VALUES(1,1,1,10,3,15);
INSERT INTO sales_fact VALUES(1,2,1,20,5,35);
INSERT INTO sales_fact VALUES(2,2,1,40,15,135);
INSERT INTO sales_fact VALUES(2,3,1,10,1,13);
SELECT * FROM sales_summary_bytime;
DELETE FROM sales_fact WHERE product_key = 1;
SELECT * FROM sales_summary_bytime;
UPDATE sales_fact SET units_sold = units_sold * 2;
SELECT * FROM sales_summary_bytime;
```

{% include links.html %}

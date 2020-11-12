---
title: "PostgreSQL Transaction Yönetimi"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 6, 2020
summary: "PostgreSQL Transaction Yönetimi"
sidebar: mydoc_sidebar
permalink: mydoc_transaction_yonetimi.html
folder: mydoc
---

## PostgreSQL Transaction Yönetimi

Transaction’lar veritabanı yönetimindeki temel kavramlardan birisidir. Buradaki temel nokta, transaction’ların birden fazla iş adımını paketlemesi ve işlemin kesintiye uğraması durumunda **ya hep ya hiç** mantığı güderek, yapılan adımları da geri alabilmesidir. Öte yandan, transaction adımları arasındaki ara haller diğer eş zamanlı transaction’lar tarafından görünmezdir ve eğer bir problem yaşanır da transaction’ın tamamlanmasını engelleyen durumlar ortaya çıkarsa önceki adımların hiçbiri veritabanına uygulanmaksızın geri alınır.

Örneğin çok sayıda müşterisinin hesabının tutulduğu bir banka veritabanını düşünelim. Bu veritabanında müşterilerin olduğu kadar banka şubelerinin de bütçeleri saklanmakta. Varsayalım ki A müşterisinin hesabından B müşterisinin hesabına 100$ ödeme kaydedeceğiz. En basit haliyle aşağıdaki gibi bir dizi sorgu yazmamız gerekir.

```sql
UPDATE hesaplar SET bakiye = bakiye - 100.00
    WHERE isim = 'A';
UPDATE subeler SET bakiye = bakiye - 100.00
    WHERE isim = (SELECT sube_adi FROM hesaplar WHERE isim = 'A');
UPDATE hesaplar SET bakiye = bakiye + 100.00
    WHERE isim = 'B';
UPDATE subeler SET bakiye = bakiye + 100.00
    WHERE isim = (SELECT sube_adi FROM hesaplar WHERE isim = 'B');
```

Bu kadar basit bir işlem için dahi bir dizi UPDATE komutu uygulamamız gerekmektedir. Bankanın görevi bu yukarıdaki UPDATE’lerin tamamının yapıldığından emin olmaktır. Eğer hepsi gerçekleşmeyecekse hiç biri de gerçekleşmemelidir. Zira istenen şey B’ye ödenen 100$’ın A’dan alınmamış olması değildir. Aynı şekilde A’dan 100$ kesilip B’ye ödenmezse bu da bir başka problem olacaktır. Bu sebeple herhangi bir sorun ortaya çıkarsa, birşeyler ters giderse yapılan işlemlerin geri alınacağına dair bir garantimizin olması gerekir. Bir sorun oluşması ve tüm adımların tamamlanamaması halinde yapılmaya başlanan adımlar geri alınmalıdır. Bu yüzden yapılmak istenen tüm bu güncelleme adımlarının gruplanarak bir transaction içinde saklanması bize bu garantiyi sunar. Transaction bir bütündür. Yani diğer transactionlar açısından bakıldığında her transaction tüm öğeleriyle tek parçadır, ya olur ya olmaz.

İhtiyaç duyduğumuz bir diğer garanti ise transaction tamamlandığında ve veritabanı tarafından tanındığında kalıcı olarak kaydedilmiş olmalıdır ve sonrasında bir çökme durumu yaşansa bile transaction’la yapılmak istenen değişiklik kaydedilmiş olmalıdır. B müşterisi hesabından para çekse, ve bir şekilde veritabanında bir çökme yaşansa B’nin hesabından çekilen miktarın düşüldüğü bilgisinin de veritabanına kaydedilmiş olmasını bekleriz. Transaction mekanizmasına sahip veritabanlarında bir transaction bünyesinde yapılan tüm veri güncellemelerinin, **transaction tamamlandı** bilgisi kullanıcıya bildirilmeden hemen önce kalıcı bir depoya (örneğin diske) kaydedildiği taahhüt edilir.

Transaction mekanizmasına sahip veritabanlarındaki bir diğer önemli özellik ise aynı anda işlemekte olan ve henüz tamamlanmamış transactionlardan hiçbirisinin kendisi dışındaki tamamlanmamış transaction’ları göremiyor olması gerekliliğidir. Bu sebeple transaction’lar sadece ya hep ya hiç mantığıyla diske yazılmakla kalmamalı, aynı zamanda işler haldeki tamamlanmamış transaction’lar birbirine görünmez de olmalıdır. Ne zaman bir transaction tamamlanarak diske yazılır, bu andan itibaren başlayan transaction’lar tarafından görünür hale gelmelidir.

Transaction paketi ``BEGIN`` ve ``COMMIT`` ifadeleri arasında yazılır. Bu komutların arasında kalan ifadelere transaction bloğu da denilir. Blok içinde birçok komut ardarda yazılabilir. Eğer COMMIT ifadesine başarıyla ulaşılabilirse bütün komutlar başarıyla veritabanına yazılmış olur. Eğer az önce de bahsedildiği gibi bir sorun yaşanırsa blok işletilmez.

Bir transaction bloğu içindeki önemli köşe noktaları için ``SAVEPOINT`` oluşturulabilir. Belli şartlar sağlandığında SAVEPOINT’e dönülmek üzere ``ROLLBACK`` komutu verilirse tanımlanmış bu SAVEPOINT’e kadarki adımlar her halükarda diske işlenmiş olacaktır. SAVEPOINT komutu transaction bloğu içindeki ara bir COMMIT noktası gibi çalışır. Örneğin A’dan B’ye transfer edilmek istenen 100$’ı düşünelim. A bu transferi göndermek isterken hesabında 100$ yoksa bu işlemin kesilmesi ve o ana kadar yapılmış tüm UPDATE işlemlerinin geri alınması gerekecektir. Bu sebeple COMMIT yerine ROLLBACK bu noktada uygulanmalıdır.

PostgreSQL aslında tüm SQL cümlelerini (biz açıkça yazmasak dahi) BEGIN ve COMMIT ile sarmalayarak işletir. Yani biz sorgunun başına BEGIN koymasak dahi koyar ve sonuna COMMIT ekleyerek çalıştırır. Aşağıda BEGIN, COMMIT, SAVEPOINT ve ROLLBACK kullanılmış bir transaction örneği bulunmaktadır. A’nın C’ye ödeme yapması istenirken B’nin hesabının güncellenmesi gibi bir hata yaşandığı için zamanında oluşturulmuş *my_savepoint*'e ROLLBACK ile geri dönülmüş ve nihayetinde C’ye ödenecek para C’nin hesabında güncelleme yapılarak ödenmiştir.

```sql
BEGIN;
UPDATE hesaplar SET bakiye = bakiye - 100.00
    WHERE isim = ‘A’;
SAVEPOINT my_savepoint;
UPDATE hesaplar SET bakiye = bakiye + 100.00
    WHERE isim = 'B';
ROLLBACK TO my_savepoint;
UPDATE hesaplar SET bakiye = bakiye + 100.00
    WHERE isim = 'C';
COMMIT;
```

Transaction kavramına ilave olarak transaction modu diye bir kavram da vardır. Her transaction, ayarlı olduğu modda işletilmekte ve bu modlar ``SET TRANSACTION`` komutuyla değiştirilebilmektedir. Bu komut kullanıldığında sadece içinde bulunulan transaction etkilenmekte, bu transaction dışındaki transaction’lar için değişen herhangi bir durum olmamaktadır. Bunun yapılabilmesi istendiğinde ise ``SET SESSION CHARACTERISTICS`` komutu kullanılmalıdır. Bu sayede komut sonrasında oturum boyunca başlatılan tüm transactionlar için mod karakteristiği değişecektir.

Bir transaction’ın karakteristik özellikleri arasında transaction izolasyon düzeyi, transaction erişim modu (okuma, yazma ve okuma / yazma) ve ertelenebilir modun değiştirilmesine izin verilmektedir. Ek olarak veritabanındaki varolan durum da, (oturum varsayılanı olarak değil de) sadece içinde bulunulan transaction için seçilebilir.

{% include note.html content="Bir transaction için izolasyon düzeyi, bu transaction’ın aynı anda çalışan diğerleri tarafından üretilen hangi verileri görebileceği konusunu belirler." %}

**SERIALIZABLE**: En katı transaction izolasyon seviyesidir. Bu izolasyon seviyesinde commit edilen tüm transactionlar için seri transaction düzeyi (ardışıklaştırma) varmış gibi davranılır; yani eş zamanlı transaction’ların uygulanması aynı anda yapılmaz, birbiri ardına sırayla gönderilmişler gibi gerçekleşir. Serializable izolasyon düzeyinin mantığı transactionların işlenmesinin tamamen kontrol edildiği ve böylece hataya sebep olabilecek tüm ihtimallerin ortadan kaldırıldığı bir veritabanı ortamının yaratılması üzerinedir. Aynı Repeatable Read düzeyi gibi, bu izolasyon düzeyini kullanan uygulamalar da transaction’larını ardışıklaştırma (serialization) hataları yüzünden yeniden işletmeye hazırlıklı olmalıdır. Bu durum, süreç doğru yönetilmezse, “eş zamanlı işleyemeyeceği” için hata üreten çok sayıda transaction’la sonuçlanabileceği gibi performans olarak yavaşlamaya yol açabilir.

**REPEATABLE READ**: Bu izolasyon düzeyi sadece transaction başlamadan önce commit edilmiş verileri görebilir. Commit edilmemiş, ya da transaction’ın işletilmesi sırasında eş zamanlı transaction’lar tarafından commit edilen değişiklikler bu izolasyon düzeyinde görünmezdir. Bununla birlikte, sorgu, kendi transaction paketi içinde önceden yapılmış (fakat hala commit edilmemiş) değişikleri görerek işlemini sürdürür. Bu, SQL standardının bu seviye için gerektirdiğinden daha zorlu bir güçlü bir garanti sunarken ardışıklaştırma anomalileri haricindeki tüm diğer fenomenlerin (dirty read, tekrarlanamaz okuma, gölge okuma) gerçekleşmesinin de önüne geçer. Repeatable Read izolasyon düzeyi transaction’ın başlangıç anında değil, transaction kontrolü içermeyen ilk satırda bir snapshot alır ve hep bu veri seti üzerinde çalışmaya devam eder. Bu şekilde Read Commited’dan farklılaşan Repeatable Read izolasyon düzeyinde aynı transaction içinde ardarda yapılan select sorguları hep aynı veri setini görecektir. Yani diğer bir deyişle, transaction başladıktan sonra başka transaction’lar tarafından yapılmış commit’ler transactiondaki sorgulara yansımayacaktır. Bu izolasyon düzeyini kullanan uygulamalar da transaction’larını ardışıklaştırma (serialization) hataları yüzünden yeniden işletmeye hazırlıklı olmalıdır.

Repeatable Read modunda transaction’lar veri görüntülerken, transactionlarının başlangıcında aldıkları snapshot’ları gösterirler. Eş zamanlı diğer transaction’lar çalışılan tabloda güncelleme yapıyorsa değişmiş veriyi göstermezler. Bununla birlikte bu modda iki eş zamanlı transaction aynı satır üzerinde güncelleme yapıyorsa sadece ilk COMMIT’in diske yazılmasına izin verilirken, ikinci COMMIT aynı satır üzerinde eş zamanlı güncelleme olduğuna dair bir hata mesajı görür. Sonuç olarak ya ilk COMMIT yapılan ROLLBACK yapacaktır, ya da ikinci COMMIT yapmaya çalışan güncelleme işleminden vazgeçecektir.

**READ COMMITTED**: Repeatable Read’den de zayıf, en düşük düzeyli izolasyon seviyesidir ve PostgreSQL’de varsayılan olarak read commited modu seçili gelmektedir. Bir transaction’daki SELECT cümleciği (FOR UPDATE / SHARE cümleciği içermediğinde) sadece kendi başladığı andan önce COMMIT edilmiş verileri görebilir ve bunlar üzerinde çalışabilir. Bununla birlikte read commiter modunda transaction içindeki SELECT’ler kendisinden önce aynı transaction içinde gerçekleşen güncellemeleri görür. Hatta aynı transaction içinde olsalar dahi, birbirinin ardından gönderilmiş iki SELECT ifadesi dahi farklı verileri görüntüleyebilir. Bu durumun olması için bahsi geçen aynı transaction içindeki ardışık iki select cümlesi arasında, eşzamanlı başka transaction’lardan commit yapılması ve verinin değişmiş olması gerekir.

{% include note.html content="SQL standardında, PostgreSQL tarafından kullanılmayan READ UNCOMMITED izolasyon seviyesi de bulunmaktadır. PostgreSQL’de transaction izolasyon düzeyi READ UNCOMMITED olarak çalıştırılmaya çalışıldığında hata vermez, ancak READ COMMITED gibi çalışmaya devam eder." %}

{% include note.html content="PostgreSQL’de transaction izolasyon düzeyi, transaction’ın ilk sorgu veya veri modifikasyon satırı (select, insert, update, delete, fetch veya copy) okunduktan sonra değiştirilmez." %}

Transaction Erişim Modu, bir transaction’ın okuma/yazma modunda mı yoksa sadece okuma modunda mı olduğunu belirler. Varsayılan mod okuma / yazma modudur ve bu mod okuma moduna dönüştürüldüğünde, transaction içinde geçici tablolar hariç herhangi bir tabloya `INSERT`, `UPDATE`, `DELETE` ve `COPY` komutlarının uygulanmasına izin verilmez. Aynı şekilde okuma modunda veritabanı yapısı ve hakları üzerindeki değişiklikler de yapılamaz. Bu sebeple okuma moduna geçildiğinde `CREATE`, `ALTER`, `DROP`, `COMMENT`, `GRANT`, `REVOKE`, `TRUNCATE`, `EXPLAIN ANALYZE` ve `EXECUTE` komutları da işletilemeyip hata döndürür.

{% include links.html %}

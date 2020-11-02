## Bu Dokümantasyon Projesine Katkı Dökümantasyonu

### 1. Projeyi Düzenleme

1. Projeyi düzenlemek için öncelikle repository'nin **fork** edilmesi gerekmektedir.

2. Repository fork edildikten sonra yapılmak istenilen katkılar aşağıda belirtildiği şekilde yapılır
   ve açıklayıcı bir commit mesajı ile **commit** yapılır.
   
3. Commit yapıldıktan sonra repository'e dönülür ve açıklayıcı bir mesaj ile **pull request** açılır.

4. Pull request açıldıktan sonra **proje yöneticileri** duruma göre katkıyı ekler veya geri çevirir.

### 2. Yeni Sayfa Ekleme
Dökümantasyonda bulunan sayfalar markdown formatında ```/pages/mydoc``` klasöründe bulunmaktadır. 
Yeni sayfa eklemek isteyen kullanıcılar öncelikle bu klasörde yeni bir markdown dosyası
oluşturmalıdır. Bu dosyanın isim formatı şu şekildedir:

```
dosya_ismi.md      Örnek:veri_tipleri.md
```

Markdown, okunması ve yazması kolay bir düz metin formatıdır. Bu markdown dosyaları
HTML'e dönüştürülerek web sayfasına konulmaktadır. Markdown hakkında daha fazla
bilgiye şu sayfada ulaşılabilir: [Mastering Markdown](https://guides.github.com/features/mastering-markdown/).

Her Markdown dosyasının en başına aşağıdaki formatta bir kısım eklenmelidir.

```
---
title: Veri Tipleri
keywords: veri tipleri, veri tipi
last_updated: 27 Ekim 2020
tags: [veri_tipleri]
summary: "Bu sayfa PostgreSQL'de bulunan veri tiplerini tanıtmaktadır."
sidebar: mydoc_sidebar
permalink: veri_tipleri.html
folder: mydoc
---
```

**title, keywords, tags ve summary:**  sayfanın konusuna göre değiştirilir. 
**last_updated:** dosyanın yaratıldığı tarih girilir.
**sidebar** ```mydoc_sidebar``` olmak **zorundadır**. 
**permalink** Markdown dosyasının ismi (.md hariç) sonuna .html konarak eklenir. 
Örnek:

```
veri_tipleri.md --> veri_tipleri.html
```

**folder** ```mydoc``` yazılır ve sayfanın içeriğine geçilir.

Markdown dosyası oluşturulup kaydedildikten sonra soldaki panele
eklenmesi gerekmektedir. Bunun için ```_data\sidebars\mydoc_sidebar.yml``` dosyasına
gidilir. Bu dosyanın özel bir formatı vardır ve bu formata uymak **zorunludur**. Eğer bir
indentation hatası bulunursa konsolda hata mesajı verilir ve web sitesi çalışmayı durdurur.
Her bir alt bölmeye geçildiğinde üsttekine göre bir boşluk bırakılır ve istenilen sayfa eklenir. 
Örnek:

```
  - title: Giriş
    output: web, pdf
    folderitems:

    - title: Support
      url: /mydoc_support.html
      output: web, pdf

    - title: Deneme
      url: /deneme.html
      output: web, pdf

  - title: Veri Tipleri
    output: web, pdf
    folderitems:

    - title: Sayısal Veri Tipleri
      url: /mydoc_sayisal_veri_tipleri.html
      output: web, pdf

    - title: Metinsel Veri Tipleri
      url: /mydoc_metinsel_veri_tipleri.html
      output: web, pdf
```

**title:** İstenilen sayfanın panelde gözükecek adı (title) belirlenir.

**URL:** Markdown dosyasının ismi (.md hariç) sonuna .html uzantısı, başına / eklenerek yazılır. Örnek:

```
veri_tipleri.md --> /veri_tipleri.html
```

Output kısmına ise web, pdf yazılır ve oluşturulan metin, dosyanın istenilen kısmına eklenir. Örnek:
```
- title: Veri Tipleri
  url: /veri_tipleri.html
  output: web, pdf
```

### 3. Sayfa Düzenleme

1. Sayfa düzenlemek için her sayfanın başında Düzenle butonu bulunmaktadır. Düzenlemek istenilen sayfada bu butona basılır.
![Düzenle](/images/contribution_images/1.jpg)

2. Sayfa açıldıktan sonra sağ üstteki kalem ikonuna basılır.
![Kalem](/images/contribution_images/2.jpg)

3. Metin editörü açıldıktan sonra istenilen değişiklikler yapılır. Bu değişikliklerin önizlemesi üstte bulunan *Preview Changes* kısmında görülebilir.
![Preview Changes](/images/contribution_images/3.jpg)

4. Değişiklikler yapıldıktan sonra açıklayıcı bir mesaj yazılır ve *Propose Changes* butonuna basılır.
![Propose Changes](/images/contribution_images/4.jpg)

5. Açılan sayfada *Create Pull Request" butonuna basılır.
![Create Pull Request](/images/contribution_images/5.jpg)

6. Açılan pull request sayfasında *Create Pull Request" butonuna basılır ve değişiklikler proje yöneticisine gönderilir.
![Send Request](/images/contribution_images/6.jpg)

7. Yapılan değişiklikler **proje yöneticisi** tarafından duruma göre kabul edilir veya geri çevrilir.

### 4. Markdown Rehberi

##### Paragraflar

Markdown'da paragraf yaratmak için metinler arasında bir ya da daha fazla boş satır bırakın.

```
Örnek:
Markdown kullanması çok basit bir metin formatıdır.

Sanırım gelecekte yazacağım bütün dökümanlarda Markdown kullanacağım.
```

##### Başlıklar
```
# En büyük başlık (HTML'de <h1>)
## Biraz daha küçük (HTML'de <h2>)
###### En küçük başlık (HTML'de <h6>)
```

Çıktı şu şekilde gözükecek:
# En büyük başlık
## Biraz daha küçük
###### En küçük başlık

---

##### Vurgu
```
*Bu yazı italik yazılır*
_Bu yazı da italik yazılır_

**Bu yazı kalın yazılır**
__Bu yazı da kalın yazılır__

_Bunları **birleştirebilirsiniz**._
```

Çıktı şu şekilde gözükecek:

*Bu yazı italik yazılır*

_Bu yazı da italik yazılır_

**Bu yazı kalın yazılır**

__Bu yazı da kalın yazılır__

_Bunları **birleştirebilirsiniz**._

---

##### Listeler

###### 1. Sırasız
```
* Dosya 1
* Dosya 2
  * Dosya 2a
  * Dosya 2b
```
Çıktı şu şekilde gözükecek:

* Dosya 1
* Dosya 2
  * Dosya 2a
  * Dosya 2b

###### 2. Sıralı
```
1. Dosya 1
2. Dosya 2
3. Dosya 3
   1. Dosya 3a
   2. Dosya 3b
```
Çıktı şu şekilde gözükecek:

1. Dosya 1
2. Dosya 2
3. Dosya 3
   1. Dosya 3a
   2. Dosya 3b

---

##### Resimler
```
Örnek:
![GitHub Logo](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)
Format: ![Başlık](url)

!! url kısmına eklemek istediğiniz resmin proje dosyasında yolu verilerekte ekleyenebilir.
```
Çıktı şu şekilde gözükecek:

![GitHub Logo](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)

---

##### Bağlantılar
```
Örnek:
http://github.com - otomatik!
[GitHub](http://github.com)
```
Çıktı şu şekilde gözükecek:

http://github.com

[GitHub](http://github.com)

---

##### Alıntılar
```
Örnek:
Atatürk'ün dediği gibi:

> Hayatı ve özgürlüğü için ölümü 
>
> göze alan bir millet asla yenilmez.
```

Çıktı şu şekilde gözükecek:

Atatürk'ün dediği gibi:

> Hayatı ve özgürlüğü için ölümü
>
> göze alan bir millet asla yenilmez.

---

##### Satır içinde kod
```
Örnek:
Bu kısımda bir `<addr>` elementi kullanabilirsin.
```
Çıktı şu şekilde gözükecek:

Bu kısımda bir `<addr>` elementi kullanabilirsin.

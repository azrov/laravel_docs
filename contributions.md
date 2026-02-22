# Laravel 12 Dokümantasyonu: Katkı Rehberi (Contribution Guide)

## Hata Raporları (Bug Reports)

Aktif iş birliğini teşvik etmek için Laravel, yalnızca hata raporlarından ziyade pull request'leri şiddetle tavsiye eder. Pull request'ler yalnızca "incelemeye hazır" ("ready for review") olarak işaretlendiklerinde (taslak durumunda değil) ve yeni özellikler için tüm testler geçtiğinde incelenecektir. "Taslak" durumunda bırakılan, bekleyen ve aktif olmayan pull request'ler birkaç gün sonra kapatılacaktır.

Ancak, bir hata raporu dosyalarsanız, sorununuz bir başlık ve sorunun net bir açıklamasını içermelidir. Ayrıca mümkün olduğunca ilgili bilgiyi ve sorunu gösteren bir kod örneğini de eklemelisiniz. Bir hata raporunun amacı, sizin - ve başkalarının - hatayı tekrarlamasını ve bir düzeltme geliştirmesini kolaylaştırmaktır.

Unutmayın, hata raporları, aynı soruna sahip diğer kişilerin sorunu çözme konusunda sizinle iş birliği yapabilmesi umuduyla oluşturulur. Hata raporunun otomatik olarak herhangi bir etkinlik göreceğini veya başkalarının onu düzeltmek için atlayacağını beklemeyin. Bir hata raporu oluşturmak, sorunu çözme yolunda kendinize ve başkalarına yardımcı olmaya yarar. Katkıda bulunmak isterseniz, sorun izleyicilerimizde (issue trackers) listelenen hataları düzelterek yardımcı olabilirsiniz. Tüm Laravel sorunlarını görüntülemek için GitHub'da kimlik doğrulaması yapmış olmanız gerekir.

Laravel'i kullanırken hatalı DocBlock, PHPStan veya IDE uyarıları fark ederseniz, bir GitHub sorunu oluşturmayın. Bunun yerine, sorunu düzeltmek için lütfen bir pull request gönderin.

Laravel kaynak kodu GitHub üzerinde yönetilir ve Laravel projelerinin her biri için depolar (repositories) bulunmaktadır:

*   Laravel Framework
*   Laravel Application
*   Laravel Documentation
*   Laravel Dusk
*   Laravel Cashier
*   Laravel Echo
*   Laravel Envoy
*   Laravel Horizon
*   Laravel Passport
*   Laravel Sanctum
*   Laravel Scout
*   Laravel Socialite
*   Laravel Telescope
*   Laravel Pint
*   Laravel Breeze
*   Laravel Jetstream
*   Laravel Octane
*   Laravel Pennant
*   Laravel Folio
*   Laravel Prompts
*   Laravel Pulse

## Destek Soruları (Support Questions)

Laravel'in GitHub sorun izleyicileri, Laravel yardımı veya desteği sağlamak için tasarlanmamıştır. Bunun yerine aşağıdaki kanallardan birini kullanın:

*   Laravel Forums
*   Laravel Discord
*   Laravel Stack Overflow
*   Laravel subreddit
*   Laraconf US Slack
*   Laravel China

## Çekirdek Geliştirme Tartışmaları (Core Development Discussion)

Yeni özellikler veya mevcut Laravel davranışında iyileştirmeler önerebilirsiniz. Bunu, Laravel framework deposunun GitHub tartışma panosunda (GitHub discussion board) yapabilirsiniz. Yeni bir özellik önerirseniz, lütfen özelliği tamamlamak için gereken kodun en azından bir kısmını uygulamaya istekli olun.

Hatalar, yeni özellikler ve mevcut özelliklerin uygulanmasıyla ilgili gayri resmi tartışmalar, Laravel Discord sunucusunun `#internals` kanalında gerçekleşir. Laravel'in bakımcısı (maintainer) Taylor Otwell, hafta içi 8:00-17:00 (UTC-06:00 veya America/Chicago) saatleri arasında kanalda bulunur ve diğer zamanlarda kanala ara sıra katılır.

## Hangi Dal (Which Branch)?

Tüm hata düzeltmeleri, hata düzeltmelerini destekleyen en son sürüme (şu anda `12.x`) gönderilmelidir. Hata düzeltmeleri, yalnızca yaklaşan sürümde bulunan özellikleri düzeltmedikçe asla `master` dalına gönderilmemelidir.

Geçerli sürümle tamamen geriye dönük uyumlu olan küçük özellikler, en son kararlı dala (şu anda `12.x`) gönderilebilir.

Büyük yeni özellikler veya geriye dönük uyumsuz değişiklikler (breaking changes) içeren özellikler her zaman yaklaşan sürümü içeren `master` dalına gönderilmelidir.

## Derlenmiş Varlıklar (Compiled Assets)

`laravel/laravel` deposundaki `resources/css` veya `resources/js` dosyalarının çoğu gibi derlenmiş bir dosyayı etkileyecek bir değişiklik gönderiyorsanız, derlenmiş dosyaları commit etmeyin. Büyük boyutları nedeniyle, bir bakımcı tarafından gerçekçi bir şekilde incelenemezler. Bu, kötü niyetli kodun Laravel'e enjekte edilmesi için bir yol olarak kullanılabilir. Bunu savunmacı bir şekilde önlemek için, tüm derlenmiş dosyalar Laravel bakımcıları tarafından oluşturulacak ve commit edilecektir.

## Güvenlik Açıkları (Security Vulnerabilities)

Laravel içinde bir güvenlik açığı keşfederseniz, lütfen Taylor Otwell'e taylor@laravel.com adresinden bir e-posta gönderin. Tüm güvenlik açıkları derhal ele alınacaktır.

## Kodlama Stili (Coding Style)

Laravel, PSR-2 kodlama standardını ve PSR-4 otomatik yükleme standardını takip eder.

### PHPDoc

Aşağıda geçerli bir Laravel dokümantasyon bloğu örneği verilmiştir. `@param` özniteliğinden sonra iki boşluk, ardından argüman türü, iki boşluk daha ve son olarak değişken adının geldiğine dikkat edin:
```php
/**
 * Container ile bir bağlama kaydet.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```
Yerel türlerin (native types) kullanılması nedeniyle `@param` veya `@return` öznitelikleri gereksiz olduğunda, kaldırılabilirler:
```php
/**
 * İşi yürüt.
 */
public function handle(AudioProcessor $processor): void
{
    // ...
}
```
Ancak, yerel tür genel (generic) olduğunda, lütfen genel türü `@param` veya `@return` özniteliklerini kullanarak belirtin:
```php
/**
 * Mesaj için ekleri al.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/dosya/yolu'),
    ];
}
```

### StyleCI

Kod stiliniz mükemmel değilse endişelenmeyin! StyleCI, pull request'ler birleştirildikten sonra herhangi bir stil düzeltmesini otomatik olarak Laravel deposuna birleştirecektir. Bu, katkının içeriğine odaklanmamızı sağlar, kod stiline değil.

## Davranış Kuralları (Code of Conduct)

Laravel davranış kuralları, Ruby davranış kurallarından türetilmiştir. Davranış kurallarının herhangi bir ihlali Taylor Otwell'e (taylor@laravel.com) bildirilebilir:

*   Katılımcılar, karşıt görüşlere karşı hoşgörülü olacaklardır.
*   Katılımcılar, dillerinin ve eylemlerinin kişisel saldırılardan ve küçük düşürücü kişisel yorumlardan arınmış olmasını sağlamalıdır.
*   Katılımcılar, başkalarının sözlerini ve eylemlerini yorumlarken her zaman iyi niyet varsaymalıdır.
*   Makul bir şekilde taciz olarak değerlendirilebilecek davranışlara tolerans gösterilmeyecektir.
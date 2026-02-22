# Laravel 12 Dokümantasyonu: E-posta Doğrulama (Email Verification)

## Giriş

Birçok web uygulaması, kullanıcıların uygulamayı kullanmadan önce e-posta adreslerini doğrulamalarını gerektirir. Bu özelliği oluşturduğunuz her uygulama için elle yeniden uygulamak zorunda kalmak yerine, Laravel e-posta doğrulama istekleri göndermek ve doğrulamak için kullanışlı yerleşik servisler sağlar.

Hızlı başlamak mı istiyorsunuz? Yeni bir Laravel uygulamasına bir Laravel uygulama başlangıç kiti (application starter kit) kurun. Başlangıç kitleri, e-posta doğrulama desteği de dahil olmak üzere tüm kimlik doğrulama sisteminizin iskelesini (scaffolding) oluşturacaktır!

### Model Hazırlığı (Model Preparation)
Başlamadan önce, `App\Models\User` modelinizin `Illuminate\Contracts\Auth\MustVerifyEmail` sözleşmesini (contract) uyguladığını (implement) doğrulayın:
```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    // ...
}
```
Bu arayüz (interface) modelinize eklendikten sonra, yeni kaydolan kullanıcılara otomatik olarak bir e-posta doğrulama bağlantısı içeren bir e-posta gönderilecektir. Bu sorunsuz bir şekilde gerçekleşir çünkü Laravel, `Illuminate\Auth\Events\Registered` olayı (event) için `Illuminate\Auth\Listeners\SendEmailVerificationNotification` dinleyicisini (listener) otomatik olarak kaydeder.

Uygulamanızda bir başlangıç kiti kullanmak yerine kaydolmayı manuel olarak uyguluyorsanız, bir kullanıcının kaydı başarılı olduktan sonra `Illuminate\Auth\Events\Registered` olayını gönderdiğinizden (dispatch) emin olmalısınız:
```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

### Veritabanı Hazırlığı (Database Preparation)
Ardından, `users` tablonuz, kullanıcının e-posta adresinin doğrulandığı tarih ve saati saklamak için bir `email_verified_at` sütunu içermelidir. Tipik olarak bu, Laravel'in varsayılan `0001_01_01_000000_create_users_table.php` veritabanı migration'ında bulunur.

## Rotalama (Routing)

E-posta doğrulamayı doğru bir şekilde uygulamak için üç rotanın (routes) tanımlanması gerekecektir. İlk olarak, kullanıcıya kayıttan sonra Laravel'in gönderdiği doğrulama e-postasındaki e-posta doğrulama bağlantısını tıklamaları gerektiğini bildiren bir bildirim görüntülemek için bir rotaya ihtiyaç duyulacaktır.

İkinci olarak, kullanıcı e-postadaki e-posta doğrulama bağlantısını tıkladığında oluşturulan istekleri (requests) işlemek için bir rotaya ihtiyaç duyulacaktır.

Üçüncü olarak, kullanıcı ilk doğrulama bağlantısını yanlışlıkla kaybederse, bir doğrulama bağlantısını yeniden göndermek için bir rotaya ihtiyaç duyulacaktır.

### E-posta Doğrulama Bildirimi (The Email Verification Notice)
Daha önce belirtildiği gibi, kullanıcıya Laravel tarafından kayıttan sonra e-posta ile gönderilen e-posta doğrulama bağlantısını tıklamaları talimatını veren bir görünüm (view) döndürecek bir rota tanımlanmalıdır. Bu görünüm, kullanıcılar önce e-posta adreslerini doğrulamadan uygulamanın diğer bölümlerine erişmeye çalıştıklarında onlara gösterilecektir. Unutmayın, `App\Models\User` modeliniz `MustVerifyEmail` arayüzünü uyguladığı sürece bağlantı kullanıcıya otomatik olarak e-posta ile gönderilir:
```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```
E-posta doğrulama bildirimini döndüren rota, `verification.notice` olarak adlandırılmalıdır. Rotaya bu tam adın atanması önemlidir, çünkü Laravel ile birlikte gelen `verified` ara yazılımı (middleware), bir kullanıcı e-posta adresini doğrulamamışsa otomatik olarak bu rota adına yönlendirecektir.

E-posta doğrulamayı manuel olarak uygularken, doğrulama bildirimi görünümünün içeriğini kendiniz tanımlamanız gerekir. Gerekli tüm kimlik doğrulama ve doğrulama görünümlerini içeren iskeleyi (scaffolding) istiyorsanız, Laravel uygulama başlangıç kitlerine göz atın.

### E-posta Doğrulama İşleyicisi (The Email Verification Handler)
Ardından, kullanıcının kendilerine e-posta ile gönderilen e-posta doğrulama bağlantısını tıkladığında oluşturulan istekleri işleyecek bir rota tanımlamamız gerekiyor. Bu rota, `verification.verify` olarak adlandırılmalı ve `auth` ve `signed` ara yazılımları atanmalıdır:
```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');
```
Devam etmeden önce, bu rotaya daha yakından bakalım. İlk olarak, tipik `Illuminate\Http\Request` örneği yerine bir `EmailVerificationRequest` istek türü kullandığımızı fark edeceksiniz. `EmailVerificationRequest`, Laravel ile birlikte gelen bir form isteğidir (form request). Bu istek, isteğin `id` ve `hash` parametrelerini doğrulamayı otomatik olarak üstlenecektir.

Ardından, doğrudan istek üzerindeki `fulfill` metodunu çağırmaya devam edebiliriz. Bu metod, kimliği doğrulanmış kullanıcı üzerinde `markEmailAsVerified` metodunu çağıracak ve `Illuminate\Auth\Events\Verified` olayını gönderecektir. `markEmailAsVerified` metodu, varsayılan `App\Models\User` modeline `Illuminate\Foundation\Auth\User` temel sınıfı (base class) aracılığıyla sunulur. Kullanıcının e-posta adresi doğrulandıktan sonra, onu istediğiniz yere yönlendirebilirsiniz.

### Doğrulama E-postasını Yeniden Gönderme (Resending the Verification Email)
Bazen bir kullanıcı e-posta adresi doğrulama e-postasını kaybedebilir veya yanlışlıkla silebilir. Buna uyum sağlamak için, kullanıcının doğrulama e-postasının yeniden gönderilmesini talep etmesine izin veren bir rota tanımlamak isteyebilirsiniz. Daha sonra, doğrulama bildirimi görünümünüze basit bir form gönderme düğmesi yerleştirerek bu rotaya bir istekte bulunabilirsiniz:
```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Doğrulama bağlantısı gönderildi!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

### Rotaları Koruma (Protecting Routes)
Rota ara yazılımları (route middleware), yalnızca doğrulanmış kullanıcıların belirli bir rotaya erişmesine izin vermek için kullanılabilir. Laravel, `Illuminate\Auth\Middleware\EnsureEmailIsVerified` ara yazılım sınıfı için bir takma ad (alias) olan `verified` ara yazılım takma adını içerir. Bu takma ad Laravel tarafından otomatik olarak kaydedildiğinden, tek yapmanız gereken `verified` ara yazılımını bir rota tanımına eklemektir. Tipik olarak, bu ara yazılım `auth` ara yazılımıyla eşleştirilir:
```php
Route::get('/profile', function () {
    // Yalnızca doğrulanmış kullanıcılar bu rotaya erişebilir...
})->middleware(['auth', 'verified']);
```
Doğrulanmamış bir kullanıcı, bu ara yazılımın atandığı bir rotaya erişmeye çalışırsa, otomatik olarak `verification.notice` adlı rotaya yönlendirilecektir.

## Özelleştirme (Customization)

#### Doğrulama E-postasını Özelleştirme (Verification Email Customization)
Varsayılan e-posta doğrulama bildirimi çoğu uygulamanın gereksinimlerini karşılasa da, Laravel e-posta doğrulama posta mesajının nasıl oluşturulacağını özelleştirmenize izin verir.

Başlamak için, `Illuminate\Auth\Notifications\VerifyEmail` bildirimi tarafından sağlanan `toMailUsing` metoduna bir closure iletebilirsiniz. Closure, bildirimi alan bildirilebilir model örneğini (notifiable model instance) ve kullanıcının e-posta adresini doğrulamak için ziyaret etmesi gereken imzalı e-posta doğrulama URL'sini alacaktır. Closure, bir `Illuminate\Notifications\Messages\MailMessage` örneği döndürmelidir. Tipik olarak, `toMailUsing` metodunu uygulamanızın `AppServiceProvider` sınıfının `boot` metodundan çağırmalısınız:
```php
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    // ...

    VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
        return (new MailMessage)
            ->subject('E-posta Adresini Doğrula')
            ->line('E-posta adresinizi doğrulamak için aşağıdaki düğmeye tıklayın.')
            ->action('E-posta Adresini Doğrula', $url);
    });
}
```
Posta bildirimleri hakkında daha fazla bilgi edinmek için lütfen posta bildirimi dokümantasyonuna bakın.

## Olaylar (Events)

Laravel uygulama başlangıç kitlerini kullanırken, Laravel e-posta doğrulama süreci sırasında bir `Illuminate\Auth\Events\Verified` olayı gönderir (dispatches). Uygulamanız için e-posta doğrulamayı manuel olarak yönetiyorsanız, doğrulama tamamlandıktan sonra bu olayları manuel olarak göndermek isteyebilirsiniz.
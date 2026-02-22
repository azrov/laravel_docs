# Laravel 12 Dokümantasyonu: Parola Sıfırlama (Resetting Passwords)

## Giriş

Çoğu web uygulaması, kullanıcıların unuttukları parolalarını sıfırlamaları için bir yol sağlar. Laravel, oluşturduğunuz her uygulama için bu özelliği elle yeniden uygulamak zorunda kalmanız yerine, parola sıfırlama bağlantıları göndermek ve parolaları güvenli bir şekilde sıfırlamak için kullanışlı servisler sağlar.

Hızlı başlamak mı istiyorsunuz? Yeni bir Laravel uygulamasına bir Laravel uygulama başlangıç kiti (application starter kit) kurun. Laravel'in başlangıç kitleri, unutulan parolaları sıfırlama da dahil olmak üzere tüm kimlik doğrulama sisteminizin iskelesini (scaffolding) oluşturacaktır!

### Yapılandırma (Configuration)
Uygulamanızın parola sıfırlama yapılandırma dosyası `config/auth.php` konumunda saklanır. Bu dosyada size sunulan seçenekleri gözden geçirdiğinizden emin olun. Varsayılan olarak Laravel, `database` parola sıfırlama sürücüsünü (driver) kullanacak şekilde yapılandırılmıştır.

Parola sıfırlama `driver` yapılandırma seçeneği, parola sıfırlama verilerinin nerede saklanacağını tanımlar. Laravel iki sürücü içerir:

*   `database` - parola sıfırlama verileri ilişkisel bir veritabanında (relational database) saklanır.
*   `cache` - parola sıfırlama verileri önbellek tabanlı depolarınızdan birinde saklanır.

### Sürücü Ön Gereksinimleri (Driver Prerequisites)

#### Veritabanı (Database)
Varsayılan `database` sürücüsünü kullanırken, uygulamanızın parola sıfırlama token'larını saklamak için bir tablo oluşturulmalıdır. Tipik olarak bu, Laravel'in varsayılan `0001_01_01_000000_create_users_table.php` veritabanı migration'ında bulunur.

#### Önbellek (Cache)
Ayrıca, özel bir veritabanı tablosu gerektirmeyen, parola sıfırlamalarını işlemek için kullanılabilir bir `cache` sürücüsü de vardır. Girdiler (entries), kullanıcının e-posta adresine göre anahtarlanır (keyed), bu nedenle uygulamanızın başka bir yerinde e-posta adreslerini önbellek anahtarı olarak kullanmadığınızdan emin olun:
```php
'passwords' => [
    'users' => [
        'driver' => 'cache',
        'provider' => 'users',
        'store' => 'passwords', // İsteğe bağlı...
        'expire' => 60,
        'throttle' => 60,
    ],
],
```
`artisan cache:clear` komutunun parola sıfırlama verilerinizi temizlemesini önlemek için isteğe bağlı olarak `store` yapılandırma anahtarıyla ayrı bir önbellek deposu (cache store) belirtebilirsiniz. Değer, `config/cache.php` yapılandırma değerinizde yapılandırılmış bir depoya karşılık gelmelidir.

### Model Hazırlığı (Model Preparation)
Laravel'in parola sıfırlama özelliklerini kullanmadan önce, uygulamanızın `App\Models\User` modeli `Illuminate\Notifications\Notifiable` özelliğini (trait) kullanmalıdır. Tipik olarak bu özellik, yeni Laravel uygulamalarıyla oluşturulan varsayılan `App\Models\User` modeline zaten dahil edilmiştir.

Ardından, `App\Models\User` modelinizin `Illuminate\Contracts\Auth\CanResetPassword` sözleşmesini (contract) uyguladığını (implement) doğrulayın. Framework ile birlikte gelen `App\Models\User` modeli zaten bu arayüzü (interface) uygular ve arayüzü uygulamak için gereken metotları dahil etmek üzere `Illuminate\Auth\Passwords\CanResetPassword` özelliğini kullanır.

### Güvenilir Ana Bilgisayarları (Trusted Hosts) Yapılandırma
Varsayılan olarak Laravel, HTTP isteğinin `Host` başlığının içeriğine bakılmaksızın aldığı tüm isteklere yanıt verecektir. Ayrıca, `Host` başlığının değeri, bir web isteği sırasında uygulamanıza mutlak URL'ler (absolute URLs) oluşturulurken kullanılacaktır.

Tipik olarak, web sunucunuzu (Nginx veya Apache gibi) yalnızca belirli bir ana bilgisayar adıyla (hostname) eşleşen istekleri uygulamanıza gönderecek şekilde yapılandırmalısınız. Ancak, web sunucunuzu doğrudan özelleştirme yeteneğiniz yoksa ve Laravel'e yalnızca belirli ana bilgisayar adlarına yanıt vermesi talimatını vermeniz gerekiyorsa, bunu uygulamanızın `bootstrap/app.php` dosyasındaki `trustHosts` ara yazılım (middleware) metodunu kullanarak yapabilirsiniz. Bu, özellikle uygulamanız parola sıfırlama işlevselliği sunduğunda önemlidir.

Bu ara yazılım metodu hakkında daha fazla bilgi edinmek için lütfen TrustHosts ara yazılım dokümantasyonuna bakın.

## Rotalama (Routing)

Kullanıcıların parolalarını sıfırlamasına izin verme desteğini doğru bir şekilde uygulamak için birkaç rota (routes) tanımlamamız gerekecektir. İlk olarak, kullanıcının e-posta adresi aracılığıyla bir parola sıfırlama bağlantısı talep etmesini sağlamak için bir çift rotaya ihtiyacımız olacak. İkinci olarak, kullanıcı kendilerine e-posta ile gönderilen parola sıfırlama bağlantısını ziyaret edip parola sıfırlama formunu doldurduğunda parolayı fiilen sıfırlamak için bir çift rotaya ihtiyacımız olacak.

### Parola Sıfırlama Bağlantısını Talep Etme (Requesting the Password Reset Link)

#### Parola Sıfırlama Bağlantısı Talep Formu (The Password Reset Link Request Form)
İlk olarak, parola sıfırlama bağlantılarını talep etmek için gereken rotaları tanımlayacağız. Başlamak için, parola sıfırlama bağlantısı talep formunu içeren bir görünüm (view) döndüren bir rota tanımlayacağız:
```php
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');
```
Bu rotanın döndürdüğü görünüm, bir `email` alanı içeren bir forma sahip olmalıdır; bu form, `password.email` adlı rotaya gönderilecektir.

#### Form Gönderimini İşleme (Handling the Form Submission)
Ardından, "parolamı unuttum" görünümünden gelen form gönderim isteğini işleyen bir rota tanımlayacağız. Bu rota, e-posta adresini doğrulamaktan ve ilgili kullanıcıya parola sıfırlama isteğini göndermekten sorumlu olacaktır:
```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Password;

Route::post('/forgot-password', function (Request $request) {
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::ResetLinkSent
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
})->middleware('guest')->name('password.email');
```
Devam etmeden önce, bu rotayı daha ayrıntılı inceleyelim. İlk olarak, isteğin `email` alanı doğrulanır. Ardından, Laravel'in yerleşik "parola aracısını" (password broker) (Password facade'ı aracılığıyla) kullanarak kullanıcıya bir parola sıfırlama bağlantısı göndereceğiz. Parola aracısı, kullanıcıyı verilen alana (bu durumda e-posta adresi) göre almaktan ve kullanıcıya Laravel'in yerleşik bildirim sistemi (notification system) aracılığıyla bir parola sıfırlama bağlantısı göndermekten sorumlu olacaktır.

`sendResetLink` metodu bir "durum" kısa adı (slug) döndürür. Bu durum, kullanıcıya isteklerinin durumu hakkında kullanıcı dostu bir mesaj görüntülemek için Laravel'in yerelleştirme yardımcıları (localization helpers) kullanılarak çevrilebilir. Parola sıfırlama durumunun çevirisi, uygulamanızın `lang/{lang}/passwords.php` dil dosyası tarafından belirlenir. Durum kısa adının olası her değeri için bir girdi, `passwords` dil dosyasında bulunur.

Varsayılan olarak, Laravel uygulama iskeleti (skeleton) `lang` dizinini içermez. Laravel'in dil dosyalarını özelleştirmek isterseniz, bunları `lang:publish` Artisan komutu aracılığıyla yayınlayabilirsiniz.

`Password` facade'ının `sendResetLink` metodunu çağırırken Laravel'in uygulamanızın veritabanından kullanıcı kaydını nasıl alacağını merak ediyor olabilirsiniz. Laravel parola aracısı, veritabanı kayıtlarını almak için kimlik doğrulama sisteminizin "kullanıcı sağlayıcılarını" (user providers) kullanır. Parola aracısı tarafından kullanılan kullanıcı sağlayıcısı, `config/auth.php` yapılandırma dosyanızın `passwords` yapılandırma dizisi içinde yapılandırılır. Özel kullanıcı sağlayıcıları yazma hakkında daha fazla bilgi edinmek için kimlik doğrulama dokümantasyonuna bakın.

Parola sıfırlamalarını manuel olarak uygularken, görünümlerin ve rotaların içeriğini kendiniz tanımlamanız gerekir. Tüm gerekli kimlik doğrulama ve doğrulama mantığını içeren iskeleyi (scaffolding) istiyorsanız, Laravel uygulama başlangıç kitlerine göz atın.

### Parolayı Sıfırlama (Resetting the Password)

#### Parola Sıfırlama Formu (The Password Reset Form)
Ardından, kullanıcı kendilerine e-posta ile gönderilen parola sıfırlama bağlantısını tıklayıp yeni bir parola sağladığında parolayı fiilen sıfırlamak için gerekli rotaları tanımlayacağız. İlk olarak, kullanıcı parola sıfırlama bağlantısını tıkladığında görüntülenecek olan parola sıfırlama formunu gösterecek rotayı tanımlayalım. Bu rota, parola sıfırlama isteğini daha sonra doğrulamak için kullanacağımız bir `token` parametresi alacaktır:
```php
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');
```
Bu rotanın döndürdüğü görünüm, bir `email` alanı, bir `password` alanı, bir `password_confirmation` alanı ve rotamız tarafından alınan gizli `$token`ın değerini içermesi gereken gizli bir `token` alanı içeren bir form görüntülemelidir.

#### Form Gönderimini İşleme (Handling the Form Submission)
Elbette, parola sıfırlama formu gönderimini fiilen işlemek için bir rota tanımlamamız gerekiyor. Bu rota, gelen isteği doğrulamaktan ve kullanıcının parolasını veritabanında güncellemekten sorumlu olacaktır:
```php
use App\Models\User;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;
use Illuminate\Support\Str;

Route::post('/reset-password', function (Request $request) {
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function (User $user, string $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PasswordReset
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
})->middleware('guest')->name('password.update');
```
Devam etmeden önce, bu rotayı daha ayrıntılı inceleyelim. İlk olarak, isteğin `token`, `email`, `password` ve `password_confirmation` nitelikleri doğrulanır. Ardından, parola sıfırlama isteği kimlik bilgilerini doğrulamak için Laravel'in yerleşik "parola aracısını" (Password facade'ı aracılığıyla) kullanacağız.

Parola aracısına verilen token, e-posta adresi ve parola geçerliyse, `reset` metoduna iletilen closure çağrılacaktır. Kullanıcı örneğini ve parola sıfırlama formuna sağlanan düz metin parolayı alan bu closure içinde, kullanıcının parolasını veritabanında güncelleyebiliriz.

`reset` metodu bir "durum" kısa adı döndürür. Bu durum, kullanıcıya isteklerinin durumu hakkında kullanıcı dostu bir mesaj görüntülemek için Laravel'in yerelleştirme yardımcıları kullanılarak çevrilebilir. Parola sıfırlama durumunun çevirisi, uygulamanızın `lang/{lang}/passwords.php` dil dosyası tarafından belirlenir. Durum kısa adının olası her değeri için bir girdi, `passwords` dil dosyasında bulunur. Uygulamanız bir `lang` dizini içermiyorsa, `lang:publish` Artisan komutunu kullanarak onu oluşturabilirsiniz.

Devam etmeden önce, `Password` facade'ının `reset` metodunu çağırırken Laravel'in uygulamanızın veritabanından kullanıcı kaydını nasıl alacağını merak ediyor olabilirsiniz. Laravel parola aracısı, veritabanı kayıtlarını almak için kimlik doğrulama sisteminizin "kullanıcı sağlayıcılarını" kullanır. Parola aracısı tarafından kullanılan kullanıcı sağlayıcısı, `config/auth.php` yapılandırma dosyanızın `passwords` yapılandırma dizisi içinde yapılandırılır. Özel kullanıcı sağlayıcıları yazma hakkında daha fazla bilgi edinmek için kimlik doğrulama dokümantasyonuna bakın.

## Süresi Dolmuş Token'ları Silme (Deleting Expired Tokens)

`database` sürücüsünü kullanıyorsanız, süresi dolmuş parola sıfırlama token'ları veritabanınızda hala mevcut olacaktır. Ancak, `auth:clear-resets` Artisan komutunu kullanarak bu kayıtları kolayca silebilirsiniz:
```bash
php artisan auth:clear-resets
```
Bu süreci otomatikleştirmek isterseniz, komutu uygulamanızın zamanlayıcısına (scheduler) eklemeyi düşünün:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('auth:clear-resets')->everyFifteenMinutes();
```

## Özelleştirme (Customization)

#### Sıfırlama Bağlantısını Özelleştirme (Reset Link Customization)
`ResetPassword` bildirim sınıfı tarafından sağlanan `createUrlUsing` metodunu kullanarak parola sıfırlama bağlantısı URL'sini özelleştirebilirsiniz. Bu metod, bildirimi alan kullanıcı örneğini ve parola sıfırlama bağlantı token'ını alan bir closure kabul eder. Tipik olarak, bu metodu uygulamanızın `AppServiceProvider` sınıfının `boot` metodundan çağırmalısınız:
```php
use App\Models\User;
use Illuminate\Auth\Notifications\ResetPassword;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    ResetPassword::createUrlUsing(function (User $user, string $token) {
        return 'https://example.com/reset-password?token='.$token;
    });
}
```

#### Sıfırlama E-postasını Özelleştirme (Reset Email Customization)
Kullanıcıya parola sıfırlama bağlantısını göndermek için kullanılan bildirim sınıfını kolayca değiştirebilirsiniz. Başlamak için, `App\Models\User` modelinizde `sendPasswordResetNotification` metodunu geçersiz kılın (override). Bu metod içinde, bildirimi kendi oluşturduğunuz herhangi bir bildirim sınıfını kullanarak gönderebilirsiniz. Parola sıfırlama `$token`ı, metodun aldığı ilk argümandır. Bu `$token`ı, seçtiğiniz parola sıfırlama URL'sini oluşturmak ve bildiriminizi kullanıcıya göndermek için kullanabilirsiniz:
```php
use App\Notifications\ResetPasswordNotification;

/**
 * Kullanıcıya bir parola sıfırlama bildirimi gönder.
 *
 * @param  string  $token
 */
public function sendPasswordResetNotification($token): void
{
    $url = 'https://example.com/reset-password?token='.$token;

    $this->notify(new ResetPasswordNotification($url));
}
```
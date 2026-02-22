# Laravel 12 Dokümantasyonu: Şifreleme (Encryption)

## Giriş

Laravel'in şifreleme servisleri, AES-256 ve AES-128 şifrelemesini kullanarak OpenSSL aracılığıyla metni şifrelemek ve şifresini çözmek için basit, kullanışlı bir arayüz sağlar. Laravel'in tüm şifrelenmiş değerleri, bir mesaj doğrulama kodu (message authentication code - MAC) kullanılarak imzalanır, böylece şifrelendikten sonra temel değerleri değiştirilemez veya kurcalanamaz.

## Yapılandırma (Configuration)

Laravel'in şifreleyicisini (encrypter) kullanmadan önce, `config/app.php` yapılandırma dosyanızda `key` yapılandırma seçeneğini ayarlamalısınız. Bu yapılandırma değeri, `APP_KEY` ortam değişkeni tarafından yönetilir. Bu değişkenin değerini oluşturmak için `php artisan key:generate` komutunu kullanmalısınız, çünkü `key:generate` komutu, uygulamanız için kriptografik olarak güvenli bir anahtar oluşturmak üzere PHP'nin güvenli rastgele bayt üretecini (secure random bytes generator) kullanacaktır. Tipik olarak, `APP_KEY` ortam değişkeninin değeri Laravel'in kurulumu sırasında sizin için oluşturulacaktır.

### Şifreleme Anahtarlarını Sorunsuzca Döndürme (Gracefully Rotating Encryption Keys)
Uygulamanızın şifreleme anahtarını değiştirirseniz, tüm kimliği doğrulanmış kullanıcı oturumlarının oturumu kapatılır. Bunun nedeni, oturum çerezleri de dahil olmak üzere her çerezin Laravel tarafından şifrelenmesidir. Ayrıca, önceki şifreleme anahtarınızla şifrelenmiş herhangi bir verinin şifresini çözmek artık mümkün olmayacaktır.

Bu sorunu hafifletmek için Laravel, uygulamanızın `APP_PREVIOUS_KEYS` ortam değişkeninde önceki şifreleme anahtarlarınızı listelemenize izin verir. Bu değişken, önceki tüm şifreleme anahtarlarınızın virgülle ayrılmış bir listesini içerebilir:
```
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```
Bu ortam değişkenini ayarladığınızda, Laravel değerleri şifrelerken her zaman "geçerli" şifreleme anahtarını kullanacaktır. Ancak, değerlerin şifresini çözerken Laravel önce geçerli anahtarı deneyecek ve geçerli anahtarla şifre çözme başarısız olursa, Laravel değerin şifresini çözebilen bir anahtar bulana kadar tüm önceki anahtarları deneyecektir.

Sorunsuz şifre çözmeye yönelik bu yaklaşım, şifreleme anahtarınız döndürülse bile kullanıcıların uygulamanızı kesintisiz kullanmaya devam etmesine olanak tanır.

## Şifreleyiciyi Kullanma (Using the Encrypter)

#### Bir Değeri Şifreleme (Encrypting a Value)
Bir değeri, `Crypt` facade'ı tarafından sağlanan `encryptString` metodunu kullanarak şifreleyebilirsiniz. Tüm şifrelenmiş değerler OpenSSL ve AES-256-CBC şifresi kullanılarak şifrelenir. Ayrıca, tüm şifrelenmiş değerler bir mesaj doğrulama kodu (MAC) ile imzalanır. Entegre mesaj doğrulama kodu, kötü niyetli kullanıcılar tarafından kurcalanmış herhangi bir değerin şifresinin çözülmesini önleyecektir:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    /**
     * Kullanıcı için bir DigitalOcean API token'ı sakla.
     */
    public function store(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'token' => Crypt::encryptString($request->token),
        ])->save();

        return redirect('/secrets');
    }
}
```

#### Bir Değerin Şifresini Çözme (Decrypting a Value)
Değerlerin şifresini, `Crypt` facade'ı tarafından sağlanan `decryptString` metodunu kullanarak çözebilirsiniz. Değer düzgün bir şekilde çözülemezse, örneğin mesaj doğrulama kodu geçersizse, bir `Illuminate\Contracts\Encryption\DecryptException` fırlatılacaktır:
```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```
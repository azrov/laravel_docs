# Laravel 12 Dokümantasyonu: Hash'leme (Hashing)

## Giriş

Laravel'in `Hash` facade'ı, kullanıcı parolalarını saklamak için güvenli Bcrypt ve Argon2 hash'lemesi sağlar. Laravel uygulama başlangıç kitlerinden (application starter kits) birini kullanıyorsanız, kayıt ve kimlik doğrulama için varsayılan olarak Bcrypt kullanılacaktır.

Bcrypt, parolaları hash'lemek için harika bir seçimdir çünkü "iş faktörü" (work factor) ayarlanabilir; bu, donanım gücü arttıkça bir hash oluşturmak için gereken sürenin de artırılabileceği anlamına gelir. Parolaları hash'lerken, yavaş olmak iyidir. Bir algoritmanın bir parolayı hash'lemesi ne kadar uzun sürerse, kötü niyetli kullanıcıların uygulamalara karşı kaba kuvvet saldırılarında (brute force attacks) kullanılabilecek olası tüm dize hash değerlerinden oluşan "gökkuşağı tabloları" (rainbow tables) oluşturması da o kadar uzun sürer.

## Yapılandırma (Configuration)

Varsayılan olarak Laravel, verileri hash'lerken `bcrypt` hashing sürücüsünü kullanır. Ancak, `argon` ve `argon2id` dahil olmak üzere birkaç başka hash'leme sürücüsü de desteklenmektedir.

Uygulamanızın hash'leme sürücüsünü `HASH_DRIVER` ortam değişkenini kullanarak belirtebilirsiniz. Ancak, Laravel'in tüm hash'leme sürücüsü seçeneklerini özelleştirmek isterseniz, `config:publish` Artisan komutunu kullanarak eksiksiz hash'leme yapılandırma dosyasını yayınlamalısınız:
```bash
php artisan config:publish hashing
```

## Temel Kullanım (Basic Usage)

### Parolaları Hash'leme (Hashing Passwords)
`Hash` facade'ında `make` metodunu çağırarak bir parolayı hash'leyebilirsiniz:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    /**
     * Kullanıcı için parolayı güncelle.
     */
    public function update(Request $request): RedirectResponse
    {
        // Yeni parola uzunluğunu doğrula...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

#### Bcrypt İş Faktörünü Ayarlama (Adjusting The Bcrypt Work Factor)
Bcrypt algoritmasını kullanıyorsanız, `make` metodu, `rounds` seçeneğini kullanarak algoritmanın iş faktörünü (work factor) yönetmenize olanak tanır; ancak, Laravel tarafından yönetilen varsayılan iş faktörü çoğu uygulama için kabul edilebilir düzeydedir:
```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

#### Argon2 İş Faktörünü Ayarlama (Adjusting The Argon2 Work Factor)
Argon2 algoritmasını kullanıyorsanız, `make` metodu, `memory`, `time` ve `threads` seçeneklerini kullanarak algoritmanın iş faktörünü yönetmenize olanak tanır; ancak, Laravel tarafından yönetilen varsayılan değerler çoğu uygulama için kabul edilebilir düzeydedir:
```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```
Bu seçenekler hakkında daha fazla bilgi için lütfen Argon hash'leme ile ilgili resmi PHP dokümantasyonuna bakın.

### Bir Parolanın Bir Hash'le Eşleştiğini Doğrulama (Verifying That a Password Matches a Hash)
`Hash` facade'ı tarafından sağlanan `check` metodu, belirli bir düz metin dizesinin (plain-text string) belirli bir hash ile eşleşip eşleşmediğini doğrulamanıza olanak tanır:
```php
if (Hash::check('düz-metin', $hashedPassword)) {
    // Parolalar eşleşiyor...
}
```

### Bir Parolanın Yeniden Hash'lenmesi Gerekip Gerekmediğini Belirleme (Determining if a Password Needs to be Rehashed)
`Hash` facade'ı tarafından sağlanan `needsRehash` metodu, parola hash'lendikten sonra hash'leyici tarafından kullanılan iş faktörünün değişip değişmediğini belirlemenize olanak tanır. Bazı uygulamalar, uygulamanın kimlik doğrulama süreci sırasında bu kontrolü yapmayı tercih eder:
```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('düz-metin');
}
```

## Hash Algoritması Doğrulaması (Hash Algorithm Verification)

Hash algoritması manipülasyonunu önlemek için Laravel'in `Hash::check` metodu, önce verilen hash'in uygulamanın seçili hash'leme algoritması kullanılarak oluşturulduğunu doğrulayacaktır. Algoritmalar farklıysa, bir `RuntimeException` istisnası fırlatılacaktır.

Bu, hash'leme algoritmasının değişmesinin beklenmediği ve farklı algoritmaların kötü niyetli bir saldırının göstergesi olabileceği çoğu uygulama için beklenen davranıştır. Ancak, uygulamanızda birden çok hash'leme algoritmasını desteklemeniz gerekiyorsa (örneğin bir algoritmadan diğerine geçiş yaparken), `HASH_VERIFY` ortam değişkenini `false` olarak ayarlayarak hash algoritması doğrulamasını devre dışı bırakabilirsiniz:
```
HASH_VERIFY=false
```
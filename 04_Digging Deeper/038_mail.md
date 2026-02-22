# Laravel 12 Dokümantasyonu: E-posta (Mail)

## Giriş

E-posta göndermek karmaşık olmak zorunda değildir. Laravel, popüler Symfony Mailer bileşeni tarafından desteklenen temiz, basit bir e-posta API'si sağlar. Laravel ve Symfony Mailer, SMTP, Mailgun, Postmark, Resend, Amazon SES ve `sendmail` aracılığıyla e-posta göndermek için sürücüler (drivers) sağlayarak, yerel veya bulut tabanlı bir hizmet aracılığıyla hızlıca e-posta göndermeye başlamanıza olanak tanır.

### Yapılandırma (Configuration)
Laravel'in e-posta servisleri, uygulamanızın `config/mail.php` yapılandırma dosyası aracılığıyla yapılandırılabilir. Bu dosya içinde yapılandırılan her bir posta gönderici (mailer), kendine özgü bir yapılandırmaya ve hatta kendine özgü bir "ulaşım yöntemine" (transport) sahip olabilir. Bu, uygulamanızın belirli e-posta mesajlarını göndermek için farklı e-posta servislerini kullanmasına olanak tanır. Örneğin, uygulamanız işlemsel (transactional) e-postaları göndermek için Postmark kullanırken, toplu e-postaları (bulk emails) göndermek için Amazon SES kullanabilir.

`mailers` yapılandırma dizisi içinde, her bir ana posta sürücüsü / ulaşım yöntemi için örnek bir yapılandırma girdisi bulunur. `default` yapılandırma değeri ise, uygulamanızın bir e-posta mesajı göndermesi gerektiğinde varsayılan olarak hangi posta göndericinin kullanılacağını belirler.

### Sürücü / Ulaşım Yöntemi Ön Gereksinimleri (Driver / Transport Prerequisites)
Mailgun, Postmark ve Resend gibi API tabanlı sürücüler, SMTP sunucuları üzerinden e-posta göndermekten genellikle daha basit ve daha hızlıdır. Mümkün olduğunda, bu sürücülerden birini kullanmanızı öneririz.

#### Mailgun Sürücüsü
Mailgun sürücüsünü kullanmak için Symfony'nin Mailgun Mailer ulaşım yöntemini Composer aracılığıyla kurun:
```bash
composer require symfony/mailgun-mailer symfony/http-client
```
Ardından, uygulamanızın `config/mail.php` yapılandırma dosyasında iki değişiklik yapmanız gerekecektir. İlk olarak, varsayılan posta göndericinizi `mailgun` olarak ayarlayın:
```php
'default' => env('MAIL_MAILER', 'mailgun'),
```
İkinci olarak, `mailers` dizinize aşağıdaki yapılandırma dizisini ekleyin:
```php
'mailgun' => [
    'transport' => 'mailgun',
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```
Uygulamanızın varsayılan posta göndericisini yapılandırdıktan sonra, aşağıdaki seçenekleri `config/services.php` yapılandırma dosyanıza ekleyin:
```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```
Birleşik Devletler Mailgun bölgesini kullanmıyorsanız, bölgenizin uç noktasını (endpoint) `services` yapılandırma dosyasında tanımlayabilirsiniz:
```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    'scheme' => 'https',
],
```

#### Postmark Sürücüsü
Postmark sürücüsünü kullanmak için Symfony'nin Postmark Mailer ulaşım yöntemini Composer aracılığıyla kurun:
```bash
composer require symfony/postmark-mailer symfony/http-client
```
Ardından, uygulamanızın `config/mail.php` yapılandırma dosyasındaki `default` seçeneğini `postmark` olarak ayarlayın. Uygulamanızın varsayılan posta göndericisini yapılandırdıktan sonra, `config/services.php` yapılandırma dosyanızın aşağıdaki seçenekleri içerdiğinden emin olun:
```php
'postmark' => [
    'key' => env('POSTMARK_API_KEY'),
],
```
Belirli bir posta gönderici tarafından kullanılması gereken Postmark mesaj akışını (message stream) belirtmek isterseniz, posta göndericinin yapılandırma dizisine `message_stream_id` yapılandırma seçeneğini ekleyebilirsiniz. Bu yapılandırma dizisi, uygulamanızın `config/mail.php` yapılandırma dosyasında bulunabilir:
```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```
Bu şekilde, farklı mesaj akışlarına sahip birden çok Postmark posta göndericisi de ayarlayabilirsiniz.

#### Resend Sürücüsü
Resend sürücüsünü kullanmak için Resend'in PHP SDK'sını Composer aracılığıyla kurun:
```bash
composer require resend/resend-php
```
Ardından, uygulamanızın `config/mail.php` yapılandırma dosyasındaki `default` seçeneğini `resend` olarak ayarlayın. Uygulamanızın varsayılan posta göndericisini yapılandırdıktan sonra, `config/services.php` yapılandırma dosyanızın aşağıdaki seçenekleri içerdiğinden emin olun:
```php
'resend' => [
    'key' => env('RESEND_API_KEY'),
],
```

#### SES Sürücüsü
Amazon SES sürücüsünü kullanmak için önce Amazon AWS SDK for PHP'yi kurmalısınız. Bu kütüphaneyi Composer paket yöneticisi aracılığıyla kurabilirsiniz:
```bash
composer require aws/aws-sdk-php
```
Ardından, `config/mail.php` dosyanızdaki `default` seçeneğini `ses` olarak ayarlayın ve `config/services.php` yapılandırma dosyanızın aşağıdaki seçenekleri içerdiğini doğrulayın:
```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```
Bir oturum token'ı (session token) aracılığıyla AWS geçici kimlik bilgilerini (temporary credentials) kullanmak için, uygulamanızın SES yapılandırmasına bir `token` anahtarı ekleyebilirsiniz:
```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'token' => env('AWS_SESSION_TOKEN'),
],
```
SES'in abonelik yönetimi özellikleriyle etkileşime girmek için, bir e-posta mesajının `headers` metodunda döndürülen diziye `X-Ses-List-Management-Options` başlığını ekleyebilirsiniz:
```php
/**
 * Mesaj başlıklarını al.
 */
public function headers(): Headers
{
    return new Headers(
        text: [
            'X-Ses-List-Management-Options' => 'contactListName=MyContactList;topicName=MyTopic',
        ],
    );
}
```
Laravel'in bir e-posta gönderirken AWS SDK'sının `SendEmail` metoduna iletmesi gereken ek seçenekleri tanımlamak isterseniz, `ses` yapılandırmanız içinde bir `options` dizisi tanımlayabilirsiniz:
```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'EmailTags' => [
            ['Name' => 'foo', 'Value' => 'bar'],
        ],
    ],
],
```

### Yedekli (Failover) Yapılandırma
Bazen, uygulamanızın e-postalarını göndermek için yapılandırdığınız harici bir hizmet çökebilir. Bu durumlarda, birincil teslimat sürücünüz çöktüğünde kullanılacak bir veya daha fazla yedek e-posta teslimat yapılandırması tanımlamak faydalı olabilir.

Bunu başarmak için uygulamanızın `failover` ulaşım yöntemini kullanan bir posta gönderici tanımlamalısınız. Yedekli posta göndericinizin yapılandırma dizisi, yapılandırılmış posta göndericilerin teslimat için hangi sırayla seçilmesi gerektiğini referans alan bir `mailers` dizisi içermelidir:
```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```
`failover` ulaşım yöntemini kullanan bir posta gönderici yapılandırdıktan sonra, yedekli işlevselliği kullanmak için uygulamanızın `.env` dosyasında `failover` posta göndericisini varsayılan posta göndericiniz olarak ayarlamanız gerekecektir:
```
MAIL_MAILER=failover
```

### Sıra Tablosu (Round Robin) Yapılandırma
`roundrobin` ulaşım yöntemi, e-posta iş yükünüzü birden çok posta gönderici arasında dağıtmanıza olanak tanır. Başlamak için uygulamanızın `roundrobin` ulaşım yöntemini kullanan bir posta gönderici tanımlayın. Sıra tablosu posta göndericinizin yapılandırma dizisi, teslimat için hangi yapılandırılmış posta göndericilerin kullanılması gerektiğini referans alan bir `mailers` dizisi içermelidir:
```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```
Sıra tablosu posta göndericiniz tanımlandıktan sonra, adını uygulamanızın e-posta yapılandırmasındaki `default` yapılandırma anahtarının değeri olarak belirterek bu posta göndericiyi uygulamanız tarafından kullanılan varsayılan posta gönderici olarak ayarlamalısınız:
```php
'default' => env('MAIL_MAILER', 'roundrobin'),
```
Sıra tablosu ulaşım yöntemi, yapılandırılmış posta göndericiler listesinden rastgele bir posta gönderici seçer ve ardından sonraki her e-posta için bir sonraki kullanılabilir posta göndericiye geçer. Yüksek kullanılabilirlik (high availability) elde etmeye yardımcı olan `failover` ulaşım yönteminin aksine, `roundrobin` ulaşım yöntemi yük dengelemesi (load balancing) sağlar.

## Postalanabilirler (Mailables) Oluşturma

Laravel uygulamaları oluştururken, uygulamanız tarafından gönderilen her e-posta türü bir "postalanabilir" (mailable) sınıf olarak temsil edilir. Bu sınıflar `app/Mail` dizininde saklanır. Uygulamanızda bu dizini görmüyorsanız endişelenmeyin, çünkü `make:mail` Artisan komutunu kullanarak ilk postalanabilir sınıfınızı oluşturduğunuzda sizin için oluşturulacaktır:
```bash
php artisan make:mail OrderShipped
```

## Postalanabilirler Yazma (Writing Mailables)

Bir postalanabilir sınıf oluşturduktan sonra, içeriğini keşfetmek için açalım. Postalanabilir sınıf yapılandırması, `envelope`, `content` ve `attachments` metotları dahil olmak üzere birkaç metotta yapılır.

`envelope` metodu, mesajın konusunu (subject) ve bazen alıcılarını tanımlayan bir `Illuminate\Mail\Mailables\Envelope` nesnesi döndürür. `content` metodu, mesaj içeriğini oluşturmak için kullanılacak Blade şablonunu tanımlayan bir `Illuminate\Mail\Mailables\Content` nesnesi döndürür.

### Göndericiyi (Sender) Yapılandırma

#### Zarfı (Envelope) Kullanma
İlk olarak, e-postanın göndericisini yapılandırmayı inceleyelim. Veya başka bir deyişle, e-postanın kimden ("from") geldiğini. Göndericiyi yapılandırmanın iki yolu vardır. İlk olarak, mesajınızın zarfında (envelope) "from" adresini belirtebilirsiniz:
```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * Mesaj zarfını al.
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```
İsterseniz, bir `replyTo` (yanıtla) adresi de belirtebilirsiniz:
```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

#### Global Bir "from" Adresi Kullanma
Ancak, uygulamanız tüm e-postaları için aynı "from" adresini kullanıyorsa, bunu oluşturduğunuz her postalanabilir sınıfa eklemek zahmetli hale gelebilir. Bunun yerine, `config/mail.php` yapılandırma dosyanızda global bir "from" adresi belirtebilirsiniz. Postalanabilir sınıf içinde başka bir "from" adresi belirtilmemişse bu adres kullanılacaktır:
```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```
Ek olarak, `config/mail.php` yapılandırma dosyanızda global bir "reply_to" adresi tanımlayabilirsiniz:
```php
'reply_to' => [
    'address' => 'example@example.com',
    'name' => 'App Name',
],
```

### Görünümü (View) Yapılandırma
Bir postalanabilir sınıfın `content` metodu içinde, e-postanın içeriğini oluştururken kullanılması gereken görünümü (view) veya şablonu tanımlayabilirsiniz. Her e-posta tipik olarak içeriğini oluşturmak için bir Blade şablonu kullandığından, e-postanızın HTML'ini oluştururken Blade şablonlama motorunun tüm gücüne ve rahatlığına sahip olursunuz:
```php
/**
 * Mesaj içerik tanımını al.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```
Tüm e-posta şablonlarınızı barındırmak için bir `resources/views/mail` dizini oluşturmak isteyebilirsiniz; ancak, bunları `resources/views` dizininiz içinde istediğiniz yere yerleştirmekte özgürsünüz.

#### Düz Metin (Plain Text) E-postalar
E-postanızın düz metin bir sürümünü tanımlamak isterseniz, mesajın `Content` tanımını oluştururken düz metin şablonunu belirtebilirsiniz. `view` parametresi gibi, `text` parametresi de e-postanın içeriğini oluşturmak için kullanılacak bir şablon adı olmalıdır. Mesajınızın hem HTML hem de düz metin sürümünü tanımlamakta özgürsünüz:
```php
/**
 * Mesaj içerik tanımını al.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```
Açıklık olması açısından, `html` parametresi `view` parametresinin bir takma adı (alias) olarak kullanılabilir:
```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

### Görünüm Verileri (View Data)

#### Genel (Public) Özellikler Aracılığıyla
Tipik olarak, e-postanın HTML'ini oluştururken kullanabileceğiniz bazı verileri görünümünüze iletmek isteyeceksiniz. Verileri görünümünüze sunmanın iki yolu vardır. İlk olarak, postalanabilir sınıfınızda tanımlanan herhangi bir genel (public) özellik otomatik olarak görünüme sunulacaktır. Örneğin, verileri postalanabilir sınıfınızın kurucusuna (constructor) iletebilir ve bu verileri sınıfta tanımlanan genel özelliklere ayarlayabilirsiniz:
```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Yeni bir mesaj örneği oluştur.
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * Mesaj içerik tanımını al.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```
Veriler genel bir özelliğe ayarlandıktan sonra, görünümünüzde otomatik olarak kullanılabilir olacaktır, böylece Blade şablonlarınızda diğer tüm verilere eriştiğiniz gibi ona erişebilirsiniz:
```blade
<div>
    Price: {{ $order->price }}
</div>
```

#### `with` Parametresi Aracılığıyla
E-posta verilerinizin biçimini şablona gönderilmeden önce özelleştirmek isterseniz, `Content` tanımının `with` parametresi aracılığıyla verilerinizi manuel olarak görünüme iletebilirsiniz. Tipik olarak, verileri yine postalanabilir sınıfın kurucusu aracılığıyla ileteceksiniz; ancak, bu verilerin otomatik olarak şablona sunulmaması için bunları `protected` veya `private` özelliklere ayarlamalısınız:
```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Yeni bir mesaj örneği oluştur.
     */
    public function __construct(
        protected Order $order,
    ) {}

    /**
     * Mesaj içerik tanımını al.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            with: [
                'orderName' => $this->order->name,
                'orderPrice' => $this->order->price,
            ],
        );
    }
}
```
# Laravel 12 Dokümantasyonu: Laravel AI SDK

## Giriş

Laravel AI SDK, OpenAI, Anthropic, Gemini ve daha fazlası gibi yapay zeka sağlayıcılarıyla (AI providers) etkileşim kurmak için birleşik, anlamlı bir API sağlar. AI SDK ile, akıllı ajanları (agents) araçlar ve yapılandırılmış çıktıyla (structured output) oluşturabilir, görüntüler üretebilir, ses sentezleyebilir ve dökümünü çıkarabilir, vektör yerleştirmeleri (vector embeddings) oluşturabilir ve çok daha fazlasını yapabilirsiniz - tüm bunlar tutarlı, Laravel dostu bir arayüz kullanarak.

## Kurulum (Installation)

Laravel AI SDK'yı Composer aracılığıyla kurabilirsiniz:
```bash
composer require laravel/ai
```
Ardından, AI SDK yapılandırma ve migration dosyalarını `vendor:publish` Artisan komutunu kullanarak yayınlamalısınız:
```bash
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```
Son olarak, uygulamanızın veritabanı migration'larını çalıştırmalısınız. Bu, AI SDK'nın konuşma depolamasını desteklemek için kullandığı `agent_conversations` ve `agent_conversation_messages` tablolarını oluşturacaktır:
```bash
php artisan migrate
```

### Yapılandırma (Configuration)
AI sağlayıcı kimlik bilgilerinizi (credentials) uygulamanızın `config/ai.php` yapılandırma dosyasında veya uygulamanızın `.env` dosyasında ortam değişkenleri (environment variables) olarak tanımlayabilirsiniz:
```
ANTHROPIC_API_KEY=
COHERE_API_KEY=
ELEVENLABS_API_KEY=
GEMINI_API_KEY=
MISTRAL_API_KEY=
OLLAMA_API_KEY=
OPENAI_API_KEY=
JINA_API_KEY=
VOYAGEAI_API_KEY=
XAI_API_KEY=
```
Metin, görüntü, ses, transkripsiyon ve yerleştirmeler için kullanılan varsayılan modeller de uygulamanızın `config/ai.php` yapılandırma dosyasında yapılandırılabilir.

### Özel Temel URL'ler (Custom Base URLs)
Varsayılan olarak Laravel AI SDK, doğrudan her sağlayıcının genel API uç noktasına (public API endpoint) bağlanır. Ancak, istekleri farklı bir uç nokta üzerinden yönlendirmeniz gerekebilir - örneğin, API anahtar yönetimini merkezileştirmek için bir proxy servisi kullanırken, hız sınırlaması (rate limiting) uygularken veya trafiği bir kurumsal ağ geçidi (corporate gateway) üzerinden yönlendirirken.

Sağlayıcı yapılandırmanıza bir `url` parametresi ekleyerek özel temel URL'ler yapılandırabilirsiniz:
```php
'providers' => [
    'openai' => [
        'driver' => 'openai',
        'key' => env('OPENAI_API_KEY'),
        'url' => env('OPENAI_BASE_URL'),
    ],

    'anthropic' => [
        'driver' => 'anthropic',
        'key' => env('ANTHROPIC_API_KEY'),
        'url' => env('ANTHROPIC_BASE_URL'),
    ],
],
```
Bu, istekleri bir proxy servisi (LiteLLM veya Azure OpenAI Gateway gibi) üzerinden yönlendirirken veya alternatif uç noktalar kullanırken kullanışlıdır.

Özel temel URL'ler şu sağlayıcılar için desteklenir: OpenAI, Anthropic, Gemini, Groq, Cohere, DeepSeek, xAI ve OpenRouter.

### Sağlayıcı Desteği (Provider Support)
AI SDK, özellikleri arasında çeşitli sağlayıcıları destekler. Aşağıdaki tablo, her özellik için hangi sağlayıcıların mevcut olduğunu özetlemektedir:

| Özellik | Sağlayıcılar |
| :--- | :--- |
| Metin (Text) | OpenAI, Anthropic, Gemini, Azure, Groq, xAI, DeepSeek, Mistral, Ollama |
| Görüntüler (Images) | OpenAI, Gemini, xAI |
| TTS (Metinden Sese) | OpenAI, ElevenLabs |
| STT (Sesten Metne) | OpenAI, ElevenLabs, Mistral |
| Yerleştirmeler (Embeddings) | OpenAI, Gemini, Azure, Cohere, Mistral, Jina, VoyageAI |
| Yeniden Sıralama (Reranking) | Cohere, Jina |
| Dosyalar (Files) | OpenAI, Anthropic, Gemini |

`Laravel\Ai\Enums\Lab` enum'u, kodunuz boyunca düz dizeler (plain strings) kullanmak yerine sağlayıcılara başvurmak için kullanılabilir:
```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::Gemini;
// ...
```

## Ajanlar (Agents)

Ajanlar, Laravel AI SDK'da yapay zeka sağlayıcılarıyla etkileşim kurmak için temel yapı taşıdır. Her ajan, büyük bir dil modeliyle (large language model - LLM) etkileşim kurmak için gereken talimatları, konuşma bağlamını, araçları ve çıktı şemasını (output schema) kapsülleyen özel bir PHP sınıfıdır. Bir ajanı özel bir asistan olarak düşünün - bir satış koçu, bir belge analizcisi, bir destek botu - bunu bir kez yapılandırır ve uygulamanız boyunca ihtiyaç duydukça sorgularsınız.

`make:agent` Artisan komutu aracılığıyla bir ajan oluşturabilirsiniz:
```bash
php artisan make:agent SalesCoach

php artisan make:agent SalesCoach --structured
```
Oluşturulan ajan sınıfı içinde, sistem istemini (system prompt) / talimatlarını, mesaj bağlamını, mevcut araçları ve çıktı şemasını (varsa) tanımlayabilirsiniz:
```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\RetrievePreviousTranscripts;
use App\Models\History;
use App\Models\User;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Messages\Message;
use Laravel\Ai\Promptable;
use Stringable;

class SalesCoach implements Agent, Conversational, HasTools, HasStructuredOutput
{
    use Promptable;

    public function __construct(public User $user) {}

    /**
     * Ajanın uyması gereken talimatları al.
     */
    public function instructions(): Stringable|string
    {
        return 'Sen bir satış koçusun. Konuşma dökümlerini analiz ediyor, geri bildirim ve genel bir satış gücü puanı sağlıyorsun.';
    }

    /**
     * Şimdiye kadarki konuşmayı oluşturan mesajların listesini al.
     */
    public function messages(): iterable
    {
        return History::where('user_id', $this->user->id)
            ->latest()
            ->limit(50)
            ->get()
            ->reverse()
            ->map(function ($message) {
                return new Message($message->role, $message->content);
            })->all();
    }

    /**
     * Ajanın kullanabileceği araçları al.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RetrievePreviousTranscripts,
        ];
    }

    /**
     * Ajanın yapılandırılmış çıktı şeması tanımını al.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'feedback' => $schema->string()->required(),
            'score' => $schema->integer()->min(1)->max(10)->required(),
        ];
    }
}
```

### Sorgulama (Prompting)
Bir ajanı sorgulamak için, önce `make` yöntemini veya standart örneklemeyi (instantiation) kullanarak bir örnek oluşturun, ardından `prompt` çağrısını yapın:
```php
$response = (new SalesCoach)
    ->prompt('Bu satış dökümünü analiz et...');

$response = SalesCoach::make()
    ->prompt('Bu satış dökümünü analiz et...');

return (string) $response;
```
`make` metodu, ajanınızı container'dan çözümler (resolves) ve otomatik bağımlılık enjeksiyonuna (automatic dependency injection) izin verir. Ayrıca ajanın kurucusuna (constructor) argümanlar iletebilirsiniz:
```php
$agent = SalesCoach::make(user: $user);
```
`prompt` metoduna ek argümanlar ileterek, sorgulama yaparken varsayılan sağlayıcıyı, modeli veya HTTP zaman aşımını (timeout) geçersiz kılabilirsiniz:
```php
$response = (new SalesCoach)->prompt(
    'Bu satış dökümünü analiz et...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);
```

### Konuşma Bağlamı (Conversation Context)
Ajanınız `Conversational` arayüzünü (interface) uygularsa (implement), varsa önceki konuşma bağlamını döndürmek için `messages` metodunu kullanabilirsiniz:
```php
use App\Models\History;
use Laravel\Ai\Messages\Message;

/**
 * Şimdiye kadarki konuşmayı oluşturan mesajların listesini al.
 */
public function messages(): iterable
{
    return History::where('user_id', $this->user->id)
        ->latest()
        ->limit(50)
        ->get()
        ->reverse()
        ->map(function ($message) {
            return new Message($message->role, $message->content);
        })->all();
}
```

#### Konuşmaları Hatırlama (Remembering Conversations)
`RemembersConversations` özelliğini (trait) kullanmadan önce, `vendor:publish` Artisan komutunu kullanarak AI SDK migration'larını yayınlamalı ve çalıştırmalısınız. Bu migration'lar, konuşmaları depolamak için gerekli veritabanı tablolarını oluşturacaktır.

Laravel'in ajanınız için konuşma geçmişini otomatik olarak saklamasını ve almasını isterseniz, `RemembersConversations` özelliğini kullanabilirsiniz. Bu özellik, `Conversational` arayüzünü manuel olarak uygulamadan konuşma mesajlarını veritabanına kaydetmenin basit bir yolunu sağlar:
```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational
{
    use Promptable, RemembersConversations;

    /**
     * Ajanın uyması gereken talimatları al.
     */
    public function instructions(): string
    {
        return 'Sen bir satış koçusun...';
    }
}
```
Bir kullanıcı için yeni bir konuşma başlatmak üzere, sorgulamadan önce `forUser` metodunu çağırın:
```php
$response = (new SalesCoach)->forUser($user)->prompt('Merhaba!');

$conversationId = $response->conversationId;
```
Konuşma ID'si yanıt üzerinde döndürülür ve gelecekte başvurmak üzere saklanabilir veya doğrudan `agent_conversations` tablosundan bir kullanıcının tüm konuşmalarını alabilirsiniz.

Mevcut bir konuşmaya devam etmek için `continue` metodunu kullanın:
```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Bunun hakkında daha fazla bilgi ver.');
```
`RemembersConversations` özelliğini kullanırken, sorgulama sırasında önceki mesajlar otomatik olarak yüklenir ve konuşma bağlamına dahil edilir. Her etkileşimden sonra yeni mesajlar (hem kullanıcı hem de asistan) otomatik olarak saklanır.

### Yapılandırılmış Çıktı (Structured Output)
Ajanınızın yapılandırılmış çıktı döndürmesini istiyorsanız, `HasStructuredOutput` arayüzünü uygulayın; bu, ajanınızın bir `schema` metodu tanımlamasını gerektirir:
```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * Ajanın yapılandırılmış çıktı şeması tanımını al.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
        ];
    }
}
```
Yapılandırılmış çıktı döndüren bir ajanı sorgularken, döndürülen `StructuredAgentResponse`'e bir dizi gibi erişebilirsiniz:
```php
$response = (new SalesCoach)->prompt('Bu satış dökümünü analiz et...');

return $response['score'];
```

### Ekler (Attachments)
Sorgulama yaparken, modele görselleri ve belgeleri inceleme olanağı sağlamak için sorguya ekler (attachments) de iletebilirsiniz:
```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Ekli satış dökümünü analiz et...',
    attachments: [
        Files\Document::fromStorage('transcript.pdf'), // Bir dosya sistemi diskinden belge ekle...
        Files\Document::fromPath('/home/laravel/transcript.md'), // Yerel bir yoldan belge ekle...
        $request->file('transcript'), // Yüklenmiş bir dosyayı ekle...
    ]
);
```
Benzer şekilde, `Laravel\Ai\Files\Image` sınıfı, bir sorguya görüntü eklemek için kullanılabilir:
```php
use App\Ai\Agents\ImageAnalyzer;
use Laravel\Ai\Files;

$response = (new ImageAnalyzer)->prompt(
    'Bu görselde ne var?',
    attachments: [
        Files\Image::fromStorage('photo.jpg'), // Bir dosya sistemi diskinden görüntü ekle...
        Files\Image::fromPath('/home/laravel/photo.jpg'), // Yerel bir yoldan görüntü ekle...
        $request->file('photo'), // Yüklenmiş bir dosyayı ekle...
    ]
);
```

### Akışla Gönderme (Streaming)
`stream` yöntemini çağırarak bir ajanın yanıtını akışla gönderebilirsiniz (stream). Döndürülen `StreamableAgentResponse`, istemciye otomatik olarak bir akış yanıtı (SSE) göndermek için bir rotadan döndürülebilir:
```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)->stream('Bu satış dökümünü analiz et...');
});
```
`then` metodu, yanıtın tamamı istemciye akışla gönderildiğinde çağrılacak bir closure sağlamak için kullanılabilir:
```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Responses\StreamedAgentResponse;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Bu satış dökümünü analiz et...')
        ->then(function (StreamedAgentResponse $response) {
            // $response->text, $response->events, $response->usage...
        });
});
```
Alternatif olarak, akışla gönderilen olaylar (events) arasında manuel olarak yineleme yapabilirsiniz:
```php
$stream = (new SalesCoach)->stream('Bu satış dökümünü analiz et...');

foreach ($stream as $event) {
    // ...
}
```

#### Vercel AI SDK Protokolünü Kullanarak Akışla Gönderme (Streaming Using the Vercel AI SDK Protocol)
Akışla gönderilebilir yanıt (streamable response) üzerinde `usingVercelDataProtocol` yöntemini çağırarak olayları Vercel AI SDK akış protokolünü kullanarak akışla gönderebilirsiniz:
```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Bu satış dökümünü analiz et...')
        ->usingVercelDataProtocol();
});
```

### Yayınlama (Broadcasting)
Akışla gönderilen olayları birkaç farklı şekilde yayınlayabilirsiniz. İlk olarak, akışla gönderilen bir olayda `broadcast` veya `broadcastNow` yöntemini çağırmanız yeterlidir:
```php
use App\Ai\Agents\SalesCoach;
use Illuminate\Broadcasting\Channel;

$stream = (new SalesCoach)->stream('Bu satış dökümünü analiz et...');

foreach ($stream as $event) {
    $event->broadcast(new Channel('channel-name'));
}
```
Veya, bir ajanın `broadcastOnQueue` yöntemini çağırarak ajan işlemini sıraya alabilir (queue) ve akışla gönderilen olayları kullanılabilir hale geldikçe yayınlayabilirsiniz:
```php
(new SalesCoach)->broadcastOnQueue(
    'Bu satış dökümünü analiz et...'
    new Channel('channel-name'),
);
```

### Sıraya Alma (Queueing)
Bir ajanın `queue` yöntemini kullanarak ajanı sorgulayabilir, ancak yanıtı arka planda işlemesine izin verebilir, böylece uygulamanızın hızlı ve duyarlı hissetmesini sağlayabilirsiniz. `then` ve `catch` yöntemleri, bir yanıt hazır olduğunda veya bir istisna oluştuğunda çağrılacak closure'ları kaydetmek için kullanılabilir:
```php
use Illuminate\Http\Request;
use Laravel\Ai\Responses\AgentResponse;
use Throwable;

Route::post('/coach', function (Request $request) {
    return (new SalesCoach)
        ->queue($request->input('transcript'))
        ->then(function (AgentResponse $response) {
            // ...
        })
        ->catch(function (Throwable $e) {
            // ...
        });

    return back();
});
```

### Araçlar (Tools)
Araçlar (tools), ajanlara, sorgulara yanıt verirken kullanabilecekleri ek işlevsellik vermek için kullanılabilir. Araçlar, `make:tool` Artisan komutu kullanılarak oluşturulabilir:
```bash
php artisan make:tool RandomNumberGenerator
```
Oluşturulan araç, uygulamanızın `app/Ai/Tools` dizinine yerleştirilecektir. Bir aracın ne yapacağını tanımlamak için, bir `handle` metodu tanımlamanız yeterlidir. Araçlar ayrıca bir `description` yöntemi tanımlayarak yapay zeka modeline ne yaptıkları hakkında bağlam sağlayabilir:
```php
<?php

namespace App\Ai\Tools;

use Laravel\Ai\Contracts\Tool;

class RandomNumberGenerator implements Tool
{
    /**
     * Aracın adını al.
     */
    public function name(): string
    {
        return 'random_number_generator';
    }

    /**
     * Aracın açıklamasını al.
     */
    public function description(): string
    {
        return 'Rastgele bir sayı üret.';
    }

    /**
     * Aracın parametrelerini tanımla.
     */
    public function parameters(): array
    {
        return [];
    }

    /**
     * Aracı çalıştır.
     */
    public function handle(): string
    {
        return (string) rand();
    }
}
```
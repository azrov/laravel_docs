# Laravel 12 Dokümantasyonu: Laravel Boost

## Giriş

Laravel Boost, yapay zeka ajanlarının (AI agents) Laravel en iyi uygulamalarına (best practices) bağlı kalarak yüksek kaliteli Laravel uygulamaları yazmalarına yardımcı olan temel yönergeleri (guidelines) ve ajan becerilerini (agent skills) sağlayarak yapay zeka destekli geliştirmeyi hızlandırır.

Boost ayrıca, yerleşik bir MCP aracını (built-in MCP tool) 17.000'den fazla Laravel'e özgü bilgi parçası içeren kapsamlı bir bilgi tabanıyla (knowledge base) birleştiren güçlü bir Laravel ekosistemi dokümantasyon API'si sağlar. Tüm bunlar, hassas, bağlam bilincinde sonuçlar (context-aware results) için yerleştirmeleri (embeddings) kullanan anlamsal arama (semantic search) yetenekleriyle geliştirilmiştir. Boost, Claude Code ve Cursor gibi yapay zeka ajanlarına, en son Laravel özellikleri ve en iyi uygulamaları hakkında bilgi edinmek için bu API'yi kullanmaları talimatını verir.

## Kurulum (Installation)

Laravel Boost, Composer aracılığıyla kurulabilir:
```bash
composer require laravel/boost --dev
```
Ardından, MCP sunucusunu ve kodlama yönergelerini (coding guidelines) yükleyin:
```bash
php artisan boost:install
```
`boost:install` komutu, kurulum işlemi sırasında seçtiğiniz kodlama ajanları için ilgili ajan yönergesi (agent guideline) ve beceri (skill) dosyalarını oluşturacaktır.

Laravel Boost kurulduktan sonra, Cursor, Claude Code veya seçtiğiniz yapay zeka ajanınızla kodlamaya başlamaya hazırsınız.

Oluşturulan MCP yapılandırma dosyasını (`.mcp.json`), yönerge dosyalarını (`CLAUDE.md`, `AGENTS.md`, `junie/`, vb.) ve `boost.json` yapılandırma dosyasını uygulamanızın `.gitignore` dosyasına eklemekte özgürsünüz, çünkü bu dosyalar `boost:install` ve `boost:update` çalıştırıldığında otomatik olarak yeniden oluşturulur.

### Ajanlarınızı (Agents) Kurun (Set Up Your Agents)

Aşağıda popüler yapay zeka araçları için Laravel Boost'un nasıl yapılandırılacağına dair hızlı bir kılavuz bulunmaktadır:

**Cursor için:**
1.  Komut paletini açın (`Cmd+Shift+P` veya `Ctrl+Shift+P`)
2.  "/open MCP Settings" üzerinde `enter` tuşuna basın
3.  `laravel-boost` için geçiş anahtarını açın

**Claude Code için:**
Claude Code desteği genellikle otomatik olarak etkinleştirilir. Etkin değilse, projenin dizininde bir kabuk (shell) açın ve aşağıdaki komutu çalıştırın:
```bash
claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

**Codex için:**
Codex desteği genellikle otomatik olarak etkinleştirilir. Etkin değilse, projenin dizininde bir kabuk açın ve aşağıdaki komutu çalıştırın:
```bash
codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

**Gemini CLI için:**
Gemini CLI desteği genellikle otomatik olarak etkinleştirilir. Etkin değilse, projenin dizininde bir kabuk açın ve aşağıdaki komutu çalıştırın:
```bash
gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

**GitHub Copilot için:**
1.  Komut paletini açın (`Cmd+Shift+P` veya `Ctrl+Shift+P`)
2.  "MCP: List Servers" üzerinde `enter` tuşuna basın
3.  `laravel-boost` öğesine gitmek için ok tuşlarını kullanın ve `enter` tuşuna basın
4.  "Start server" seçeneğini belirleyin

**Junie için:**
1.  Komut paletini açmak için `shift` tuşuna iki kez basın
2.  "MCP Settings" araması yapın ve `enter` tuşuna basın
3.  `laravel-boost` yanındaki kutuyu işaretleyin
4.  Sağ alttaki "Apply" düğmesine tıklayın

### Boost Kaynaklarını Güncel Tutma (Keeping Boost Resources Updated)
Yerel Boost kaynaklarınızı (AI yönergeleri ve becerileri) periyodik olarak güncellemek, yüklediğiniz Laravel ekosistemi paketlerinin en son sürümlerini yansıtmalarını sağlamak isteyebilirsiniz. Bunu yapmak için `boost:update` Artisan komutunu kullanabilirsiniz.
```bash
php artisan boost:update
```
Bu süreci, Composer "post-update-cmd" betiklerinize ekleyerek otomatikleştirebilirsiniz:
```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan boost:update --ansi"
        ]
    }
}
```

## MCP Sunucusu (MCP Server)

Laravel Boost, yapay zeka ajanlarının Laravel uygulamanızla etkileşime girmesi için araçlar (tools) kullanıma sunan bir MCP (Model Context Protocol) sunucusu sağlar. Bu araçlar, ajanlara uygulamanızın yapısını inceleme, veritabanını sorgulama, kod çalıştırma ve daha fazlasını yapma yeteneği verir.

### Mevcut MCP Araçları (Available MCP Tools)

| İsim | Notlar |
| :--- | :--- |
| Application Info | PHP ve Laravel sürümlerini, veritabanı motorunu, sürümleriyle birlikte ekosistem paketlerinin listesini ve Eloquent modellerini okur |
| Browser Logs | Tarayıcıdan logları ve hataları okur |
| Database Connections | Varsayılan bağlantı dahil olmak üzere mevcut veritabanı bağlantılarını inceler |
| Database Query | Veritabanına karşı bir sorgu yürütür |
| Database Schema | Veritabanı şemasını okur |
| Get Absolute URL | Göreceli yol URI'lerini mutlak olana dönüştürür, böylece ajanlar geçerli URL'ler oluşturabilir |
| Get Config | "Nokta" notasyonunu kullanarak yapılandırma dosyalarından bir değer alır |
| Last Error | Uygulamanın log dosyalarından son hatayı okur |
| List Artisan Commands | Mevcut Artisan komutlarını inceler |
| List Available Config Keys | Mevcut yapılandırma anahtarlarını inceler |
| List Available Env Vars | Mevcut ortam değişkeni anahtarlarını inceler |
| List Routes | Uygulamanın rotalarını inceler |
| Read Log Entries | Son N log girdisini okur |
| Search Docs | Yüklü paketlere göre dokümantasyon almak için Laravel tarafından barındırılan dokümantasyon API servisini sorgular |
| Tinker | Uygulama bağlamı içinde rastgele kod çalıştırır |

### MCP Sunucusunu Manuel Olarak Kaydetme (Manually Registering the MCP Server)
Bazen Laravel Boost MCP sunucusunu seçtiğiniz düzenleyiciye (editor) manuel olarak kaydetmeniz gerekebilir. MCP sunucusunu aşağıdaki ayrıntıları kullanarak kaydetmelisiniz:

| Özellik | Değer |
| :--- | :--- |
| Komut (Command) | `php` |
| Argümanlar (Args) | `artisan boost:mcp` |

JSON örneği:
```json
{
    "mcpServers": {
        "laravel-boost": {
            "command": "php",
            "args": ["artisan", "boost:mcp"]
        }
    }
}
```

## Yapay Zeka Yönergeleri (AI Guidelines)

Yapay zeka yönergeleri (AI guidelines), yapay zeka ajanlarına Laravel ekosistemi paketleri hakkında temel bağlam sağlamak için önceden yüklenen birleştirilebilir (composable) talimat dosyalarıdır. Bu yönergeler, ajanların tutarlı, yüksek kaliteli kod üretmesine yardımcı olan temel kuralları, en iyi uygulamaları ve framework'e özgü kalıpları içerir.

### Mevcut Yapay Zeka Yönergeleri (Available AI Guidelines)
Laravel Boost, aşağıdaki paketler ve framework'ler için yapay zeka yönergeleri içerir. `core` (temel) yönergeler, verilen paket için tüm sürümlerde geçerli olan genel, jenerik tavsiyeler sağlar.

| Paket | Desteklenen Sürümler |
| :--- | :--- |
| Core & Boost | core |
| Laravel Framework | core, 10.x, 11.x, 12.x |
| Livewire | core, 2.x, 3.x, 4.x |
| Flux UI | core, free, pro |
| Folio | core |
| Herd | core |
| Inertia Laravel | core, 1.x, 2.x |
| Inertia React | core, 1.x, 2.x |
| Inertia Vue | core, 1.x, 2.x |
| Inertia Svelte | core, 1.x, 2.x |
| MCP | core |
| Pennant | core |
| Pest | core, 3.x, 4.x |
| PHPUnit | core |
| Pint | core |
| Sail | core |
| Tailwind CSS | core, 3.x, 4.x |
| Livewire Volt | core |
| Wayfinder | core |
| Enforce Tests | koşullu |

Yapay zeka yönergelerinizi güncel tutmak için Boost Kaynaklarını Güncel Tutma bölümüne bakın.

### Özel Yapay Zeka Yönergeleri Ekleme (Adding Custom AI Guidelines)
Laravel Boost'u kendi özel yapay zeka yönergelerinizle zenginleştirmek için, uygulamanızın `.ai/guidelines/*` dizinine `.blade.php` veya `.md` dosyaları ekleyin. Bu dosyalar, `boost:install` komutunu çalıştırdığınızda Laravel Boost'un yönergelerine otomatik olarak dahil edilecektir.

### Boost Yapay Zeka Yönergelerini Geçersiz Kılma (Overriding Boost AI Guidelines)
Mevcut bir Boost yönergesi yoluyla (path) eşleşen kendi özel yönergelerinizi oluşturarak Boost'un yerleşik yapay zeka yönergelerini geçersiz kılabilirsiniz (override). Mevcut bir Boost yönergesi yoluyla eşleşen özel bir yönerge oluşturduğunuzda, Boost yerleşik olan yerine sizin özel sürümünüzü kullanacaktır.

Örneğin, Boost'un "Inertia React v2 Form Guidance" yönergelerini geçersiz kılmak için, `.ai/guidelines/inertia-react/2/forms.blade.php` konumunda bir dosya oluşturun. `boost:install` komutunu çalıştırdığınızda, Boost varsayılan yönerge yerine sizin özel yönergenizi dahil edecektir.

### Üçüncü Taraf Paket Yapay Zeka Yönergeleri (Third-Party Package AI Guidelines)
Üçüncü taraf bir paket sürdürüyorsanız ve Boost'un bunun için yapay zeka yönergeleri içermesini istiyorsanız, bunu paketinize bir `resources/boost/guidelines/core.blade.php` dosyası ekleyerek yapabilirsiniz. Paketinizin kullanıcıları `php artisan boost:install` komutunu çalıştırdığında, Boost yönergelerinizi otomatik olarak yükleyecektir.

Yapay zeka yönergeleri, paketinizin ne yaptığına dair kısa bir genel bakış sağlamalı, gerekli dosya yapısını veya kurallarını (conventions) ana hatlarıyla belirtmeli ve ana özelliklerinin nasıl oluşturulacağını veya kullanılacağını (örnek komutlar veya kod parçacıkları ile) açıklamalıdır. Bunları kısa, uygulanabilir (actionable) ve en iyi uygulamalara odaklanmış tutun, böylece yapay zeka kullanıcılarınız için doğru kod üretebilir. İşte bir örnek:
```markdown
## Paket Adı

Bu paket, [işlevselliğin kısa açıklaması] sağlar.

### Özellikler

- Özellik 1: [açık ve kısa açıklama].
- Özellik 2: [açık ve kısa açıklama]. Örnek kullanım:

<code-snippet name="Özellik 2 nasıl kullanılır" lang="php">
$result = PaketAdi::ozellikIki($param1, $param2);
</code-snippet>
```

## Ajan Becerileri (Agent Skills)

Ajan Becerileri (Agent Skills), ajanların belirli alanlarda çalışırken ihtiyaç anında etkinleştirebilecekleri hafif, hedefe yönelik bilgi modülleridir. Önceden yüklenen yönergelerin (guidelines) aksine, beceriler yalnızca ilgili olduklarında yüklenerek ayrıntılı kalıpların ve en iyi uygulamaların kullanılmasına olanak tanır, bu da bağlam şişkinliğini (context bloat) azaltır ve yapay zeka tarafından oluşturulan kodun alaka düzeyini (relevance) artırır.

`boost:install` komutunu çalıştırdığınızda ve bir özellik olarak becerileri seçtiğinizde, `composer.json` dosyanızda algılanan paketlere göre beceriler otomatik olarak yüklenir. Örneğin, projeniz `livewire/livewire` içeriyorsa, `livewire-development` becerisi otomatik olarak kurulacaktır.

### Mevcut Beceriler (Available Skills)

| Beceri (Skill) | Paket (Package) |
| :--- | :--- |
| `fluxui-development` | Flux UI |
| `folio-routing` | Folio |
| `inertia-react-development` | Inertia React |
| `inertia-svelte-development` | Inertia Svelte |
| `inertia-vue-development` | Inertia Vue |
| `livewire-development` | Livewire |
| `mcp-development` | MCP |
| `pennant-development` | Pennant |
| `pest-testing` | Pest |
| `tailwindcss-development` | Tailwind CSS |
| `volt-development` | Volt |
| `wayfinder-development` | Wayfinder |

Becerilerinizi güncel tutmak için Boost Kaynaklarını Güncel Tutma bölümüne bakın.

### Özel Beceriler (Custom Skills)
Kendi özel becerilerinizi oluşturmak için, uygulamanızın `.ai/skills/{skill-name}/` dizinine bir `SKILL.md` dosyası ekleyin. `boost:update` komutunu çalıştırdığınızda, özel becerileriniz Boost'un yerleşik becerilerinin yanında kurulacaktır.

Örneğin, uygulamanızın alan mantığı (domain logic) için özel bir beceri oluşturmak için:
```
.ai/skills/creating-invoices/SKILL.md
```

### Becerileri Geçersiz Kılma (Overriding Skills)
Eşleşen adlara sahip kendi özel becerilerinizi oluşturarak Boost'un yerleşik becerilerini geçersiz kılabilirsiniz. Mevcut bir Boost beceri adıyla eşleşen özel bir beceri oluşturduğunuzda, Boost yerleşik olan yerine sizin özel sürümünüzü kullanacaktır.

Örneğin, Boost'un `livewire-development` becerisini geçersiz kılmak için, `.ai/skills/livewire-development/SKILL.md` konumunda bir dosya oluşturun. `boost:update` komutunu çalıştırdığınızda, Boost varsayılan beceri yerine sizin özel becerinizi dahil edecektir.

### Üçüncü Taraf Paket Becerileri (Third-Party Package Skills)
Üçüncü taraf bir paket sürdürüyorsanız ve Boost'un bunun için beceriler içermesini istiyorsanız, bunu paketinize `resources/boost/skills/{skill-name}/SKILL.md` dosyası ekleyerek yapabilirsiniz. Paketinizin kullanıcıları `php artisan boost:install` komutunu çalıştırdığında, Boost kullanıcı tercihine göre becerilerinizi otomatik olarak kuracaktır.

Boost Becerileri, Ajan Becerileri (Agent Skills) formatını destekler ve YAML ön maddesi (frontmatter) ve Markdown talimatları içeren bir `SKILL.md` dosyası barındıran bir klasör olarak yapılandırılmalıdır. `SKILL.md` dosyası gerekli ön maddeyi (`name` ve `description`) içermeli ve isteğe bağlı olarak betikler (scripts), şablonlar (templates) ve referans materyalleri içerebilir.

Beceriler, gerekli dosya yapısını veya kurallarını ana hatlarıyla belirtmeli ve ana özelliklerinin nasıl oluşturulacağını veya kullanılacağını (örnek komutlar veya kod parçacıkları ile) açıklamalıdır. Bunları kısa, uygulanabilir ve en iyi uygulamalara odaklanmış tutun, böylece yapay zeka kullanıcılarınız için doğru kod üretebilir:
```markdown
---
name: paket-adi-gelistirme
description: Bileşenler ve iş akışları dahil olmak üzere PaketAdı özellikleriyle oluşturun ve çalışın.
---

# Paket Adı Geliştirme

## Bu beceri ne zaman kullanılır
Bu beceriyi, PaketAdı özellikleriyle çalışırken kullanın...

## Özellikler

- Özellik 1: [açık ve kısa açıklama].
- Özellik 2: [açık ve kısa açıklama]. Örnek kullanım:

$result = PaketAdi::ozellikIki($param1, $param2);
```

## Yönergeler (Guidelines) ve Beceriler (Skills)

Laravel Boost, yapay zeka ajanlarına uygulamanız hakkında bağlam sağlamak için iki farklı yol sunar: yönergeler (guidelines) ve beceriler (skills).

*   **Yönergeler**, yapay zeka ajanı başladığında önceden yüklenir ve kod tabanınız genelinde geniş çapta uygulanan Laravel kuralları ve en iyi uygulamaları hakkında temel bağlam sağlar.
*   **Beceriler**, belirli görevler üzerinde çalışırken ihtiyaç anında etkinleştirilir ve belirli alanlar (Livewire bileşenleri veya Pest testleri gibi) için ayrıntılı kalıplar içerir. Becerileri yalnızca alakalı olduklarında yüklemek, bağlam şişkinliğini azaltır ve kod kalitesini artırır.

| Özellik | Yönergeler (Guidelines) | Beceriler (Skills) |
| :--- | :--- | :--- |
| Yükleme Zamanı | Önceden, her zaman mevcut | İhtiyaç anında, alakalı olduğunda |
| Kapsam (Scope) | Geniş, temel | Odaklanmış, göreve özgü |
| Amaç | Temel kurallar & en iyi uygulamalar | Ayrıntılı uygulama kalıpları |

## Dokümantasyon API'si (Documentation API)

Laravel Boost, yapay zeka ajanlarına 17.000'den fazla Laravel'e özgü bilgi parçası içeren kapsamlı bir bilgi tabanına erişim sağlayan bir Dokümantasyon API'si (Documentation API) içerir. API, hassas, bağlam bilincinde sonuçlar sağlamak için yerleştirmelerle (embeddings) anlamsal arama (semantic search) kullanır.

`Search Docs` MCP aracı, ajanların, yüklü paketlerinize göre dokümantasyon almak için Laravel tarafından barındırılan dokümantasyon API servisini sorgulamasına olanak tanır. Boost'un yapay zeka yönergeleri ve becerileri, kodlama ajanınıza bu API'yi kullanması için otomatik olarak talimat verecektir.

| Paket | Desteklenen Sürümler |
| :--- | :--- |
| Laravel Framework | 10.x, 11.x, 12.x |
| Filament | 2.x, 3.x, 4.x, 5.x |
| Flux UI | 2.x Free, 2.x Pro |
| Inertia | 1.x, 2.x |
| Livewire | 1.x, 2.x, 3.x, 4.x |
| Nova | 4.x, 5.x |
| Pest | 3.x, 4.x |
| Tailwind CSS | 3.x, 4.x |

## Boost'u Genişletme (Extending Boost)

Boost, birçok popüler IDE ve yapay zeka ajanıyla kutudan çıktığı gibi çalışır. Kodlama aracınız henüz desteklenmiyorsa, kendi ajanınızı oluşturabilir ve Boost ile entegre edebilirsiniz.

### Diğer IDE'ler / Yapay Zeka Ajanları için Destek Ekleme (Adding Support for Other IDEs / AI Agents)
Yeni bir IDE veya yapay zeka ajanı için destek eklemek üzere, `Laravel\Boost\Install\Agents\Agent` sınıfını genişleten (extend) bir sınıf oluşturun ve ihtiyacınıza bağlı olarak aşağıdaki sözleşmelerden (contracts) birini veya birkaçını uygulayın (implement):

*   `Laravel\Boost\Contracts\SupportsGuidelines` - Yapay zeka yönergeleri (AI guidelines) için destek ekler.
*   `Laravel\Boost\Contracts\SupportsMcp` - MCP için destek ekler.
*   `Laravel\Boost\Contracts\SupportsSkills` - Ajan Becerileri (Agent Skills) için destek ekler.

#### Ajanı Yazma (Writing the Agent)
```php
<?php

declare(strict_types=1);

namespace App;

use Laravel\Boost\Contracts\SupportsGuidelines;
use Laravel\Boost\Contracts\SupportsMcp;
use Laravel\Boost\Contracts\SupportsSkills;
use Laravel\Boost\Install\Agents\Agent;

class CustomAgent extends Agent implements SupportsGuidelines, SupportsMcp, SupportsSkills
{
    // Sizin uygulamanız...
}
```
Örnek bir uygulama için `ClaudeCode.php` dosyasına bakın.

#### Ajanı Kaydetme (Registering the Agent)
Özel ajanınızı, uygulamanızın `App\Providers\AppServiceProvider` sınıfının `boot` metodunda kaydedin:
```php
use Laravel\Boost\Boost;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Boost::extend(function ($boost) {
        $boost->registerAgent(CustomAgent::class);
    });
}
```
# Laravel 12 Dokümantasyonu: Sürüm Notları (Release Notes)

## Sürümleme Şeması (Versioning Scheme)

Laravel ve diğer birinci taraf (first-party) paketleri Anlamsal Sürümleme'yi (Semantic Versioning) takip eder. Ana framework sürümleri her yıl (~Q1) yayınlanırken, alt (minor) ve yama (patch) sürümler haftada bir kadar sık yayınlanabilir. Alt ve yama sürümler asla geriye dönük uyumsuzluk (breaking changes) içermemelidir.

Uygulamanızdan veya paketinizden Laravel framework'üne veya bileşenlerine başvururken, her zaman `^12.0` gibi bir sürüm kısıtlaması kullanmalısınız, çünkü Laravel'in ana sürümleri geriye dönük uyumsuz değişiklikler içerir. Ancak, her zaman bir gün veya daha kısa sürede yeni bir ana sürüme güncelleme yapabilmenizi sağlamaya çalışıyoruz.

#### Adlandırılmış Argümanlar (Named Arguments)
Adlandırılmış argümanlar (named arguments), Laravel'in geriye dönük uyumluluk yönergeleri (backwards compatibility guidelines) kapsamında değildir. Laravel kod tabanını iyileştirmek için gerektiğinde fonksiyon argümanlarını yeniden adlandırmayı seçebiliriz. Bu nedenle, Laravel metotlarını çağırırken adlandırılmış argümanlar kullanmak dikkatli bir şekilde ve parametre adlarının gelecekte değişebileceğini anlayarak yapılmalıdır.

## Destek Politikası (Support Policy)

Tüm Laravel sürümleri için hata düzeltmeleri (bug fixes) 18 ay ve güvenlik düzeltmeleri (security fixes) 2 yıl boyunca sağlanır. Ek kütüphaneler için yalnızca en son ana sürüm hata düzeltmeleri alır. Ek olarak, Laravel tarafından desteklenen veritabanı sürümlerini inceleyin.

| Sürüm | PHP (*) | Çıkış Tarihi | Hata Düzeltmeleri Süresi | Güvenlik Düzeltmeleri Süresi |
| :--- | :--- | :--- | :--- | :--- |
| 10 | 8.1 - 8.3 | 14 Şubat 2023 | 6 Ağustos 2024 | 4 Şubat 2025 |
| 11 | 8.2 - 8.4 | 12 Mart 2024 | 3 Eylül 2025 | 12 Mart 2026 |
| 12 | 8.2 - 8.5 | 24 Şubat 2025 | 13 Ağustos 2026 | 24 Şubat 2027 |
| 13 | 8.3 - 8.5 | 2026 Q1 | 2027 Q3 | 2028 Q1 |

*   `Kırmızı` renk: Ömrünü tamamlamış (End of life)
*   `Sarı` renk: Yalnızca güvenlik düzeltmeleri (Security fixes only)
*   (*) Desteklenen PHP sürümleri

## Laravel 12

Laravel 12, yukarı akış bağımlılıklarını (upstream dependencies) güncelleyerek ve React, Svelte, Vue ve Livewire için kullanıcı kimlik doğrulaması için WorkOS AuthKit seçeneğini içeren yeni başlangıç kitleri (starter kits) sunarak Laravel 11.x'te yapılan iyileştirmeleri sürdürüyor. Başlangıç kitlerimizin WorkOS çeşidi, sosyal kimlik doğrulama (social authentication), geçiş anahtarları (passkeys) ve SSO desteği sunar.

### Minimum Geriye Dönük Uyumsuz Değişiklik (Minimal Breaking Changes)
Bu sürüm döngüsü sırasında odağımızın büyük bir kısmı, geriye dönük uyumsuz değişiklikleri (breaking changes) en aza indirmek oldu. Bunun yerine, kendimizi yıl boyunca mevcut uygulamaları bozmayan, sürekli yaşam kalitesi iyileştirmeleri (continuous quality-of-life improvements) yapmaya adadık.

Bu nedenle, Laravel 12 sürümü, mevcut bağımlılıkları yükseltmek için nispeten küçük bir "bakım sürümüdür" (maintenance release). Bu ışıkta, çoğu Laravel uygulaması herhangi bir uygulama kodunu değiştirmeden Laravel 12'ye yükseltme yapabilir.

### Yeni Uygulama Başlangıç Kitleri (New Application Starter Kits)
Laravel 12, React, Svelte, Vue ve Livewire için yeni uygulama başlangıç kitleri sunar. React, Svelte ve Vue başlangıç kitleri Inertia 2, TypeScript, shadcn/ui ve Tailwind'i kullanırken, Livewire başlangıç kitleri Tailwind tabanlı Flux UI bileşen kütüphanesini ve Laravel Volt'u kullanır.

React, Svelte, Vue ve Livewire başlangıç kitlerinin tümü, giriş, kayıt, parola sıfırlama, e-posta doğrulama ve daha fazlasını sunmak için Laravel'in yerleşik kimlik doğrulama sistemini kullanır. Ayrıca, her başlangıç kitinin WorkOS AuthKit destekli bir çeşidini sunuyoruz. Bu, sosyal kimlik doğrulama, geçiş anahtarları ve SSO desteği sağlar. WorkOS, aylık 1 milyona kadar aktif kullanıcısı olan uygulamalar için ücretsiz kimlik doğrulama sunar.

Yeni uygulama başlangıç kitlerimizin tanıtımıyla birlikte, Laravel Breeze ve Laravel Jetstream artık ek güncelleme almayacaktır.

Yeni başlangıç kitlerimizle başlamak için başlangıç kiti dokümantasyonuna göz atın.
# Zihin — Mental Performance & Digital Balance

**Zihin**, 15-25 yaş arası gençlere yönelik, freemium modelli bir **Zihinsel Performans ve Dijital Denge** uygulamasıdır. Kullanıcının günlük dijital alışkanlıklarını takip eder, yapay zeka destekli öneriler sunar ve zihinsel durumu yansıtan bir **Mind City** dioraması ile görsel motivasyon sağlar.

> Utandırma yok. Destekleyici, motive edici, somut adımlar.

---

## Özellikler

| Modül | Açıklama |
|---|---|
| **Mind City Dioraması** | Kullanıcının zihinsel performansını yansıtan, skor bazlı canlı şehir görselleştirmesi (CustomPaint). Skor arttıkça binalar yükselir, ışıklar artar; düştüğünde sis, çatlaklar ve hasar oluşur. |
| **Mental Skor** | Focus, Energy, Stability, Noise alt bileşenlerinden ağırlıklı hesaplanan 0-100 arası toplam skor. Ekran süresi verileri de skora etki eder. |
| **Ekran Süresi Takibi** | Android UsageStats API üzerinden gerçek uygulama kullanım verilerini çeker. Sosyal medya, video, oyun, mesajlaşma kategorilerinde dakika bazlı analiz. |
| **AI Koç (Gemini)** | Gemini 2.5 Flash modeli ile kişiselleştirilmiş günlük brifing, uygulama analizi, risk değerlendirmesi ve akıllı öneriler. 30 dk önbellekleme ile API kullanımı optimize edilmiştir. |
| **Dijital Detox** | AI destekli 7 günlük detox planları (kolay/orta/zor zorluk seviyeleri). Her gün için tema, hedef, görevler ve ipuçları içerir. |
| **Odaklanma Seansları** | Pomodoro tarzı 25/50/90 dakikalık odaklanma oturumları. Tamamlanan seanslar XP kazandırır ve skoru artırır. |
| **İyileşme Aktiviteleri** | Nefes egzersizi, günlük yazma, esneme ve farkındalık aktiviteleri. Hasar onarımı ve enerji restorasyonu sağlar. |
| **Kısıtlama Modülü** | Kategori bazlı günlük ekran süresi limitleri (sosyal medya, video, oyun, mesajlaşma). |
| **Detaylı Analiz** | Günlük/haftalık ekran süresi istatistikleri, uygulama bazlı kullanım dağılımı, kategori grafikleri. |
| **Bildirimler** | Günlük hatırlatıcı, ekran süresi uyarısı, seri kutlaması (3/7/14/30 gün), gece modu uyarısı. |
| **Premium (RevenueCat)** | Aylık/yıllık abonelik ile AI koç, gelişmiş analiz ve sınırsız detox planları. |
| **Onboarding** | 4 adımlı profil oluşturma: mod seçimi (öğrenci/genel), hedef belirleme, uygulama seçimi, bildirim ayarı. Tamamlandığında AI ile ilk analiz otomatik tetiklenir. |

---

## Teknik Mimari

### Offline-First, Event-Driven

- **Tüm veriler yerel SQLite veritabanında** saklanır — internet bağlantısı gerekmez.
- Şehir güncellemeleri **event-driven** — sürekli animasyon yok, pil dostu.
- Günlük max **3 ilerleme adımı** — bilgi bombardımanı yok.
- Analytics'te **PII yok** — kullanıcı kimliği hiçbir zaman dışarı çıkmaz.

### Katmanlı Mimari (Clean Architecture)

```
┌─────────────────────────────────────────┐
│  Presentation (Screens + Widgets)       │
│  └─ Riverpod Providers                  │
├─────────────────────────────────────────┤
│  Domain (Entities + Repository Intf.)   │
│  └─ ScoringEngine, CityStateMachine,    │
│     RecommendationEngine                │
├─────────────────────────────────────────┤
│  Data (Repository Impl + DataSources)   │
│  └─ SQLite, SharedPrefs, Firebase       │
├─────────────────────────────────────────┤
│  Services (AI, Analytics, Notifications,│
│            ScreenTime, Subscription)    │
└─────────────────────────────────────────┘
```

---

## Tech Stack

| Kategori | Teknoloji |
|---|---|
| Framework | Flutter 3.7+ |
| State Management | Riverpod (manual providers) |
| Navigasyon | GoRouter |
| Yerel DB | sqflite |
| Key-Value | SharedPreferences |
| Backend | Firebase (Analytics, Crashlytics, Remote Config) |
| Yapay Zeka | Gemini 2.5 Flash (HTTP API) |
| Ödeme | RevenueCat (purchases_flutter + purchases_ui_flutter) |
| Animasyon | Rive (placeholder), CustomPaint (aktif) |
| Bildirimler | flutter_local_notifications |
| UI | Google Fonts, Shimmer |
| Platform Channel | Android UsageStats API (Kotlin) |

---

## Proje Yapısı

```
lib/
├── main.dart                          # Uygulama giriş noktası
├── app.dart                           # MaterialApp widget
├── app_router.dart                    # Tüm route tanımları
│
├── core/
│   ├── constants/
│   │   ├── app_constants.dart         # Uygulama sabitleri
│   │   └── scoring_constants.dart     # Skor ağırlıkları ve eşikleri
│   └── theme/
│       └── app_theme.dart             # Dark theme, renk paleti
│
├── domain/
│   ├── entities/                      # Veri modelleri (Equatable)
│   │   ├── user_profile.dart          # Kullanıcı profili
│   │   ├── daily_stats.dart           # Günlük istatistikler
│   │   ├── city_state.dart            # Şehir durumu
│   │   ├── app_usage.dart             # Uygulama kullanımı
│   │   ├── focus_session.dart         # Odaklanma seansı
│   │   ├── recovery_log.dart          # İyileşme kaydı
│   │   └── subscription_state.dart    # Abonelik durumu
│   ├── repositories/                  # Repository arayüzleri
│   ├── scoring/
│   │   ├── scoring_engine.dart        # Deterministik skor hesaplama
│   │   └── scoring_types.dart         # ScoringInput & ScoreResult
│   ├── city/
│   │   └── city_state_machine.dart    # Şehir durum makinesi
│   └── recommendations/
│       └── recommendation_engine.dart # Kural bazlı öneri motoru
│
├── data/
│   ├── datasources/local/
│   │   ├── app_database.dart          # SQLite (8 tablo, v2)
│   │   └── shared_prefs.dart          # Onboarding, streak, cache
│   └── repositories/                  # Repository implementasyonları
│
├── services/
│   ├── ai/gemini_service.dart         # Gemini AI (cache + retry)
│   ├── analytics/                     # Firebase Analytics (PII-free)
│   ├── crash/crash_service.dart       # Crashlytics
│   ├── notifications/                 # Yerel bildirimler
│   ├── screen_time/                   # Android platform channel
│   └── subscription/                  # RevenueCat
│
├── features/                          # Feature-based modüller
│   ├── home/                          # Ana sayfa + Mind City
│   ├── onboarding/                    # 4 adımlı profil oluşturma
│   ├── focus/                         # Odaklanma seansları
│   ├── recovery/                      # İyileşme aktiviteleri
│   ├── ai_coach/                      # AI koç ekranı
│   ├── detox/                         # Dijital detox planları
│   ├── insights/                      # Detaylı analiz
│   ├── limits/                        # Kullanım limitleri
│   └── settings/                      # Ayarlar + debug
│
└── shared/providers/
    └── app_providers.dart             # Global Riverpod providers
```

---

## Skor Sistemi

### Bileşenler ve Ağırlıklar

| Bileşen | Ağırlık | Kaynak |
|---|---|---|
| **Focus** | %35 | Odaklanma dakikaları + seans bonusu |
| **Energy** | %25 | Baz 70, gece cezası, iyileşme bonusu, ekran süresi cezası |
| **Stability** | %25 | Seri günler, seans tamamlama, iyileşme |
| **Quiet** | %15 | 100 - Noise (dikkat dağıtıcı + gece + ekran gürültüsü) |

### Ekran Süresi Etkisi

- **Energy:** 2 saat+ kullanımda -5, 3 saat+ -10, 4 saat+ -15 puan
- **Noise:** Dikkat dağıtıcı uygulamalar (sosyal medya + oyun + video) dakikası × 0.15 (max 30)

### Eşik Değerleri

| Seviye | Skor Aralığı | Şehir Durumu |
|---|---|---|
| Peak | 80-100 | Parlak, altın tonlu, 12+ bina, inşaat aktif |
| Thriving | 60-79 | Sıcak tonlar, 8-11 bina, az sis |
| Building | 40-59 | Nötr, 5-7 bina, orta sis |
| Struggling | 20-39 | Soğuk tonlar, 3-4 bina, yoğun sis |
| Critical | 0-19 | Karanlık, duman, çatlaklar, hasar |

---

## Veritabanı Şeması

SQLite veritabanı `zihin.db` (versiyon 2):

| Tablo | Açıklama |
|---|---|
| `user_profiles` | Kullanıcı profili (mod, hedef, uygulamalar, bildirim) |
| `daily_stats` | Günlük performans metrikleri ve skorlar |
| `focus_sessions` | Odaklanma seans geçmişi |
| `recovery_logs` | İyileşme aktivite kayıtları |
| `city_states` | Şehir görselleştirme durumu (günlük) |
| `subscription_states` | Abonelik durumu önbelleği |
| `student_data` | Öğrenci moduna özel veriler |
| `app_usage` | Uygulama kullanım takibi (v2'de eklendi) |

---

## API Entegrasyonları

### Gemini AI

- **Model:** `gemini-2.5-flash`
- **Endpoint:** `generativelanguage.googleapis.com/v1beta`
- **Özellikler:**
  - Günlük brifing (4 saatlik periyotlarla cache)
  - Uygulama risk analizi (gün bazlı cache)
  - 7 günlük detox planı (zorluk + ortalama kullanıma göre cache)
  - Akıllı öneriler (4 saatlik periyotlarla cache)
- **Rate Limit:** 2 retry, exponential backoff (2s × attempt)
- **Cache:** 30 dakika TTL, max 20 entry (in-memory)

### RevenueCat

- **API Key:** Test store
- **Entitlement:** `Zihin - Pro`
- **Ürünler:** `monthly`, `yearly`, `lifetime`
- **Offline:** Cached entitlement status

### Firebase

- **Analytics:** PII-free event logging
- **Crashlytics:** Error tracking (production)
- **Remote Config:** Feature flags

---

## Android Platform Channel

`app.zihin.app/usage_stats` kanalı ile native Kotlin kodu üzerinden Android `UsageStatsManager` API'sine erişim:

```
hasPermission()       → bool
requestPermission()   → Settings açar
getDailyUsage(offset) → {totalMinutes, apps: [{packageName, appName, minutes, category}]}
getCategoryUsage()    → {social_media, gaming, video, messaging, other}
```

**Kategorizasyon:**
- Sosyal Medya: Instagram, TikTok, Twitter, Snapchat, Reddit, LinkedIn, BeReal, Discord...
- Video: YouTube, Netflix, Disney+, Twitch, Spotify, HBO...
- Oyun: Clash of Clans, Minecraft, Fortnite, Roblox, FIFA...
- Mesajlaşma: WhatsApp, Telegram, Slack, Teams, Viber...

---

## Ekranlar (Routes)

| Route | Ekran | Açıklama |
|---|---|---|
| `/splash` | SplashScreen | Başlangıç yükleme |
| `/onboarding` | OnboardingScreen | İlk kurulum (4 adım) |
| `/home` | HomeScreen | Ana sayfa + Mind City + skor |
| `/focus` | FocusScreen | Pomodoro odaklanma |
| `/recovery` | RecoveryScreen | İyileşme aktiviteleri |
| `/ai-coach` | AiCoachScreen | AI koç ve analiz |
| `/detox` | DetoxScreen | Dijital detox planları |
| `/insights` | InsightsScreen | Kullanım analizi |
| `/limits` | LimitsScreen | Kategori limitleri |
| `/settings` | SettingsScreen | Ayarlar + debug |

---

## Debug Özellikleri

Debug modunda (development build) aktif olan özellikler:

- **Şehir Kontrolleri:** Ana sayfada Mind City altında bina ekleme/çıkarma, seviye yükseltme, hasar ayarlama, preset seviyeleri (Peak/Good/Orta/Kötü) butonları
- **Profil Sıfırlama:** Settings > Debug > "Profili Sıfırla" ile tüm verileri silip onboarding'e dönme
- **Günlük Veri Temizleme:** Settings > Debug > "Günlük Veriyi Sıfırla"

---

## Kurulum

### Gereksinimler

- Flutter SDK ≥ 3.7.2
- Android Studio / VS Code
- Android cihaz veya emülatör (min SDK 24)
- Firebase projesi (isteğe bağlı, çalışır halde)

### Adımlar

```bash
# 1. Bağımlılıkları yükle
flutter pub get

# 2. .env dosyasını oluştur (veya mevcut olanı düzenle)
# GEMINI_API_KEY, REVENUECAT_API_KEY_ANDROID, vb.

# 3. Android cihazda çalıştır
flutter run -d <device_id>

# 4. Web'de çalıştır (kısıtlı — ekran süresi mock veri)
flutter run -d chrome
```

### Firebase Kurulumu

- `android/app/google-services.json` — Android Firebase config
- `ios/Runner/GoogleService-Info.plist` — iOS Firebase config

---

## Kurallar (Non-Negotiables)

1. **Offline-first**: Tüm veriler yerelde, internet opsiyonel
2. **Yerel veri**: PII hiçbir zaman dışarı çıkmaz
3. **Event-driven**: Sürekli animasyon yok, pil dostu
4. **Günlük max 3 adım**: Bilgi bombardımanı yok
5. **Shame-free**: Utandırma yok, destekleyici dil
6. **Hafif**: Ağır continuous animasyonlar yok

---

## Lisans

Özel proje — tüm hakları saklıdır.

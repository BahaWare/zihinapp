# Zihin — Mental Performance & Digital Balance

**Zihin** is a freemium **Mental Performance & Digital Balance** mobile app designed for young adults (ages 15-25). It tracks daily digital habits, provides AI-powered recommendations, and offers visual motivation through a **Mind City** diorama that reflects the user's mental state.

> No shaming. Supportive, motivating, actionable steps only.

---

## Features

| Module | Description |
|---|---|
| **Mind City Diorama** | A score-driven live city visualization (CustomPaint) that reflects mental performance. Buildings rise and lights brighten as the score improves; fog, cracks, and damage appear when it drops. |
| **Mental Score** | A weighted 0-100 total score computed from Focus, Energy, Stability, and Noise sub-components. Real screen time data directly impacts the score. |
| **Screen Time Tracking** | Pulls real app usage data via Android UsageStats API. Per-minute analysis across social media, video, gaming, and messaging categories. |
| **AI Coach (Gemini)** | Personalized daily briefings, app analysis, risk assessment, and smart recommendations using Gemini 2.5 Flash. API usage optimized with 30-minute response caching. |
| **Digital Detox** | AI-generated 7-day detox plans (easy/medium/hard difficulty). Each day includes a theme, goal, tasks, and tips. |
| **Focus Sessions** | Pomodoro-style 25/50/90-minute focus sessions. Completed sessions earn XP and boost the score. |
| **Recovery Activities** | Breathing exercises, journaling, stretching, and mindfulness activities. Repairs city damage and restores energy. |
| **Usage Limits** | Category-based daily screen time limits (social media, video, gaming, messaging). |
| **Detailed Analytics** | Daily/weekly screen time statistics, per-app usage breakdown, and category distribution charts. |
| **Notifications** | Daily reminders, screen time warnings, streak celebrations (3/7/14/30 days), and night mode alerts. |
| **Premium (RevenueCat)** | Monthly/yearly subscriptions unlock the AI coach, advanced analytics, and unlimited detox plans. |
| **Onboarding** | 4-step profile creation: mode selection (student/general), goal setting, app selection, notification setup. An initial AI analysis is automatically triggered on completion. |

---

## Architecture

### Offline-First, Event-Driven

- **All data is stored in a local SQLite database** — no internet connection required.
- City updates are **event-driven** — no continuous animations, battery-friendly.
- Maximum **3 daily progress steps** — no information overload.
- **No PII in analytics** — user identity never leaves the device.

### Layered Architecture (Clean Architecture)

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

| Category | Technology |
|---|---|
| Framework | Flutter 3.7+ |
| State Management | Riverpod (manual providers) |
| Navigation | GoRouter |
| Local DB | sqflite |
| Key-Value Store | SharedPreferences |
| Backend | Firebase (Analytics, Crashlytics, Remote Config) |
| AI | Gemini 2.5 Flash (HTTP API) |
| Payments | RevenueCat (purchases_flutter + purchases_ui_flutter) |
| Animation | Rive (placeholder), CustomPaint (active) |
| Notifications | flutter_local_notifications |
| UI | Google Fonts, Shimmer |
| Platform Channel | Android UsageStats API (Kotlin) |

---

## Project Structure

```
lib/
├── main.dart                          # App entry point
├── app.dart                           # MaterialApp widget
├── app_router.dart                    # All route definitions
│
├── core/
│   ├── constants/
│   │   ├── app_constants.dart         # App-wide constants
│   │   └── scoring_constants.dart     # Score weights & thresholds
│   └── theme/
│       └── app_theme.dart             # Dark theme, color palette
│
├── domain/
│   ├── entities/                      # Data models (Equatable)
│   │   ├── user_profile.dart          # User profile
│   │   ├── daily_stats.dart           # Daily statistics
│   │   ├── city_state.dart            # City state
│   │   ├── app_usage.dart             # App usage records
│   │   ├── focus_session.dart         # Focus session
│   │   ├── recovery_log.dart          # Recovery log
│   │   └── subscription_state.dart    # Subscription state
│   ├── repositories/                  # Repository interfaces
│   ├── scoring/
│   │   ├── scoring_engine.dart        # Deterministic score calculation
│   │   └── scoring_types.dart         # ScoringInput & ScoreResult
│   ├── city/
│   │   └── city_state_machine.dart    # City state machine
│   └── recommendations/
│       └── recommendation_engine.dart # Rule-based recommendation engine
│
├── data/
│   ├── datasources/local/
│   │   ├── app_database.dart          # SQLite (8 tables, v2)
│   │   └── shared_prefs.dart          # Onboarding, streak, cache
│   └── repositories/                  # Repository implementations
│
├── services/
│   ├── ai/gemini_service.dart         # Gemini AI (cache + retry)
│   ├── analytics/                     # Firebase Analytics (PII-free)
│   ├── crash/crash_service.dart       # Crashlytics
│   ├── notifications/                 # Local notifications
│   ├── screen_time/                   # Android platform channel
│   └── subscription/                  # RevenueCat
│
├── features/                          # Feature-based modules
│   ├── home/                          # Home screen + Mind City
│   ├── onboarding/                    # 4-step profile creation
│   ├── focus/                         # Focus sessions
│   ├── recovery/                      # Recovery activities
│   ├── ai_coach/                      # AI coach screen
│   ├── detox/                         # Digital detox plans
│   ├── insights/                      # Detailed analytics
│   ├── limits/                        # Usage limits
│   └── settings/                      # Settings + debug tools
│
└── shared/providers/
    └── app_providers.dart             # Global Riverpod providers
```

---

## Scoring System

### Components & Weights

| Component | Weight | Source |
|---|---|---|
| **Focus** | 35% | Focus minutes + session bonus |
| **Energy** | 25% | Base 70, late-night penalty, recovery bonus, screen time penalty |
| **Stability** | 25% | Streak days, session completion, recovery |
| **Quiet** | 15% | 100 - Noise (distractions + late night + screen noise) |

### Screen Time Impact

- **Energy:** -5 at 2h+, -10 at 3h+, -15 at 4h+ of daily screen time
- **Noise:** Distracting app minutes (social media + gaming + video) x 0.15 (capped at 30)

### Threshold Levels

| Level | Score Range | City State |
|---|---|---|
| Peak | 80-100 | Bright, golden tones, 12+ buildings, active construction |
| Thriving | 60-79 | Warm tones, 8-11 buildings, light fog |
| Building | 40-59 | Neutral, 5-7 buildings, moderate fog |
| Struggling | 20-39 | Cold tones, 3-4 buildings, heavy fog |
| Critical | 0-19 | Dark, smoke, cracks, visible damage |

---

## Database Schema

SQLite database `zihin.db` (version 2):

| Table | Description |
|---|---|
| `user_profiles` | User profile (mode, goal, apps, notifications) |
| `daily_stats` | Daily performance metrics and scores |
| `focus_sessions` | Focus session history |
| `recovery_logs` | Recovery activity records |
| `city_states` | City visualization state (daily) |
| `subscription_states` | Subscription status cache |
| `student_data` | Student-mode specific data |
| `app_usage` | App usage tracking (added in v2) |

---

## API Integrations

### Gemini AI

- **Model:** `gemini-2.5-flash`
- **Endpoint:** `generativelanguage.googleapis.com/v1beta`
- **Features:**
  - Daily briefing (cached in 4-hour windows)
  - App risk analysis (cached per day)
  - 7-day detox plan (cached by difficulty + average usage)
  - Smart recommendations (cached in 4-hour windows)
- **Rate Limiting:** 2 retries with exponential backoff (2s x attempt)
- **Cache:** 30-minute TTL, max 20 entries (in-memory)

### RevenueCat

- **API Key:** Test store
- **Entitlement:** `Zihin - Pro`
- **Products:** `monthly`, `yearly`, `lifetime`
- **Offline:** Cached entitlement status

### Firebase

- **Analytics:** PII-free event logging
- **Crashlytics:** Error tracking (production)
- **Remote Config:** Feature flags

---

## Android Platform Channel

Access to the Android `UsageStatsManager` API via native Kotlin code through the `app.zihin.app/usage_stats` channel:

```
hasPermission()       → bool
requestPermission()   → Opens system settings
getDailyUsage(offset) → {totalMinutes, apps: [{packageName, appName, minutes, category}]}
getCategoryUsage()    → {social_media, gaming, video, messaging, other}
```

**App Categorization:**
- Social Media: Instagram, TikTok, Twitter, Snapchat, Reddit, LinkedIn, BeReal, Discord...
- Video: YouTube, Netflix, Disney+, Twitch, Spotify, HBO...
- Gaming: Clash of Clans, Minecraft, Fortnite, Roblox, FIFA...
- Messaging: WhatsApp, Telegram, Slack, Teams, Viber...

---

## Screens (Routes)

| Route | Screen | Description |
|---|---|---|
| `/splash` | SplashScreen | Initial loading |
| `/onboarding` | OnboardingScreen | First-time setup (4 steps) |
| `/home` | HomeScreen | Main dashboard + Mind City + score |
| `/focus` | FocusScreen | Pomodoro focus sessions |
| `/recovery` | RecoveryScreen | Recovery activities |
| `/ai-coach` | AiCoachScreen | AI coach & analysis |
| `/detox` | DetoxScreen | Digital detox plans |
| `/insights` | InsightsScreen | Usage analytics |
| `/limits` | LimitsScreen | Category limits |
| `/settings` | SettingsScreen | Settings + debug tools |

---

## Debug Features

Available only in debug mode (development builds):

- **City Controls:** Buttons below the Mind City on the home screen for adding/removing buildings, upgrading levels, adjusting damage, and preset levels (Peak/Good/Medium/Bad)
- **Profile Reset:** Settings > Debug > "Reset Profile" to wipe all data and return to onboarding
- **Daily Data Wipe:** Settings > Debug > "Reset Daily Data"

---

## Setup

### Requirements

- Flutter SDK >= 3.7.2
- Android Studio / VS Code
- Android device or emulator (min SDK 24)
- Firebase project (optional, works without it)

### Steps

```bash
# 1. Install dependencies
flutter pub get

# 2. Create or edit the .env file
# GEMINI_API_KEY, REVENUECAT_API_KEY_ANDROID, etc.

# 3. Run on an Android device
flutter run -d <device_id>

# 4. Run on web (limited — screen time uses mock data)
flutter run -d chrome
```

### Firebase Setup

- `android/app/google-services.json` — Android Firebase config
- `ios/Runner/GoogleService-Info.plist` — iOS Firebase config

---

## Non-Negotiable Rules

1. **Offline-first**: All data stored locally, internet is optional
2. **Local data**: PII never leaves the device
3. **Event-driven**: No continuous animations, battery-friendly
4. **Max 3 daily steps**: No information overload
5. **Shame-free**: No shaming language, supportive tone only
6. **Lightweight**: No heavy continuous animations

---

## License

Private project — all rights reserved.


# (LAN: TR) Zihin — Mental Performance & Digital Balance

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

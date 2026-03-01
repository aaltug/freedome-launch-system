# FREEedome - UTM Yapısı ve Otomasyon Mimarisi

Bu doküman, Lansman Sisteminin dış trafiği içeri alma (UTM) ve içerideki lead'i bir viral döngüye çevirme (Otomasyon) sistemini teknik seviyede açıklar.

---

## 1. Mükemmel UTM Yapılandırma Formatı

Dışarıdaki (Paid ve Organic) her link için referans sözlüğü:

```
https://freedome.app/?utm_source=meta&utm_medium=paid_social&utm_campaign=freedome_launch_d1&utm_content=hook1_pain&utm_term=smb_owner
```

*   `utm_source`: `meta` | `tiktok` | `linkedin` | `email` (Trafiğin doğduğu yer)
*   `utm_medium`: `paid_social` | `organic` | `referral` (Para ödenip ödenmediği/format)
*   `utm_campaign`: `freedome_launch_d1` | `freedome_launch_d2` (Hangi günün kohortu)
*   `utm_content`: `hook1_pain` | `hook2_identity` | `hook3_fomo` (A/B Test Kazananı Belirlemek İçin En Kritik Parametre)
*   `utm_term`: `smb_owner` | `growth_marketer` | `agency_owner` (Hedefleme/Kitle kırılımı)

---

## 2. Sistem Mimarisi ve Otomasyon Akış Diyagramı

Form gönderildiği andan itibaren başlayan tetikleyici zincirinin vizualizasyonu.
*(Grafiği görüntülemek için markdown önizleyicisini kullanabilirsiniz)*

```mermaid
graph TD
    classDef trigger fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#fff;
    classDef process fill:#1e293b,stroke:#818cf8,stroke-width:1px,color:#fff;
    classDef decision fill:#312e81,stroke:#a5b4fc,stroke-width:2px,color:#fff;
    classDef positive fill:#064e3b,stroke:#34d399,stroke-width:2px,color:#fff;
    classDef negative fill:#7f1d1d,stroke:#f87171,stroke-width:1px,color:#fff;
    classDef viral fill:#1e1b4b,stroke:#c084fc,stroke-width:3px,color:#fff;

    A["Form Gönderildi (Tally/Typeform)"]:::trigger
    B{"CRM Lead Puanı (HubSpot)"}:::decision
    
    C["Kalite Skoru > 60"]:::positive
    D["Kalite Skoru <= 60"]:::negative
    
    A --> B
    B --> C
    B --> D
    
    D --> E["HubSpot Nurture Sekansı (7 Gün / 3 E-posta)"]:::process
    
    C --> F["Onay E-postası + Trial Aktivasyon (Make/Zapier)"]:::process
    
    F --> G{"E-posta Açıldı mı? (D+1)"}:::decision
    
    G -- Hayır --> H["Satış Temsilcisi: LinkedIn DM veya SMS Tetikle"]:::process
    G -- Evet --> I{"Aktivasyon / Kayıt Yapıldı mı?"}:::decision
    
    I -- Evet --> J["Slack Alert: Yeni Aktif Lead + Onboarding Maili"]:::trigger
    I -- Hayır --> E
    
    J --> K["'Arkadaşını Davet Et' Maili Gönder (Viral Loops)"]:::viral
    
    K -- Link Paylaşıldı --> L["Kullanıcıya Ekstra Kredi / Ödül Ver"]:::positive
```

### Akış Detayları ve Kullanılacak Araçlar (Tech Stack)

Aşağıdaki tablo, form gönderiminden aktivasyona kadar olan tetikleyici zincirinin araç bazlı mimarisini tanımlar.

| # | Tetikleyici | Aksiyon | Araç / Platform |
| :--- | :--- | :--- | :--- |
| **1** | Form gönderildi | Lead CRM'e düşer, skor atanır | Tally / Typeform → Zapier → HubSpot |
| **2** | Skor > 60 | Onay e-postası + trial aktivasyon linki gönder | HubSpot Workflow veya Make |
| **3** | Skor ≤ 60 | Nurture sekansına al (3 e-posta / 7 gün) | HubSpot / Brevo Sequence |
| **4** | E-posta açılmadı (D+1) | SMS veya LinkedIn DM tetikle | Twilio veya PhantomBuster |
| **5** | Trial aktivasyon yapıldı | Satış ekibine bildirim + onboarding maili | Slack alert + HubSpot Task |
| **6** | Referral linki paylaşıldı | Paylaşan kişiye ekstra kredi / ödül | Viral Loops veya ReferralHero |


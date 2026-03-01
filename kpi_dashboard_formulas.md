# FREEedome - KPI Dashboard ve Sinyal Formülleri

Bu doküman, Lansman Kampanyasının ölçümlenmesi için talep edilen Google Sheet hiyerarşisini ve insan müdahalesine gerek bırakmayan "Karar Sinyalleri" otomasyonunu (Çift Sinyal Kuralı) formülize eder.

## Google Sheet Yapısı ve Hücre Başlıkları

Dört temel sekme bulunur: `Dashboard`, `Lead Log`, `Kreatif Performans`, `Karar Notu`.

### Sekme 1: Dashboard (Günlük Özet)
Bu sekme, tüm operasyonun genel sağlığını gösterir. Formüllerle beslenir.
- **A Sütunu:** Tarih/Saat
- **B Sütunu:** Ziyaretçi Sayısı
- **C Sütunu:** Landing Page CTR (Tıklama Oranı - % formatında)
- **D Sütunu:** Lead Form Dolum Oranı (%)
- **E Sütunu:** Ortalama Lead Kalite Skoru 
- **F Sütunu:** Aktivasyon (D+1)
- **G Sütunu:** Sistem Tavsiyesi *(Otomatik Karar Sinyali Hücresi)*

---

## Karar Formülleri ve Sinyal Algoritmaları ("G Sütunu" İçin)

Dokümanın `3.3 Ölçeklendirme ve Optimizasyon Sinyalleri` bölümündeki "Çift Sinyal Kuralı" bu alanda Sheets/Excel formülüne dönüştürülmüştür. Bu hücrelerin **koşullu biçimlendirme** (Yeşil=Ölçekle, Kırmızı/Sarı=Optimize Et) ile görselleştirilmesi hedeflenir.

### 1. CTR ve Form Dolum Oranı Karar Matrisi
- *Kural:* Landing CTR > %5 **VE** form dolum > %30 ise → **ÖLÇEKLE**
- *Kural:* Landing CTR < %2 ise → **OPTİMİZE ET (Hero/CTA Değiş)**
- *Kural:* Form dolum < %15 ise → **OPTİMİZE ET (Form Alanı Azalt)**

> **Hücre (G2) Formülü:**
```excel
=IFS(
  AND(C2 > 0.05; D2 > 0.30); "🟢 ÖLÇEKLE (Budçeyi Artır)",
  C2 < 0.02; "🔴 OPTİMİZE ET (Hero Metni/CTA Değiştir)",
  D2 < 0.15; "🔴 OPTİMİZE ET (Form Sürtünmesini Azalt)",
  TRUE; "🟡 İZLEMEDE KAL"
)
```

### 2. Maliyet (CPL) ve Aktivasyon Karar Matrisi
*(Dashboard'da "H" ve "I" sütunlarında "Avg. CPL" ve "Aktivasyon Oranı" olduğu varsayılmıştır)*
- *Kural:* CPL < $6 **VE** aktivasyon oranı > %45 ise → **ÖLÇEKLE**
- *Kural:* CPL > $15 ise → **OPTİMİZE ET (Hedefleme/Kreatif Daralt)**
- *Kural:* Aktivasyon < %30 ise → **OPTİMİZE ET (Onboarding Yeniden Yaz)**

> **Hücre (H2) Formülü:**
```excel
=IFS(
  AND(CPL_Hücresi < 6; Aktivasyon_Hücresi > 0.45); "🟢 ÖLÇEKLE (Sistem Kârlı)",
  CPL_Hücresi > 15; "🔴 KAMPANYAYI DURDUR / DARALT",
  Aktivasyon_Hücresi < 0.30; "🟡 ONBOARDING E-POSTASINI İYİLEŞTİR",
  TRUE; "🟡 İZLEMEDE KAL"
)
```

### 3. Viral Katsayı ve Referral (Çift Sinyal Örneği)
Kural, tek bir sinyalden ziyade diğer sekmelerin de onaylanmasını gerektiriyordu.

```excel
=EĞER(Referral_Katsayısı > 0.20; 
  "🟢 VİRAL ETKİ BAŞLADI (Referral Kampanyasını Ön Plana Çıkar)"; 
  "🟡 OPTİMİZE T (Ödül/Motivasyon Yetersiz)"
)
```

---

## Kreatif Performans ve A/B Test Sinyalleri (Sekme 3)

Bu sekme, Görev 2'deki Başlık varyasyonlarının (V1 = Otorite, V2 = Hız, V3 = FOMO) karşılaştırılmasını otomatikleştirir. 

Varsayım:
*   A Sütunu: Hook Adı (Örn: V3 - FOMO/Kıskançlık)
*   B Sütunu: CTR
*   C Sütunu: CPL ($)
*   D Sütunu: Video Tamamlanma (Skip Oranı)

> **"Kazanan Kreatifi Seç" Formülü:**
*(En yüksek CTR ile en düşük CPL'ı birlikte test eden ağırlıklı puanlama)*
```excel
=EĞER(VE(B2>MAX($B$2:$B$4)*0.9; C2<MIN($C$2:$C$4)*1.1); "👑 KAZANAN VARYANT"; "❌ KAPAT")
```
*Açıklama:* Eğer bu hook'un CTR'ı en iyi CTR'ın %90'ından büyükse ve CPL'i en düşük CPL'den %10 marjıyla geride değilse kazanan ilan edilir.

---

## 3.2 Günlük Karar Notu Akışı (EOD - Gün Sonu)
Gün sonu notlarında kullanılacak çerçeve her zaman üç parçalı olmalıdır:
1. **Sinyal (Signal):** Sistem bize ne söylüyor? (Örn; Form dolum %12'ye düştü.)
2. **Kabul Ettiğimiz Neden (Hypothesis):** Neden oldu? (Örn: 3. opsiyonel hücre kafa karıştırıyor.)
3. **Eylem (Action / Rx):** Yarın neyi değiştireceğiz? (Örn: Bütçe hücresini kaldır, A/B teste sok.)


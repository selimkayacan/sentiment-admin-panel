# Sentiment Platform — Admin Panel

## Ne yapar
Tek sayfalık statik HTML (GitHub Pages). Supabase anon key ile doğrudan DB'ye bağlanır. Sunucuya ihtiyaç yok.

**URL:** https://selimkayacan.github.io/sentiment-admin-panel

## Sayfalar
| Sayfa | data-page | Açıklama |
|-------|-----------|----------|
| Müşteriler | `customers` | Müşteri listesi + detay (markalar, lokasyonlar, kullanıcılar, connectorlar) |
| Rakip Markalar | `competitors` | Global phantom brand listesi |
| Maliyet | `cost` | Marka bazında aylık API maliyet özeti |
| Ayarlar | `settings` | Supabase kredensiyelleri + servis URL'leri |

## Müşteri detay akışı
```
Müşteri → Markalar (admin ekler) → Şube Ekle (location_wizard.html, brand_id ile açılır)
       → Lokasyonlar (tüm markalar altındaki şubeler, marka kolonu var)
       → Kullanıcılar
       → Connector Erişimleri
       → Otomatik Çekim (sadece SaaS Full)
```

## Önemli state değişkenleri
```js
let currentCustomer = null;   // açık müşteri objesi
let allBrands = [];            // müşterinin markaları (loadBrands ile dolar)
let allCustomers = [];
let allModes = [];
let allConnectors = [];
```

## Temel fonksiyonlar
| Fonksiyon | Açıklama |
|-----------|----------|
| `openCustomer(id)` | Müşteri detayını açar, loadBrands + loadLocations + loadUsers... çağırır |
| `loadBrands(customerId)` | brands tablosundan `is_phantom=false` markalar |
| `createBrand()` | modal-new-brand formundan marka oluşturur |
| `deleteBrand(brandId)` | Markayı + şubelerini siler |
| `openLocationWizardForBrand(brandId, name)` | location_wizard.html'i `brand_id` param ile açar |
| `loadLocations(customerId)` | Tüm marka şubelerini marka adıyla listeler |
| `loadCostPage()` | cost_records → brands join, marka bazında gruplar |

## Maliyet sayfası
- `cost_records` tablosunu `brands(name)` join ile çeker
- Gruplandırma: `customerName || brandName` composite key
- cost_type değerleri: `claude_api`, `google_places_api`, `serpapi`
- Para birimi: TRY (₺)

## Supabase bağlantısı
Kredensiyeller index.html içinde hardcode:
```js
const SUPABASE_URL      = 'https://escjdzsfxzwvshapzugb.supabase.co';
const SUPABASE_ANON_KEY = '...';
```
Sistem URL'leri (gmaps_url, eksi_url, twitter_url, analysis_url, app_secret...) `system_settings` tablosunda.

## Location Wizard
`location_wizard.html` — ayrı bir sayfa, popup olarak açılır.
- `?brand_id=XXX&brand_name=YYY` parametresiyle marka bazında çalışır
- `?customer_id=XXX` ile de çalışır (geriye dönük uyumluluk)
- Kayıt sonrası `postMessage({type:'locations_saved', count:N})` gönderir

## Son değişiklikler
- Markalar bölümü eklendi (admin markayı oluşturuyor, müşteri değil)
- Lokasyonlar tablosuna Marka kolonu eklendi
- Maliyet sayfası marka bazına taşındı + SerpAPI kolonu eklendi
- Yeni Marka modalı eklendi (`modal-new-brand`)

# 🛠️ UstaPlatform - Şehrin Uzmanlık Platformu

**Ders:** Nesne Yönelimli Programlama (NYP) ve İleri C#  
**Proje Adı:** UstaPlatform  
**Amaç:** Arcadia şehrindeki kayıp uzmanları (tesisatçı, elektrikçi, marangoz vb.) vatandaş talepleriyle eşleştiren, dinamik fiyatlama ve akıllı rota planlama yapabilen, genişletilebilir bir yazılım platformu geliştirmek.

---

## 🚩 1. Proje Hikayesi ve Problem Özeti

**Bağlam:**  
Arcadia adlı şehirde belediyenin "Uzman Kayıt Sistemi" çökmüştür. Vatandaşlar artık uygun uzmanlara ulaşamamakta, ustalar ise iş planlarını etkin şekilde yönetememektedir.

**Sorun:**  
- Vatandaşlar doğru ustayı bulamıyor.  
- Fiyat tahmini ve planlama yok.  
- Mevcut sistem tek tip ve genişletilemez.

**Çözüm:**  
UstaPlatform, vatandaşların taleplerini en uygun ustayla eşleştiren, fiyatlandırmayı dinamik olarak hesaplayan, rotalama yapan ve yeni kurallarla genişletilebilen bir uygulama sunar.

---

## 🧠 2. Sistem Mimarisi ve Varlıklar

**Temel İş Akışı:**
1. Vatandaş talep açar.  
2. Sistem uygun ustayı bulur.  
3. İş Emri oluşturulur.  
4. Fiyat hesaplanır.  
5. Usta’nın çizelgesine ve rotasına eklenir.

| **Varlık** | **Açıklama** | **Ana Sorumluluk** |
|-------------|--------------|--------------------|
| **Usta (Master)** | Hizmet veren uzman. | Uzmanlık alanı, puan, yoğunluk. |
| **Vatandaş (Citizen)** | Hizmet talep eden kişi. | Talep oluşturma. |
| **Talep (Request)** | Vatandaşın açtığı iş talebi. | İşi ve zamanı tanımlama. |
| **İş Emri (WorkOrder)** | Onaylanmış ve ustaya atanmış iş. | Fiyat, zaman, rota. |
| **Fiyat Kuralı (IPricingRule)** | Fiyatı dinamik değiştiren eklenti. | Fiyat motoruna (engine) takılabilir. |
| **Rota (Route)** | Ustanın günlük adres sırası. | IEnumerable arayüzüyle gezilebilir koleksiyon. |
| **Çizelge (Schedule)** | Ustaların iş planı. | Dizinleyici (Indexer) içerir. |

---

## 🧩 3. Teknik Gereklilikler

### A. SOLID Prensipleri
- **SRP (Tek Sorumluluk):** Her sınıf tek bir işi yapar.  
- **OCP (Açık/Kapalı):** Yeni fiyat kuralı eklemek için ana uygulama yeniden derlenmez; sadece yeni DLL eklenir.  
- **DIP (Bağımlılıkların Tersine Çevrilmesi):** Üst katmanlar arayüzlere bağımlıdır (örneğin `IPricingRule`, `IWorkOrderRepository`).

### B. C# Özellikleri
- **init-only** özelliklerle kimlik (ID) gibi alanlar değiştirilemez.  
- **Nesne ve Koleksiyon Başlatıcılar** okunaklı test verisi sağlar.  
- **Dizinleyici:** `Schedule[DateOnly gün]` ile günlük iş emirlerine erişim.  
- **Özel IEnumerable<T>:** `Route` sınıfı `IEnumerable<(int X, int Y)>` uygular.  
- **Statik Yardımcılar:** `Guard`, `MoneyFormatter`, `GeoHelper` gibi genel amaçlı static sınıflar.

---

## 🔌 4. Plug-in (Eklenti) Mimarisi ve Dinamik Fiyatlama

### Amaç
Yeni mahalle kuralları ve fiyat politikaları, ana uygulama koduna dokunmadan sisteme eklenebilmelidir.

### Yapı
- **`IPricingRule`**: Tüm fiyat kuralları bu arayüzü uygular.  
- **`PricingEngine`**: Başlangıçta belirli bir klasörü tarar, içindeki tüm DLL’leri yükler.  
- DLL içindeki her sınıf `IPricingRule` uyguluyorsa otomatik olarak sisteme dahil edilir.

### Fiyat Hesaplama Akışı

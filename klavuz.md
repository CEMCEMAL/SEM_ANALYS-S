# Membrane Science Analyzer v2.0
## Kullanım Kılavuzu

> **SEM / TEM · Porozite · Fouling · Aktif Bölge Analizi**  
> Sürüm: 2.0 · Python 3.8+

## İçindekiler

1. [Genel Bakış](#1-genel-bakış)
2. [Kurulum](#2-kurulum)
3. [Arayüz Tanıtımı](#3-arayüz-tanıtımı)
4. [Görüntü Yükleme](#4-görüntü-yükleme)
5. [Ölçek Kalibrasyonu](#5-ölçek-kalibrasyonu)
6. [Analiz Modları](#6-analiz-modları)
   - 6.1 [Porozite Analizi](#61-porozite-analizi)
   - 6.2 [Fouling Katmanı Analizi](#62-fouling-katmanı-analizi)
   - 6.3 [Aktif Bölge / Katalizör Analizi](#63-aktif-bölge--katalizör-analizi)
   - 6.4 [Karşılaştırmalı Analiz](#64-karşılaştırmalı-analiz)
7. [Parametreler](#7-parametreler)
8. [Sonuçlar ve Grafikler](#8-sonuçlar-ve-grafikler)
9. [Dışa Aktarım](#9-dışa-aktarım)
10. [Klavye Kısayolları](#10-klavye-kısayolları)
11. [Desteklenen Dosya Formatları](#11-desteklenen-dosya-formatları)
12. [Metrik Açıklamaları](#12-metrik-açıklamaları)
13. [Sık Karşılaşılan Sorunlar](#13-sık-karşılaşılan-sorunlar)
14. [İpuçları ve En İyi Uygulamalar](#14-i̇puçları-ve-en-i̇yi-uygulamalar)

---

## 1. Genel Bakış

**Membrane Science Analyzer**, SEM (Taramalı Elektron Mikroskobu) ve TEM (Geçirimli Elektron Mikroskobu) görüntüleri üzerinde kantitatif analiz yapmak için geliştirilmiş açık kaynaklı bir masaüstü uygulamasıdır.

### Temel Özellikler

| Özellik | Açıklama |
|---|---|
| Porozite Analizi | Otsu, Adaptive ve Triangle eşikleme yöntemleriyle gözenek tespiti |
| Fouling Tespiti | Sobel + Canny kenar haritası ile kirlenme katmanı kalınlığı |
| Aktif Bölge Analizi | K-Means kümeleme + CLAHE normalizasyonu |
| Karşılaştırma | İki görüntü arasında ∆ porozite hesabı |
| Ölçek Kalibrasyonu | px/µm girişiyle µm cinsinden çap raporlama |
| Dışa Aktarım | CSV, JSON, PDF (grafik gömülü) |
| Çift Dil | Türkçe / İngilizce arayüz |

### Teknik Altyapı

```
Python ─── OpenCV ─── Thresholding, Sobel, Canny, K-Means
        ├── scikit-image ── Regionprops, CLAHE, Canny
        ├── SciPy ──────── Gaussian filtre, gradyan
        ├── Matplotlib ─── Grafikler, PDF figürü
        ├── ReportLab ──── PDF rapor oluşturma
        └── Tkinter ────── Arayüz
```

---

## 2. Kurulum

### 2.1 Gereksinimler

- Python **3.8** veya üzeri
- İşletim sistemi: Windows 10+, macOS 11+, Ubuntu 20.04+

### 2.2 Bağımlılıkları Yükle

```bash
pip install opencv-python pillow scikit-image numpy scipy matplotlib reportlab
```

### 2.3 İsteğe Bağlı: Sürükle & Bırak Desteği

```bash
pip install tkinterdnd2
```

> Bu paket kurulu değilse uygulama normal çalışır; yalnızca sürükle-bırak özelliği devre dışı kalır.

### 2.4 Uygulamayı Başlat

```bash
python membrane_analyzer_v2.py
```

---

## 3. Arayüz Tanıtımı

```
┌─────────────────────────────────────────────────────────────────────┐
│  TOPBAR  │ Uygulama adı · Versiyon · Dil butonu · Durum göstergesi  │
├──────────┼──────────────────────────────────────┬──────────────────┤
│          │                                      │                  │
│  KENAR   │         GÖRÜNTÜ CANVAS               │  METRİK          │
│  ÇUBUĞU  │     (Zoom · Pan · Drop desteği)      │  KARTLARI        │
│          │                                      │  (%  adet  px)   │
│  Görüntü │                                      │                  │
│  Yükleme ├──────────────────────────────────────┴──────────────────┤
│          │  [ Sonuçlar ] [ Grafikler ] [ İş Geçmişi ]              │
│  Ölçek   │                                                          │
│  Modu    │  Analiz çıktıları ve matplotlib grafikleri               │
│  Params  │                                                          │
│  İşlemler│                                                          │
└──────────┴──────────────────────────────────────────────────────────┘
```

### Panel Açıklamaları

**Kenar Çubuğu (sol)**
- **Görüntü Girişi** — Birincil ve karşılaştırma görüntülerini yükle
- **Ölçek Kalibrasyonu** — px/µm değeri gir
- **Analiz Modu** — 4 moddan birini seç
- **Parametreler** — Eşikleme yöntemi, gözenek boyutu, fouling yönü, küme sayısı
- **İşlemler** — Çalıştır, Dışa aktar (CSV/JSON/PDF), Sıfırla

**Görüntü Canvas (orta-üst)**
- Yüklenen görüntüyü gösterir
- Analiz sonrası işlenmiş görüntüyü (örn. gözenek maskesi) gösterir
- Zoom ve pan desteği vardır

**Metrik Kartları (sağ)**
- Porozite %, Gözenek sayısı, Ortalama çap, Fouling kalınlığı
- Her analiz sonrası otomatik güncellenir

**Sekmeler (alt)**
- **Sonuçlar** — Tüm sayısal çıktılar
- **Grafikler** — Matplotlib görselleştirmeleri
- **İş Geçmişi** — Zaman damgalı oturum logu

---

## 4. Görüntü Yükleme

Görüntü yüklemek için üç yöntem kullanılabilir:

### Yöntem 1: Aç Butonu
Kenar çubuğunda **"Birincil Görüntü"** altındaki **Aç** butonuna tıklayın, dosya seçiciden görüntünüzü seçin.

### Yöntem 2: Sürükle & Bırak
Dosyayı doğrudan görüntü canvas alanına sürükleyip bırakın.  
*(tkinterdnd2 paketi kurulu olmalıdır)*

### Yöntem 3: Son Dosyalar Menüsü
Üst menüden **Dosya → Son Dosyalar** yolunu izleyerek son 8 dosyadan birini açın.  
Son dosyalar `~/.membrane_analyzer_recent.json` dosyasında saklanır.

### Klavye Kısayolu
`Ctrl + O` ile dosya seçici açılır.

### Görüntü Metadata Çubuğu
Görüntü yüklendikten sonra canvas üstünde şu bilgiler gösterilir:

```
dosya_adi.tif  ·  2048×1536  ·  1ch  ·  8bit  ·  3.2 MB
```

### Desteklenen Formatlar

| Format | Uzantı | Not |
|--------|--------|-----|
| PNG | `.png` | Önerilen lossless format |
| JPEG | `.jpg`, `.jpeg` | Kayıplı sıkıştırma — dikkatli kullanın |
| TIFF | `.tif`, `.tiff` | 16-bit desteklenir, otomatik 8-bit'e dönüştürülür |
| BMP | `.bmp` | — |
| AVIF | `.avif` | — |
| WebP | `.webp` | — |

> ⚠️ **Not:** JPEG formatındaki görüntüler sıkıştırma kayıpları nedeniyle eşikleme hassasiyetini olumsuz etkileyebilir. Bilimsel analizlerde **TIFF** veya **PNG** kullanmanız önerilir.

---

## 5. Ölçek Kalibrasyonu

Ölçek kalibrasyonu, gözenek çaplarının piksel (px) yerine **mikrometre (µm)** cinsinden raporlanmasını sağlar.

### Kalibrasyonu Ayarla

1. Kenar çubuğunda **"Ölçek Kalibrasyonu"** bölümünü bulun
2. **px / µm** alanına SEM/TEM ölçek çubuğunuzdaki değeri girin
3. `0` bırakırsanız sonuçlar piksel cinsinden raporlanır

### px/µm Değerini Bulma

SEM görüntünüzdeki ölçek çubuğuna bakın:

```
Örnek: Ölçek çubuğu = 500 nm = 0.5 µm ve 100 px uzunluğunda
→ px/µm = 100 px / 0.5 µm = 200 px/µm
```

### Kalibrasyon Etkisi

Kalibrasyon girildiğinde Porozite Analizi sonuçlarına şu satır eklenir:

```
Ort. Eşdeğer Çap (µm):   0.847
```

---

## 6. Analiz Modları

### 6.1 Porozite Analizi

Membran görüntüsündeki gözenekleri tespit eder ve morfometrik istatistikler üretir.

**Çalışma Prensibi:**
1. Görüntü griye dönüştürülür
2. Seçilen eşikleme yöntemiyle ikili maske oluşturulur
3. Morfolojik açma işlemiyle gürültü temizlenir
4. Bağlı bileşenler etiketlenir, minimum boyutun altındakiler elenir
5. Her gözenek için alan, çap ve eksantriklik hesaplanır

**Çıktı Metrikleri:**

| Metrik | Açıklama |
|--------|----------|
| Porozite (%) | Gözenek alanı / toplam alan × 100 |
| Toplam Gözenek | Min. boyutu geçen gözenek sayısı |
| Ort. Eşdeğer Çap (px) | 2 × √(alan / π) formülüyle hesaplanan çap |
| Std. Sapma (px) | Çap dağılımının standart sapması |
| Min / Max Çap | En küçük ve en büyük gözenek çapı |
| D10 / D50 / D90 | Yüzdelik çap değerleri |
| Ort. Eksantriklik | 0 = mükemmel daire, 1 = çizgi |

**Görsel Çıktı:**  
Canvas'ta gözenekler **yeşil** renkte orijinal görüntü üzerine bindirilerek gösterilir.

---

### 6.2 Fouling Katmanı Analizi

Membran yüzeyindeki kirlenme (fouling) katmanının kalınlığını ve yoğunluğunu ölçer.

**Çalışma Prensibi:**
1. Seçilen yöne göre satır/sütun yoğunluk profili çıkarılır
2. Sobel + Canny kenar tespiti uygulanır
3. Yoğunluk gradyanı Gaussian filtre ile yumuşatılır
4. En yüksek gradyan noktası fouling/membran arayüzü olarak belirlenir
5. Arayüz iki bölgeye ayırır: fouling katmanı ve membran bölgesi

**Fouling Yönü Seçimi:**

| Seçenek | Açıklama |
|---------|----------|
| `top` | Fouling katmanı görüntünün üst kısmında |
| `bottom` | Fouling katmanı görüntünün alt kısmında |
| `left` | Fouling katmanı görüntünün sol kısmında |
| `right` | Fouling katmanı görüntünün sağ kısmında |

**Çıktı Metrikleri:**

| Metrik | Açıklama |
|--------|----------|
| Arayüz (px satır) | Fouling/membran sınırının piksel konumu |
| Fouling Kalınlığı (px) | Fouling katmanının piksel kalınlığı |
| Fouling Kalınlığı (%) | Toplam görüntü yüksekliğine oranı |
| Fouling Ort. Yoğunluk | Fouling bölgesinin gri değer ortalaması |
| Membran Ort. Yoğunluk | Membran bölgesinin gri değer ortalaması |
| Kontrast (σ) | Görüntünün standart sapması |
| **SNR (dB)** | Sinyal-Gürültü Oranı — fouling/membran kontrast kalitesi |

> **SNR yorumu:**  
> `> 20 dB` → Güçlü kontrast, güvenilir arayüz tespiti  
> `10–20 dB` → Orta kontrast, sonuçları manuel doğrulayın  
> `< 10 dB` → Düşük kontrast, analiz güvenilirliği düşük

---

### 6.3 Aktif Bölge / Katalizör Analizi

Membran yüzeyindeki yüksek yoğunluklu (parlak) ve düşük yoğunluklu (koyu) aktif bölgeleri tespit eder.

**Çalışma Prensibi:**
1. CLAHE (Contrast Limited Adaptive Histogram Equalization) ile kontrast artırılır
2. 90. yüzdelik üstündeki pikseller parlak, 10. yüzdelik altındakiler koyu maske olarak tanımlanır
3. Morfolojik açma ile gürültü temizlenir
4. K-Means kümeleme ile görüntü `k` parçaya bölünür
5. Her küme için segmentasyon haritası oluşturulur

**Çıktı Metrikleri:**

| Metrik | Açıklama |
|--------|----------|
| K-Means Küme Sayısı | Kullanılan küme sayısı (2–12) |
| Parlak Bölge Sayısı | Yüksek yoğunluklu aktif bölge adedi |
| Koyu Bölge Sayısı | Düşük yoğunluklu aktif bölge adedi |
| Toplam Aktif Bölge | Parlak + Koyu |
| Parlak Kaplama (%) | Parlak bölgelerin görüntü alanına oranı |
| Koyu Kaplama (%) | Koyu bölgelerin görüntü alanına oranı |
| Toplam Kaplama (%) | Tüm aktif bölgelerin toplam kapladığı alan |

---

### 6.4 Karşılaştırmalı Analiz

İki farklı membran görüntüsü (örn. temiz vs. fouled) arasında porozite karşılaştırması yapar.

**Kullanım:**
1. **Birincil Görüntü** alanına ilk görüntüyü yükleyin
2. **Karşılaştırma Görüntüsü** alanına ikinci görüntüyü yükleyin
3. Modu **"Karşılaştırmalı Analiz"** olarak seçin
4. Analizi çalıştırın

**Çıktı Metrikleri:**

| Metrik | Açıklama |
|--------|----------|
| G1 Porozite (%) | Birinci görüntünün porozitesi |
| G2 Porozite (%) | İkinci görüntünün porozitesi |
| Delta Porozite | G2 − G1 farkı (`+` değer: G2 daha gözenekli) |
| G1 / G2 Gözenek Sayısı | Her görüntünün gözenek adedi |
| G1 / G2 Ort. Çap | Her görüntünün ortalama gözenek çapı |

> **∆ Porozite yorumu:**  
> `> +5%` → Anlamlı farklılık (sarı uyarı gösterilir)  
> `0 ± 5%` → Benzer porozite  
> `< -5%` → Birinci görüntü daha gözenekli

---

## 7. Parametreler

### Eşikleme Yöntemi

| Yöntem | Açıklama | Ne Zaman Kullanılır |
|--------|----------|---------------------|
| **Otsu** | Global optimal eşik; iki tepe noktalı histogramlar için ideal | Homojen arka plan, yüksek kontrast |
| **Adaptive** | Lokal pencere tabanlı eşikleme | Düzensiz aydınlatma, yerel kontrast farklılıkları |
| **Triangle** | Tek tepe noktalı histogramlar için Zack yöntemi | Seyrek gözenek dağılımı, düşük porozite |

### Min. Gözenek Boyutu (px)

Belirtilen pikselden küçük bölgeler gözenek sayılmaz. Gürültüyü ve artefaktları filtrelemek için kullanılır.

- **Küçük değer (1–10 px):** Tüm mikroyapı tespit edilir, gürültüye duyarlı
- **Orta değer (10–50 px):** Dengeli — çoğu SEM analizi için önerilen
- **Büyük değer (50–200 px):** Yalnızca büyük gözenekler sayılır

### Fouling Yönü

Fouling katmanının görüntüdeki konumunu belirtir. Yanlış seçilirse arayüz tespiti hatalı olur. Görüntünüzü inceleyerek fouling'in hangi kenardan başladığını belirleyin.

### K-Means Küme Sayısı (2–12)

Aktif Bölge analizinde görüntünün kaç yoğunluk grubuna bölüneceğini belirler.

- **2–3:** Basit iki fazlı yapılar
- **4–6:** Orta karmaşıklıktaki membranlar (varsayılan: 5)
- **7–12:** Çok fazlı veya heterojen malzemeler

> Küme sayısı artırıldıkça hesaplama süresi uzar.

---

## 8. Sonuçlar ve Grafikler

### Sonuçlar Sekmesi

Analiz tamamlandıktan sonra **Sonuçlar** sekmesine otomatik olarak geçilir. Renk kodlaması:

| Renk | Anlam |
|------|-------|
| 🟢 Yeşil (teal) | Normal değer |
| 🟡 Sarı (amber) | Dikkat gerektiren değer |
| 🔵 Mavi | Bilgi / parametre |
| 🔴 Kırmızı | Hata |

### Grafikler Sekmesi

Her analiz modu farklı grafikler üretir:

**Porozite:**
- Gözenek maskesi görüntüsü
- Boyut dağılımı histogramı (D10/D50/D90 çizgileriyle)
- Eksantriklik dağılımı histogramı

**Fouling:**
- Kenar haritası + arayüz çizgisi
- Yoğunluk profili + fouling bölgesi dolgu alanı

**Aktif Bölge:**
- Normalize görüntü
- Parlak bölge maskesi
- Koyu bölge maskesi
- K-Means segmentasyon haritası

**Karşılaştırma:**
- Porozite karşılaştırma çubuk grafiği (değer etiketleriyle)
- Gözenek sayısı karşılaştırma çubuk grafiği

### Canvas Zoom & Pan

| İşlem | Eylem |
|-------|-------|
| Yakınlaştır | Fare tekerleği yukarı |
| Uzaklaştır | Fare tekerleği aşağı |
| Kaydır | Sol tık basılı tut + sürükle |
| Görünümü sıfırla | Çift tıkla |

---

## 9. Dışa Aktarım

### CSV Dışa Aktarım

Tüm sayısal sonuçları tablo formatında kaydeder.

**Sütunlar:**
```
Parametre | Değer | Modül | Görüntü Dosyası | Dışa Aktarım Tarihi | Versiyon
```

**Kısayol:** `Ctrl + S`

---

### JSON Dışa Aktarım

Makine tarafından okunabilir, yapılandırılmış format. Programatik analiz için idealdir.

**Örnek çıktı:**
```json
{
  "app_version": "2.0",
  "exported_at": "2025-03-15T14:32:11",
  "image": {
    "path": "/data/sem_001.tif",
    "width": 2048,
    "height": 1536,
    "file_kb": 3276.8,
    "scale_um_per_px": 200.0
  },
  "modules": {
    "porosity": {
      "method": "otsu",
      "porosity_pct": 12.847,
      "pore_count": 341,
      "mean_diam_px": 18.23,
      "mean_diam_um": 0.0911,
      "d10_px": 11.4,
      "d50_px": 17.8,
      "d90_px": 26.3
    }
  }
}
```

---

### PDF Dışa Aktarım

Grafiklerin görüntüsünü ve tüm sayısal sonuçları içeren profesyonel A4 rapor oluşturur.

**PDF içeriği:**
- Rapor başlığı, görüntü adı ve tarih
- Gömülü matplotlib figürü (120 DPI)
- Her modül için parametreli sonuç tablosu

**Kısayol:** `Ctrl + Shift + S`

> ⚠️ PDF dışa aktarımı için `reportlab` paketi kurulu olmalıdır:
> ```bash
> pip install reportlab
> ```

---

## 10. Klavye Kısayolları

| Kısayol | İşlev |
|---------|-------|
| `Ctrl + O` | Görüntü aç |
| `F5` veya `Ctrl + R` | Analizi çalıştır |
| `Ctrl + S` | CSV dışa aktar |
| `Ctrl + Shift + S` | PDF dışa aktar |
| `Ctrl + Z` | Uygulamayı sıfırla |

---

## 11. Desteklenen Dosya Formatları

| Format | Okuma | Öneri |
|--------|-------|-------|
| TIFF (8/16-bit) | ✅ | ⭐ Bilimsel analiz için en iyi seçim |
| PNG | ✅ | ⭐ Lossless, gürültüsüz |
| BMP | ✅ | Büyük dosya boyutu |
| JPEG | ✅ | ⚠️ Kayıplı — dikkatli kullanın |
| AVIF | ✅ | — |
| WebP | ✅ | — |

---

## 12. Metrik Açıklamaları

### Eşdeğer Çap (Equivalent Diameter)

Gözenek alanıyla aynı alana sahip bir dairenin çapıdır:

```
d_eq = 2 × √(Alan / π)
```

### Eksantriklik (Eccentricity)

Gözenek şeklinin dairesellikten sapmasını ölçer:

```
0.0 → Mükemmel daire
0.5 → Orta elips
1.0 → Çizgi (tam elips)
```

### D10 / D50 / D90 Yüzdelikleri

- **D10:** Gözeneklerin %10'u bu değerin altında çapa sahiptir
- **D50:** Medyan çap (gözeneklerin yarısı bu değerin altında, yarısı üstünde)
- **D90:** Gözeneklerin %90'ı bu değerin altında çapa sahiptir

Geniş bir D10–D90 aralığı, heterojen bir gözenek boyut dağılımına işaret eder.

### SNR (Signal-to-Noise Ratio)

Fouling analizinde membran ile fouling bölgeleri arasındaki kontrast kalitesini dB cinsinden ölçer:

```
SNR = 20 × log10(|Membran Yoğunluğu − Fouling Yoğunluğu| / Gürültü)
```

Yüksek SNR değeri, arayüz tespitinin daha güvenilir olduğunu gösterir.

### Porozite (%)

```
Porozite = (Gözenek Piksel Sayısı / Toplam Piksel Sayısı) × 100
```

---

## 13. Sık Karşılaşılan Sorunlar

### Görüntü Açılmıyor

**Belirti:** Hata mesajı veya boş canvas  
**Çözüm:**
- Dosya yolunda Türkçe karakter / boşluk var mı kontrol edin
- Dosyanın bozuk olmadığını doğrulayın
- 16-bit TIFF otomatik dönüştürülür; 32-bit float formatlar desteklenmez

---

### Çok Az veya Çok Fazla Gözenek Tespit Ediliyor

**Az gözenek:**
- Min. Gözenek Boyutu değerini düşürün
- Eşikleme yöntemini `adaptive` olarak değiştirin
- Görüntü kontrastını SEM yazılımında artırın

**Fazla gözenek (gürültü):**
- Min. Gözenek Boyutu değerini artırın (örn. 20–50 px)
- `otsu` veya `triangle` yöntemini deneyin

---

### Fouling Arayüzü Yanlış Tespit Ediliyor

**Belirti:** Arayüz çizgisi görüntüde yanlış konumda  
**Çözüm:**
- Fouling Yönü parametresinin doğru seçildiğini kontrol edin
- Görüntüde fouling katmanının keskin bir sınır oluşturduğundan emin olun
- Çok düşük SNR değerleri (< 10 dB) güvenilmez tespite işaret eder

---

### PDF Dışa Aktarım Çalışmıyor

**Belirti:** "Eksik Paket" hatası  
**Çözüm:**
```bash
pip install reportlab
```

---

### Sürükle & Bırak Çalışmıyor

**Çözüm:**
```bash
pip install tkinterdnd2
```
Paketi yükledikten sonra uygulamayı yeniden başlatın.

---

### Analiz Çok Uzun Sürüyor

K-Means (Aktif Bölge) analizi büyük görüntülerde yavaş çalışabilir.  
**Çözüm:**
- Küme sayısını azaltın (2–4)
- Görüntüyü analiz öncesinde yeniden boyutlandırın

---

## 14. İpuçları ve En İyi Uygulamalar

### Görüntü Kalitesi

- Analize başlamadan önce SEM görüntünüzün **odakta ve kontrast açısından optimize** edildiğinden emin olun
- **16-bit TIFF** formatı en yüksek dinamik aralığı sağlar
- JPEG kullanmak zorundaysanız, en düşük sıkıştırma kalitesini seçin

### Eşikleme Yöntemi Seçimi

```
Homojen arka plan + yüksek kontrast  →  Otsu
Düzensiz aydınlatma / vinyeting      →  Adaptive
Seyrek gözenek / düşük porozite      →  Triangle
```

### Tekrarlanabilirlik için İş Akışı

1. Her analiz için aynı görüntü formatını kullanın (tercihen TIFF)
2. Ölçek kalibrasyonunu ilk seferinde belirleyin ve not edin
3. Parametreleri JSON dosyasında saklayın (JSON export metadata içerir)
4. Aynı parametre setiyle birden fazla görüntüyü analiz edin

### Yeniden Çalıştırma Uyarısı

Parametreleri değiştirdiğinizde kenar çubuğunda **sarı uyarı mesajı** belirir:

```
⚠  Parametreler değişti — yeniden çalıştırın
```

Bu mesajı gördüğünüzde analizi yeniden çalıştırmanız gerekir; aksi hâlde gösterilen sonuçlar eski parametrelere aittir.

### Büyük Veri Setleri için

- Karşılaştırmalı analizde görüntülerin aynı büyüklükte ve çözünürlükte olmasına dikkat edin
- İş Geçmişi sekmesi oturum logu tutar; CSV/JSON çıktılarını arşivleyerek sonuçları uzun vadeli saklayın

---

## Lisans ve Katkı

Bu uygulama açık kaynak olarak geliştirilmiştir. Hata bildirimi, özellik önerisi veya katkı için proje deposuyla iletişime geçin.

---

*Membrane Science Analyzer v2.0 · Kullanım Kılavuzu*  
*Son güncelleme: 2025*

---
layout: post
title: "Otokodlayıcı Tabanlı Özellik Çıkarımı ve Sınıflandırma"
date: 2025-06-12
categories: [makine-öğrenmesi, otokodlayıcı, özellik-çıkarımı]
tags: [autoencoder, feature-extraction, classification]
---

## İçindekiler
- [Giriş](#giriş)  
- [Yöntemler](#yöntemler)  
  - [Veri Seti ve Ön İşleme](#veri-seti-ve-ön-işleme)  
  - [Model Mimarileri](#model-mimarileri)  
  - [Eğitim Prosedürü](#eğitim-prosedürü)  
- [Bulgular](#bulgular)  
- [Tartışma](#tartışma)  
- [Sonuç](#sonuç)  
- [Kaynaklar](#kaynaklar)  

---

## Giriş
Yüksek boyutlu veri, makine öğrenmesi modellerinin hem eğitim süresini uzatır hem de aşırı öğrenme riskini artırır. Otokodlayıcılar (autoencoder) ise bu sorunu, verinin temel yapısını koruyarak boyut indirgeme yoluyla hafifletmeyi amaçlar. Bu çalışmada, UCI Forest CoverType veri seti üzerinde iki farklı iş akışını karşılaştırıyoruz:

1. **Ham Veri + Random Forest**  
2. **Autoencoder ile Özellik Çıkarımı + Random Forest**

Amacımız, düşük boyutlu temsilin sınıflandırma başarısını nasıl etkilediğini anlamak ve veri sıkıştırma yönteminin pratikteki fayda-maliyet dengesini ortaya koymak.

---

## Yöntemler

### Veri Seti ve Ön İşleme
Çalışmada UCI Forest CoverType veri setinden ilk 10.000 örnek kullanıldı.  

- **Özellikler:** Sayısal orman verileri (topoğrafik ve higrometrik ölçümler).  
- **Etiket:** `Cover_Type` (1–7 arası sınıflar).  
- **Ön İşleme Adımları** (`data_preprocessing.py`):
  1. `Cover_Type` kolonunu bağımsız değişkenlerden ayırdık.  
  2. Özellikleri **StandardScaler** ile normalize ettik.  
  3. **Stratified** %80 eğitim / %20 test olacak şekilde böldük.  
  4. Çıktılar: `X_train.csv`, `X_test.csv`, `y_train.csv`, `y_test.csv`.  

Bu adımlar, model eğitimi sırasında veri dağılımının korunmasını ve skaladan kaynaklanan sapmaların minimize edilmesini sağladı.

### Model Mimarileri
İki farklı iş akışı tanımlandı:

- **A. Ham Veri + Random Forest**  
  - **Model:** `RandomForestClassifier(n_estimators=200)`  
  - **Neden?** Güçlü, ayarlaması nispeten kolay ve yüksek boyutlu verilerle etkili çalışan bir yöntem.  

- **B. Autoencoder Tabanlı Özellik Çıkarımı + Random Forest**  
  1. **Autoencoder Tasarımı** (`autoencoder.py`):
     - Tek katmanlı encoder–decoder mimarisi  
     - Gizli boyut: 32  
     - Aktivasyon: ReLU  
  2. **Kod Çıkarımı:** Eğitilen encoder ile `X_train_encoded.csv` ve `X_test_encoded.csv` üretildi.  
  3. **Sınıflandırma:** Oluşan 32 boyutlu temsiller, yine **RandomForestClassifier(n_estimators=200)** ile sınıflandırıldı.

Bu iki yaklaşım, aynı sınıflandırıcı altyapısıyla karşılaştırılarak autoencoder etkisi izole edildi.

### Eğitim Prosedürü
Aşağıdaki komutlar sırasıyla çalıştırılarak tüm adımlar tamamlandı:

```bash
# 1) Veri ön işleme
python data_preprocessing.py

# 2) Autoencoder eğitimi ve kod çıkarımı
python autoencoder.py

# 3) Random Forest sınıflandırıcı eğitimi
python classifier.py
```
Her bir aşama tamamlandığında ilgili çıktı dosyaları ve eğitim logları results.txt içine kaydedildi.

## 4. Bulgular  
Veri seti üzerinde yaptığımız testler, iki yaklaşım arasındaki farkı net biçimde ortaya koydu:

| Model                                     | Test Accuracy | Precision (macro) | Recall (macro) | F1-Score (macro) |
|-------------------------------------------|--------------:|-----------------:|--------------:|-----------------:|
| **Ham Veri + Random Forest**              |        0.8490 |             0.84 |          0.84 |             0.84 |
| **Autoencoder → RF**                      |        0.8205 |             0.81 |          0.82 |             0.81 |

> “Ham veriye dayalı model, zengin özellik kümesi sayesinde %84.9’luk en yüksek doğruluk değerine ulaştı. Autoencoder’dan çıkan 32 boyutlu temsiller ise %82.1 gibi makul bir başarı sergiledi.”

Ayrıca eğitim süreleri şu şekilde ölçüldü:
- **Veri Ön İşleme:** ~1 dakika  
- **Autoencoder Eğitimi + Kod Çıkarımı:** ~12 dakika  
- **Random Forest Eğitimi:** ~5 dakika  

Autoencoder adımı, toplam akış süresini iki katına yakın uzattı; buna karşın özellik setini 54’dan 32’ye indirgedi.

---

## 5. Tartışma  
1. **Doğruluk Farkı ve Bilgi Kaybı**  
   - Boyut indirgeme sırasında bazı ayırt edici sinyaller sıkıştırılarak kayboldu. Bu, yaklaşık %2.8’lik düşüşe yol açtı.  
2. **Genellenebilirlik Avantajı**  
   - Daha düşük boyutlu temsiller aşırı öğrenmeyi azaltabilir; farklı veri dağılımlarında daha stabil performans sunabilir.  
3. **Hesaplama-Maliyet Dengesi**  
   - Autoencoder eğitimi ek zaman ve kaynak gerektiriyor. Kritik uygulamalarda ham veri + RF daha pratik olabilir.  
4. **İleri Yönlendirmeler**  
   - **Stacked Autoencoder** veya **Variational Autoencoder** ile derin temsiller elde etmeyi deneyin.  
   - Gürültülü ortamlarda **Denoising Autoencoder** kullanarak performansı iyileştirebilirsiniz.  
   - Boyut indirgeme sonrası **XGBoost** veya **LightGBM** gibi alternatif sınıflandırıcıları test etmek faydalı olabilir.

---

## 6. Sonuç  
Bu  çalışmada:
- **Ham Veri + Random Forest**, en yüksek sınıflandırma doğruluğunu (%84.9) sunarken  
- **Autoencoder Tabanlı Yaklaşım**, %82.1 doğruluk karşılığında 54 → 32 boyut indirgeme sağladı  

Sonuç olarak, “performans” mı yoksa “öznitelik verimliliği” mi önceliğiniz olduğuna göre uygun iş akışını seçebilirsiniz. Eğer model genellenebilirliği ve veri sıkıştırma kritikse autoencoder, eğitim süresinin kısa olması öncelikliyse ham veri senaryosu tercih edilmeli.

---

## 7. Kaynaklar  
1. Breneman, L. & Hall, L. O. (1998). *UCI Forest CoverType Data Set*.  
2. Pedregosa, F. et al. (2011). *Scikit-learn: Machine Learning in Python*.  
3. Chollet, F. (2015). *Keras Documentation*.  
4. Goodfellow, I., Bengio, Y. & Courville, A. (2016). *Deep Learning*.  

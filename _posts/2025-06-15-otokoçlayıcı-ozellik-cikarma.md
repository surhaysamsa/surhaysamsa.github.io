---
layout: post
title: "Otokodlayıcı Tabanlı Özellik Çıkarımı ve Sınıflandırma: IMRAD Yaklaşımı"
date: 2025-06-15
categories: [makine-öğrenmesi, otokodlayıcı, özellik-çıkarımı]
tags: [autoencoder, feature-extraction, classification, IMRAD]
---

## İçindekiler
- [Giriş](#giriş)  
- [Yöntemler](#yöntemler)  
- [Bulgular](#bulgular)  
- [Tartışma](#tartışma)  
- [Sonuç](#sonuç)  
- [Kaynaklar](#kaynaklar)  

---

## Giriş
Makine öğrenmesi uygulamalarında yüksek boyutlu veriler, gürültü ve fazla parametre nedeniyle sınıflandırma modellerinin genellenebilirliğini olumsuz etkileyebilir. Otokodlayıcılar (autoencoder) bu noktada boyut indirgeme ve anlamsal temsiller elde etmek için sıkça tercih edilir. Bu çalışmada:

1. **Hipotez:** Otokodlayıcı ile çıkarılan özellikler, ham veri temelli sınıflandırma performansından anlamlı ölçüde farklılaşır.  
2. **Amaç:** UCI Forest CoverType veri seti üzerinde “ham veri + Random Forest” ile “autoencoder tabanlı özellik çıkarımı + Random Forest” yaklaşımlarını karşılaştırmak.

---

## Yöntemler

### 1. Veri Seti ve Ön İşleme
- **Veri:** UCI Forest CoverType (ilk 10.000 örnek).  
- **Ön işleme adımları** (bakınız `data_preprocessing.py`):  
  1. `Cover_Type` etiketini ayırdık.  
  2. Özellikleri StandardScaler ile ölçekledik.  
  3. Stratified %80 eğitim / %20 test olarak böldük.  
  4. Çıktılar: `X_train.csv`, `X_test.csv`, `y_train.csv`, `y_test.csv`.

### 2. Model Mimarileri
- **A. Ham Veri + Random Forest**  
  - 200 ağaçlı RandomForestClassifier (scikit-learn).  
  - Kod: `classifier.py` içindeki `train_random_forest(X_train, y_train)`.

- **B. Autoencoder Tabanlı Özellik Çıkarımı + Random Forest**  
  1. `autoencoder.py` ile tek katmanlı, gizli boyut = 32, aktivasyon = ReLU bir autoencoder eğittik.  
  2. Encoder’dan elde edilen kodları (`X_train_encoded.csv`, `X_test_encoded.csv`) ölçekledik.  
  3. Aynı Random Forest sınıflandırıcıyı bu yeni özellikler üzerinde eğittik.

### 3. Eğitim Prosedürü
```bash
# Ön işleme
python data_preprocessing.py  

# Autoencoder eğitimi ve kod çıkarımı
python autoencoder.py  

# Sınıflandırma
python classifier.py  
```
## Bulgular

| Model                                         | Test Accuracy |
|-----------------------------------------------|--------------:|
| Ham Veri + Random Forest                      |         0.849 |
| Autoencoder Özellikleri + Random Forest       |         0.821 |

- **Karşılaştırma:**  
  - Ham veri temelli model %84.9 doğruluk sağladı.  
  - Autoencoder tabanlı özellik çıkarımı ile model %82.1 doğruluk gösterdi.  
- **Eğitim Süreleri:**  
  - Veri ön işleme: ~1 dk  
  - Autoencoder eğitimi ve kod çıkarımı: ~12 dk  
  - Random Forest eğitimi: ~5 dk  

---

## Tartışma

1. **Performans Farkı:**  
   - Boyut indirgeme sırasında bazı ayırt edici bilgilerin sıkıştırılması, sınıflandırma doğruluğunu %2.8 oranında düşürdü.  
2. **Genellenebilirlik ve Aşırı Öğrenme:**  
   - Daha düşük boyutlu temsiller, overfitting riskini azaltarak farklı veri setlerinde daha kararlı sonuçlar verebilir.  
3. **Hesaplama Maliyeti:**  
   - Autoencoder eğitimi ek zaman ve kaynak gerektirirken, ham veri ile doğrudan Random Forest daha hızlı işlem sunuyor.  
4. **Genişletilebilirlik:**  
   - Katman sayısı artırılarak (stacked autoencoder) veya VAE/Denoising AE gibi alternatif mimarilerle performans artırılabilir.  

---

## Sonuç

Bu çalışmada, UCI Forest CoverType veri seti üzerinde “ham veri + Random Forest” ve “autoencoder tabanlı özellik çıkarımı + Random Forest” yaklaşımları karşılaştırıldı. Sonuç olarak:

- Güçlü sınıflandırıcılar (ör. Random Forest), ham veri üzerinde daha yüksek doğruluk sağlıyor.  
- Autoencoder’lar, düşük boyutlu ve genellenebilir özellikler elde etmek için etkili bir yöntem olarak öne çıkıyor; ancak ek hesaplama yükü getiriyor.  

Projenin gereksinimleri doğrultusunda, ham veri performansını mı yoksa boyut indirgeme avantajını mı önceliklendireceğinize karar verilebilir.

---

## Kaynaklar

1. Breneman, L. & Hall, L. O. (1998). *UCI Forest CoverType Data Set*.  
2. Pedregosa, F. et al. (2011). *Scikit-learn: Machine Learning in Python*.  
3. Chollet, F. (2015). *Keras Documentation*.  
4. Goodfellow, I., Bengio, Y. & Courville, A. (2016). *Deep Learning*.

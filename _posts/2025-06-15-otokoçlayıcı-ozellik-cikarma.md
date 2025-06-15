---
layout: post
title: "Otokodlayıcı Tabanlı Özellik Çıkarımı ve Ham Veri Sınıflandırma Karşılaştırması"
date: 2025-06-15
categories: [makine-öğrenmesi, otokodlayıcı, özellik-çıkarımı]
tags: [autoencoder, feature-extraction, classification]
---

## İçindekiler
- [Giriş](#giriş)
- [Yöntem](#yöntem)
  - [Veri Seti ve Ön İşleme](#veri-seti-ve-ön-işleme)
  - [Model Mimarileri](#model-mimarileri)
- [Deneysel Sonuçlar](#deneysel-sonuçlar)
- [Tartışma](#tartışma)
- [Sonuç](#sonuç)
- [Kaynaklar](#kaynaklar)

## Giriş
Bu yazıda, UCI Forest CoverType veri seti üzerinde “ham veriyle sınıflandırma” ile “otokodlayıcı tabanlı özellik çıkarımı + sınıflandırma” yaklaşımlarını karşılaştırıyoruz. Amacımız, autoencoder’ların sınıflandırma performansını nasıl etkilediğini pratikte görmek.

## Yöntem

### Veri Seti ve Ön İşleme
- **Veri seti:** `covtype.csv` (ilk 10.000 örnek)  
- **Ön işleme:**  
  1. Sınıf etiketini (`Cover_Type`) ayırdık.  
  2. %80 eğitim, %20 test, stratified split ile böldük.  
  3. `data_preprocessing.py` ile `X_train.csv`, `X_test.csv`, `y_train.csv`, `y_test.csv` dosyaları oluşturduk.

### Model Mimarileri
- **Ham Veri Modeli:**  
  - Random Forest (200 ağaç, scikit-learn)  
  - Özellik ölçekleme: StandardScaler  
- **Otokodlayıcı + Sınıflandırma:**  
  1. `autoencoder.py` ile tek katmanlı, 32 boyutlu kodlama (ReLU)  
  2. Çıkan kodu (`X_train_encoded.csv`, `X_test_encoded.csv`) StandardScaler ile ölçekledik.  
  3. Aynı Random Forest ile sınıflandırma yaptık.

## Deneysel Sonuçlar

| Model                                        | Accuracy |
|----------------------------------------------|---------:|
| Ham Veri ile Sınıflandırma                   |    0.8490 |
| Otokodlayıcı Tabanlı Özellik Çıkarımı ile    |    0.8205 |

> **Komutlar**  
> ```bash
> python data_preprocessing.py
> python autoencoder.py
> python classifier.py
> ```

## Tartışma
- **Ham veri** Random Forest’a zengin bilgi sağladığı için %84.9 doğruluk sundu.  
- **Otokodlayıcı**, boyut indirgeme yaparken bazı ayırt edici bilgileri sıkıştırıyor ve performans %82.1’e geriliyor.  
- Yine de düşük boyutlu, daha genellenebilir özellikler elde etmek istendiğinde autoencoder’lar avantajlı olabilir.

## Sonuç
Autoencoder tabanlı yaklaşım, yüksek boyutlu ve gürültülü veri durumlarında faydalı olabilir. Ancak güçlü sınıflandırıcılar (ör. Random Forest) ham veride çalışan versiyonlarda daha iyi sonuç verebiliyor. Projenizi gereksinime göre yapılandırın.

## Kaynaklar
1. UCI ML Repo – Forest CoverType Data Set  
2. Scikit-learn Documentation  
3. Keras Documentation  

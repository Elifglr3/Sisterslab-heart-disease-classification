
# 1. Problem Tanımı

## Projenin Amacı
Bu projenin temel amacı, bireylerin sahip olduğu klinik ve demografik özelliklere göre kalp hastalığı riski taşıyıp taşımadığını tahmin edebilen bir makine öğrenmesi modeli geliştirmektir.

## Hedef Değişken (target)
- `0`: Kalp hastalığı yok  
- `1`: Kalp hastalığı var  

Modelin temel görevi, bu ikili sınıflandırmayı mümkün olduğunca doğru yapmaktır.

## Problem Türü
**İkili Sınıflandırma (Binary Classification)**  
> Kullanılacak algoritmalar: Logistic Regression, Random Forest, XGBoost, vb.

## Veri Seti Kaynağı
- Kaynak: Kaggle – UCI Heart Disease Dataset  
- Klinikler: Cleveland, Hungary, Switzerland, VA Long Beach  

## Medikal Amaç
Kalp hastalıkları, dünya çapında ölüm nedenlerinin başında gelmektedir. Klinik verilerle geliştirilen bu model, erken uyarı sistemleri için yorumlanabilir ve güvenilir bir tahmin mekanizması sunmayı hedeflemektedir.

# 2. Veri Seti Özellikleri

## Genel Bilgiler
- 920 satır × 16 sütun

## Değişken Türleri

### Sayısal Değişkenler:
- `age`: Yaş (yıl)
- `trestbps`: Dinlenme sırasındaki kan basıncı (mm Hg)
- `chol`: Serum kolestrol (mg/dl)
- `thalach`: Maksimum kalp atış hızı
- `oldpeak`: ST segmenti depresyonu

### İkili Kategorik Değişkenler:
- `sex`: 0 = Kadın, 1 = Erkek
- `fbs`: Açlık kan şekeri > 120 mg/dl (1 = Evet, 0 = Hayır)
- `exang`: Egzersize bağlı anjina (1 = Evet, 0 = Hayır)

### Çok Kategorili Değişkenler:
- `cp`: Göğüs ağrısı tipi (0–3)
- `restecg`: Dinlenme EKG sonuçları
- `slope`: ST segment eğimi
- `ca`: Büyük damar sayısı (0–3)
- `thal`: Talasemi tipi

# 3. Eksik Veri Değerlendirmesi

## Eksik Veri Türleri
- **MCAR**: Tamamen rastgele
- **MAR**: Başka gözlemlenebilir değişkenlerle ilişkili
- **MNAR**: Değişkenin kendisiyle ilişkili

## Eksikliği Yüksek Değişkenler (Çıkarıldı)
- `ca`: %66 eksik
- `thal`: %53 eksik

## Doldurma Stratejileri

| Tür        | Değişkenler                          | Yöntem            |
|------------|--------------------------------------|-------------------|
| Sayısal    | trestbps, chol, thalach, oldpeak     | KNNImputer (k=5)  |
| Kategorik  | slope, fbs, restecg, exang           | Mod (en sık)      |

# 4. Aykırı Değer Analizi

| Değişken   | Aykırı Sayısı | Yöntem                |
|------------|---------------|------------------------|
| age        | 0             | -                      |
| trestbps   | 28            | Medyan ile değiştirildi |
| chol       | 185           | Winsorization          |
| thalach    | 2             | Korundu                |
| oldpeak    | 16            | Medyan ile değiştirildi |

# 5. Korelasyon Analizi (Sayısal Değişkenler)

| Değişken   | Korelasyon | Açıklama                                      |
|------------|------------|-----------------------------------------------|
| thalach    | -0.38      | Daha düşük kalp atış hızı → daha yüksek risk |
| oldpeak    | +0.37      | ST depresyonu arttıkça risk artıyor           |
| age        | +0.28      | Yaş arttıkça risk artıyor                     |
| chol       | -0.22      | Zayıf ters ilişki                             |

# 6. Encoding İşlemleri

## Label Encoding

| Değişken | Önce                  | Sonra     |
|----------|-----------------------|-----------|
| sex      | 'Male', 'Female'      | 1, 0      |
| fbs      | True, False           | 1, 0      |
| exang    | True, False           | 1, 0      |

## One-Hot Encoding

- `dataset`: 4 sütun (Hungary, Switzerland, vb.)
- `cp`, `restecg`, `slope`: her biri için 2–3 ek sütun oluşturuldu

# 7. Sayısal Değişkenlerin Ölçeklendirilmesi

- **Yöntem**: `StandardScaler`
- Tüm sayısal değişkenler normalize edildi (ortalama ≈ 0, std ≈ 1)

# 8. Model Performansı (Tuning Öncesi)


| Model                   | AUC  | FN | FP | Mean Accuracy (CV) | Test Accuracy | Train Accuracy | Overfitting Farkı | Notlar                             |
| ----------------------- | ---- | -- | -- | ------------------ | ------------- | -------------- | ----------------- | ---------------------------------- |
| **Logistic Regression** | 0.91 | 13 | 19 | 0.77               | 0.83          | 0.83           | 0.00              | Dengeli ve overfitting yok         |
| **Random Forest**       | 0.91 | 13 | 16 | 0.73               | 0.84          | 1.00           | 0.16              | Aşırı öğrenme eğilimi              |
| **KNN**                 | 0.88 | 9  | 21 | 0.72               | 0.84          | 0.85           | 0.01              | En düşük FN, stabil                |
| **SVM**                 | 0.90 | 10 | 20 | 0.75               | 0.84          | 0.86           | 0.02              | Dengeli, genel başarı yüksek       |
| **XGBoost**             | 0.90 | 17 | 16 | 0.68               | 0.82          | 1.00           | 0.18              | Aşırı öğrenme net                  |
| **Gradient Boosting**   | 0.91 | 12 | 16 | 0.65               | 0.85          | 0.92           | 0.07              | ROC güçlü, overfitting orta        |
| **CatBoost**            | 0.91 | 12 | 18 | 0.71               | 0.84          | 0.96           | 0.12              | Güçlü ayırıcı ama overfit eğilimli |

---


# 9. Model Performansı (Tuning Sonrası)


| Model                   | AUC  | FN | FP | Mean Accuracy (CV) | Test Accuracy | Train Accuracy | Fark (Train - Test) | Notlar                   |
| ----------------------- | ---- | -- | -- | ------------------ | ------------- | -------------- | ------------------- | ------------------------ |
| **Logistic Regression** | 0.91 | 13 | 19 | 0.819              | 0.83          | 0.83           | 0.00                | Dengeli, overfitting yok |
| **Random Forest**       | 0.91 | 13 | 16 | 0.818              | 0.84          | 1.00           | 0.16                | Aşırı öğrenme riski var  |
| **KNN**                 | 0.88 | 9  | 21 | 0.804              | 0.84          | 0.85           | 0.01                | En düşük FN, istikrarlı  |
| **SVM**                 | 0.90 | 10 | 20 | 0.818              | 0.84          | 0.86           | 0.02                | Dengeli, düşük fark      |
| **XGBoost**             | 0.90 | 17 | 16 | 0.814              | 0.82          | 1.00           | 0.18                | Aşırı öğrenme gözlenmiş  |
| **Gradient Boosting**   | 0.91 | 12 | 16 | 0.803              | 0.85          | 0.92           | 0.07                | Orta düzey overfitting   |
| **CatBoost**            | 0.91 | 12 | 18 | 0.807              | 0.84          | 0.96           | 0.12                | Overfitting riski var    |

---

# 10. Klinik Değerlendirme

Bu projede, kalp hastalığı riskinin tahmini amacıyla farklı makine öğrenmesi sınıflandırma algoritmaları karşılaştırılmıştır. Veri ön işleme adımlarından başlayarak hiperparametre tuning süreci dahil tüm modelleme adımları detaylıca uygulanmıştır.

Elde edilen sonuçlara göre:

* **En iyi genel performans** Gradient Boosting modeline ait olup, test doğruluğu en yüksektir (%85).
* **En yüksek güvenilirlik ve istikrar**, overfitting riski taşımayan ve tüm metriklerde dengeli sonuçlar veren **Lojistik Regresyon** ve **SVM** modellerindedir.
* **Hasta kaçırma oranı** açısından en iyi model **KNN** olup en düşük False Negative sayısına sahiptir.
* **XGBoost ve Random Forest** modelleri eğitim verisine aşırı uyum sağlamış ve test doğruluğu ile arada ciddi farklar gözlenmiştir.

Bu değerlendirmeler ışığında, sağlık gibi hassas bir alanda kullanılacak modellerde hasta kaçırmama önceliklidir. Bu nedenle:

* **KNN** modeli hasta tespiti açısından avantajlıdır.
* **SVM ve Lojistik Regresyon**, hem yorumlanabilirlik hem de genel güvenilirlik açısından tercih edilebilir modellerdir.

Veri dengeli olduğu için metrikler güvenilirdir, ancak gerçek hayatta dengesiz veri ile karşılaşılabileceği unutulmamalıdır. Bu durumda ROC-AUC ve Recall gibi metrikler daha öncelikli değerlendirme kriterleri olmalıdır.


Kalp hastalıkları, dünyada en yaygın ölüm nedenleri arasında yer almaktadır. Bu nedenle, **erken teşhis ve risk belirleme** çalışmaları klinik açıdan hayati öneme sahiptir. Bu projede geliştirilen modeller, bir bireyin kalp hastalığı riski taşıyıp taşımadığını veriye dayalı olarak tahmin etme kapasitesine sahiptir.

* **False Negative (FN)** sayısı özellikle kritik bir metriktir çünkü hasta bireylerin sağlıklı olarak sınıflandırılması, potansiyel bir tedavi fırsatının kaçırılması anlamına gelir.
* Bu bağlamda, **KNN modeli**, en düşük FN değeriyle hasta kaçırma riskini azaltması açısından öne çıkmaktadır.
* **Lojistik Regresyon** ise medikal alanda yaygın kullanımı ve açıklanabilir yapısıyla doktorların karar süreçlerine destek olabilecek güvenilir bir modeldir.

# 🎯 Yapay Zeka ve Teknoloji Akademisi 2026 Datathon

*YZTA 2026 Datathon'u için geliştirilmiş yüksek performanslı, RMSE odaklı CatBoost tahminsel modelleme (predictive modeling) projesi.*

## 💡 Vizyon ve Çözüm Mimarisi

Bu projenin temel hedefi, bireylerin uyku ve yaşam tarzı verilerini kullanarak hedef sağlık/uyku skorunu en düşük **RMSE** hatasıyla tahmin etmektir. Veri setindeki gürültüyü ve aşırı öğrenme (overfitting) riskini izole etmek için tek bir modele bağlı kalmak yerine, **Seed Blending (Tohum Harmanlama)** stratejisi kullanılarak istikrarlı ve güçlü bir çözüm mimarisi inşa edilmiştir.

## 🔧 Veri Ön İşleme (Preprocessing)

Model performansını doğrudan etkileyen veri hazırlığı aşamasında şu adımlar uygulandı:

* **Zero-Inflation Yönetimi:** `uyku_oncesi_kafein_mg` ve `sekerleme_suresi_dk` özelliklerindeki sıfır yığılmaları fark edilerek, modele yeni bir perspektif kazandırmak adına `kafein_tuketimi_yok` ve `sekerleme_yok` şeklinde ikili (binary) niteliklere dönüştürüldü.
* **Kategorik Uzay Temizliği:** `ulke` sütununda yer alan "South Korea / Güney Kore" veya "Spain / Ispanya" gibi çoklu dil ve yazım farklılıkları normalize edilerek boyut karmaşası önlendi.
* **Hedef Sinyali Yakalama:** Korelasyon analizleri sonucunda stres skoru, REM/Derin uyku yüzdesi ve uyku bölünmesi gibi temel sinyaller tespit edilerek modelin doğru değişkenlere odaklanması sağlandı.
* **Sızıntı (Leakage) Kontrolü:** Tüm ön işleme adımları hedef değişkenden bağımsız olarak, doğrudan `X` ve `test_X` matrisleri üzerinde tasarlanarak veri sızıntısının önüne geçildi.

## 🚀 Model Optimizasyonu ve Eğitim Süreci

* **Algoritma Seçimi:** Eksik verileri (NaN) doğal yollarla işleyebilme kapasitesi ve kategorik değişkenlerdeki üstün başarısı sebebiyle **CatBoostRegressor** mimarisi kullanıldı.
* **Çapraz Doğrulama (K-Fold CV):** Modelin gerçek dünya verisine genellenebilirliğini sağlamak için KFold validasyon şeması kuruldu.
* **Ağırlıklı Harmanlama (Weighted Blending):** Model 3 farklı rastgele seed (tohum) değeriyle baştan eğitildi. Modellerin Out-of-Fold (OOF) tahminleri üzerinden 0.01 hassasiyetli bir arama (Grid Search) yapılarak **en düşük RMSE'yi veren optimum ağırlık katsayıları** bulundu. Nihai tahminler matematiksel olarak sınırlandırılarak `[0, 10]` aralığına (`np.clip`) hapsedildi.

## 📂 Çıktılar ve Raporlama

Notebook (`submisson3.ipynb`) çalıştırıldığında sonuçları şeffaf bir şekilde analiz edebilmek için aşağıdaki çıktıları üretir:

* `submission_best_notebook_oof_blend.csv` ➔ **Ana Teslim Dosyası:** Optimize edilmiş OOF ağırlıklarıyla oluşturulan nihai harmanlanmış (blend) tahminler.
* `submission_best_single_seed_3407.csv` ➔ (Yedek Teslim) En iyi performansı gösteren tekil modele ait sonuçlar.
* `score_report_best_notebook.csv` ➔ Modellerin ve nihai blend işleminin OOF RMSE performans dökümü.

## 💻 Nasıl Çalıştırılır?

Proje, Kaggle platformu ve yerel Jupyter ortamlarına tam uyumlu olarak kodlanmıştır:

1. Repoyu ortamınıza klonlayın.
2. `train.csv` ve `test_x.csv` dosyalarının notebook ile aynı dizinde olduğundan emin olun.
3. `submisson3.ipynb` dosyasını baştan sona çalıştırın. Script; tüm feature engineering, model eğitimi, çapraz doğrulama ve en iyi ağırlıkları hesaplama süreçlerini otomatik olarak tamamlayıp *submission* dosyalarını dizine kaydedecektir.

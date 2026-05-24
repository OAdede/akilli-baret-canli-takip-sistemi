# Model Geliştirme

Bu doküman veri sınıflarını, eğitim aşamalarını ve sonuçların nasıl yorumlanması gerektiğini özetler.

## Sınıflar

| Sınıf | Açıklama |
| --- | --- |
| `On Head` | Baret kafada |
| `On Belt` | Baret kemerde veya belde |
| `In Hand` | Baret elde |
| `On Surface` | Baret masa, zemin veya yüzey üzerinde |

Model kamera görüntüsü kullanmaz. Sınıflandırma, MPU6050 ivme ve jiroskop verisinden çıkarılan istatistiksel özelliklerle yapılır.

## Veri Yapısı

Sensör CSV dosyalarında temel kolonlar:

```text
time_ms, acc_x, acc_y, acc_z, gyro_x, gyro_y, gyro_z, temp_c
```

Eğitim ve test CSV dosyalarında ek olarak:

```text
label
```

kolonu bulunur. Demo veya canlı tahmin sırasında etiket kullanılmaz.

`data/raw/helmet_data.csv`, ilk gerçek kayıtların birleştirilmiş eğitim çıktısıdır. `data/raw/helmet_data_pilot.csv`, geliştirme sürecinde alınan pilot kayıt olarak tutulur. Tekil sınıf CSV dosyaları ham veri izlenebilirliği için ayrıca korunur.

## Özellikler

Model, 40 satırlık yaklaşık 2 saniyelik pencereler üzerinde çalışır. Her pencere için:

- `acc_x`, `acc_y`, `acc_z`
- `gyro_x`, `gyro_y`, `gyro_z`
- `acc_mag`
- `gyro_mag`

kanallarından mean, std, min, max, range ve RMS özellikleri çıkarılır.

## Aşamalar

| Aşama | Veri ve değerlendirme | Sonuç |
| --- | --- | ---: |
| V1 Baseline | İlk bağımsız test seti | %91,31 |
| V2 Orientation Test | Yeni bağımsız test, sol kemer yönelimi dahil | %70,55 |
| V3 Final Model | Tüm mevcut gerçek verilerle eğitim sonrası iç kontrol | %95,89 |

Önemli protokol notu: V1 aşamasındaki `data/test/` verileri, V1 bağımsız test sonucu alındıktan sonra sonraki model geliştirmesinde eğitim verisine dahil edilmiştir. V2 aşamasındaki `data/test2/` verileri de yönelim etkisini gösteren bağımsız testten sonra V3 eğitimine dahil edilmiştir. Bu nedenle V1 %91,31 ve V2 %70,55 değerleri kendi ölçüm anları için bağımsız test sonuçlarıdır. V3 %95,89 ise yalnızca iç kontrol sonucudur; V3 için yeni bağımsız final test yapılmamıştır.

## V1

V1 aşamasında bağımsız test doğruluğu %91,31 olarak ölçülmüştür. İlgili rapor ve confusion matrix:

- `results/v1_baseline/independent_test_report.txt`
- `results/v1_baseline/independent_test_confusion_matrix.png`

GitHub yayınında canlı sistem için yalnızca V3 final model binary dosyası tutulur. V1 çıktıları rapor ve grafik olarak korunur.

## V2

V2 aşamasında yeni bağımsız yönelim testi %70,55 doğruluk göstermiştir. Bu testte sol tarafta kemerde taşınan baretin büyük ölçüde `In Hand` olarak karıştırıldığı görülmüştür. Bu sonuç, yönelim farkının model performansını etkilediğini açıkça göstermiştir.

İlgili dosyalar:

- `results/v2_orientation_test/v2_test2_report.txt`
- `results/v2_orientation_test/v2_test2_confusion_matrix.png`

GitHub yayınında V2 model binary dosyası tutulmaz; V2 sonucu rapor ve confusion matrix üzerinden belgelenir.

## V3 Final Model

V3 Final Model, sağ ve sol kemer varyasyonları dahil tüm mevcut gerçek verilerle eğitilmiştir. İç kontrol doğruluğu %95,89'dur.

Bu değer yeni bağımsız final test sonucu değildir. Eğitim verisine yakın pencere yapısı ve tüm mevcut gerçek verilerin kullanılmış olması nedeniyle yalnızca iç kontrol olarak yorumlanmalıdır.

İlgili dosyalar:

- `models/helmet_rf_model_v3_final.joblib`
- `results/v3_final_model/v3_final_internal_report.txt`
- `results/v3_final_model/v3_final_internal_confusion_matrix.png`

## Etiketsiz Demo

Etiketsiz tek-durum masa demosunda model `On Surface` tahmini vermiştir. Bu demo, canlı veya offline tahmin akışının çalıştığını göstermek içindir; bağımsız test sonucu olarak sunulmaz.

## Yardımcı Script Notu

`ml/fix_test2_labels.py`, geçmişte `test2` veri etiketlerini düzeltmek için kullanılan yardımcı script olarak repoda bırakılmıştır. Ana canlı akışta çalışmaz; veri hazırlama geçmişinin izlenebilir olması için tutulur.

## Dürüst Yorum

Bu prototip model geliştirme açısından umut vericidir; ancak gerçek saha koşulları için daha geniş veri gerekir. Farklı kullanıcılar, baret takma açıları, yürüme biçimleri, titreşimli ortamlar ve montaj yerleri modele eklenmeden sistem sertifikalı güvenlik kararı olarak görülmemelidir.

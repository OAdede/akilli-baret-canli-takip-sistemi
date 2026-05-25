# Donanım Kurulumu

Bu doküman, final fiziksel prototipte kullanılan donanımı, sensör bağlantılarını, taşınabilir güç hattını ve güvenlik notlarını açıklar.

## Final Donanım Listesi

- ESP32 Dev Module
- MPU6050 ivmeölçer + jiroskop modülü
- 3.7 V 500 mAh Li-Po pil
- TP4056 şarj/koruma modülü
- XL6009 boost dönüştürücü
- Açma-kapama anahtarı
- Prototipleme bağlantıları ve montaj elemanları

Veri toplama/model geliştirme aşamasında ayrıca microSD modülü kullanılmıştır. microSD modülü, sensör kayıtlarını CSV olarak almak için kullanılan geliştirme bileşenidir; final canlı takip sürümünde zorunlu değildir.

## ESP32 - MPU6050 Bağlantısı

| ESP32 | MPU6050 |
| --- | --- |
| `3V3` | `VCC` |
| `GND` | `GND` |
| `GPIO21 / D21` | `SDA` |
| `GPIO22 / D22` | `SCL` |

Arduino kodunda I2C pinleri şu şekilde başlatılır:

```cpp
Wire.begin(21, 22);
```

## Sensör Ayarları

Eğitim verisiyle uyumlu canlı ayarlar:

| Sensör ayarı | Değer |
| --- | --- |
| İvme aralığı | +/-8G |
| Jiroskop aralığı | +/-500 dps |
| Örnekleme hedefi | Yaklaşık 20 Hz |
| Paket boyutu | 20 satır |

Bu ayarlar `arduino/akilli_baret_canli_wifi/akilli_baret_canli_wifi.ino` içinde korunur.

## Final Güç Bağlantısı

Taşınabilir prototipte Li-Po tabanlı güç hattı TP4056 üzerinden korunur, anahtar ile kontrol edilir ve XL6009 boost dönüştürücü ile ESP32 için 5.00 V seviyesine yükseltilir.

```text
Li-Po kırmızı (+)  → TP4056 B+
Li-Po siyah (-)    → TP4056 B-

TP4056 OUT+        → Anahtar orta bacak
Anahtar uç bacak   → XL6009 IN+
TP4056 OUT-        → XL6009 IN-

XL6009 OUT+ 5.00 V → ESP32 VIN / 5V
XL6009 OUT-        → ESP32 GND
```

XL6009 çıkışı multimetre ile 5.00 V seviyesine ayarlanmıştır. Pil üzerinden canlı sistem doğrulamasının ayrıca kayıt altına alınması önerilir.

## Güvenlik Notları

- ESP32'ye bağlamadan önce XL6009 çıkışının multimetre ile 5.00 V'a ayarlanması gerekir.
- XL6009 çıkışı `ESP32 3V3` pinine bağlanmaz; yalnızca `VIN / 5V` pinine bağlanır.
- Haricî güç hattı açıkken ESP32 aynı anda USB'den beslenmez.
- Şarj sırasında sistem kapalı tutulmalıdır.
- Açıkta kalan iletken temas noktaları taşınabilir kullanım öncesinde yalıtılmalıdır.
- Prototip, sertifikalı iş güvenliği ekipmanı olarak değerlendirilmemelidir.

## Wi-Fi Ayarı

Gerçek Wi-Fi bilgileri `.ino` dosyasında tutulmaz. Kurulum için:

```powershell
Copy-Item arduino\akilli_baret_canli_wifi\secrets.example.h arduino\akilli_baret_canli_wifi\secrets.h
```

`secrets.h` içinde kendi değerlerinizi yazın:

```cpp
#define WIFI_SSID "WIFI_ADINIZ"
#define WIFI_PASSWORD "WIFI_SIFRENIZ"
#define SERVER_ENDPOINT "http://BILGISAYAR_IP_ADRESI:8000/api/samples/batch"
```

`secrets.h` dosyası `.gitignore` ile GitHub dışı bırakılmıştır.

## Geliştirme Aşaması ve Final Sürüm Farkı

- İlk veri toplama aşamasında microSD ile CSV veri kaydı yapıldı.
- Final canlı sürümde microSD kaldırıldı.
- Final sürümde ESP32, MPU6050 verilerini Wi-Fi ile FastAPI sunucusuna aktarır.
- Taşınabilir kullanım için Li-Po tabanlı güç sistemi prototipe entegre edilmiştir.

Canlı kullanımda veri akışı:

```text
MPU6050 -> ESP32 -> Wi-Fi -> FastAPI -> Model -> Streamlit panel
```

şeklindedir; SD karta ihtiyaç yoktur.

## Pratik Kontrol Listesi

- ESP32 ve bilgisayar aynı yerel ağda olmalı.
- ESP32 çoğunlukla 2.4 GHz Wi-Fi ister.
- Bilgisayar güvenlik duvarı `8000` portuna gelen bağlantıyı engellememeli.
- `SERVER_ENDPOINT`, bilgisayarın aktif Wi-Fi IP adresini göstermeli.
- MPU6050 VCC pini ESP32 `3V3` ile beslenmeli.
- XL6009 çıkışı ESP32'ye bağlanmadan önce 5.00 V olarak ayarlanmalı.

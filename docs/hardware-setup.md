# Donanım Kurulumu

Bu doküman, prototipte kullanılan donanımı, kablolamayı ve güç seçeneklerini açıklar.

## Malzeme Listesi

- ESP32 Dev Module
- MPU6050 ivmeölçer + jiroskop modülü
- Jumper kablo
- USB kablo veya powerbank
- Geliştirme ve veri toplama aşaması için microSD modülü

microSD modülü canlı kullanım için zorunlu değildir. Canlı akışta ESP32, sensör verisini doğrudan Wi-Fi üzerinden FastAPI sunucusuna gönderir.

## MPU6050 Kablolama

| MPU6050 | ESP32 Dev Module |
| --- | --- |
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO21 |
| SCL | GPIO22 |

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

## microSD Modülünün Rolü

microSD modülü geliştirme sırasında veri toplamak için kullanılmıştır. Eğitim ve test CSV dosyaları bu aşamada oluşmuştur. Canlı kullanımda veri akışı:

```text
MPU6050 -> ESP32 -> Wi-Fi -> FastAPI -> Model -> Streamlit panel
```

şeklindedir; SD karta ihtiyaç yoktur.

## Güç Sistemi

Doğrulanmış çalışma:

- USB kablo
- Powerbank

Önerilen kompakt güç mimarisi:

```text
3.7 V Li-Po pil -> TP4056 şarj/koruma kartı -> 5 V boost dönüştürücü -> ESP32
```

Bu kompakt güç mimarisi, bu repo kapsamında fiziksel olarak doğrulanmış gibi sunulmamalıdır. Uygulama öncesinde akım kapasitesi, voltaj kararlılığı, ısınma ve şarj güvenliği ayrıca test edilmelidir.

## Pratik Kontrol Listesi

- ESP32 ve bilgisayar aynı yerel ağda olmalı.
- ESP32 çoğunlukla 2.4 GHz Wi-Fi ister.
- Bilgisayar güvenlik duvarı `8000` portuna gelen bağlantıyı engellememeli.
- `SERVER_ENDPOINT`, bilgisayarın aktif Wi-Fi IP adresini göstermeli.
- MPU6050 VCC pini 3V3 ile beslenmeli.

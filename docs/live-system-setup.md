# Canlı Sistem Kurulumu

Bu doküman Windows ve PowerShell üzerinde canlı sistemi çalıştırmak için pratik adımları verir.

## 1. Python Ortamı

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

PowerShell aktivasyon engeli olursa yalnızca mevcut terminal için:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.\.venv\Scripts\Activate.ps1
```

## 2. Bilgisayar IP Adresi

ESP32 ile bilgisayar aynı Wi-Fi ağında olmalıdır. Bilgisayarın Wi-Fi IP adresini bulmak için:

```powershell
ipconfig
```

`Wireless LAN adapter Wi-Fi` bölümündeki IPv4 adresini kullanın. Örnek:

```text
192.168.1.34
```

## 3. Arduino Secrets Dosyası

```powershell
Copy-Item arduino\akilli_baret_canli_wifi\secrets.example.h arduino\akilli_baret_canli_wifi\secrets.h
```

`secrets.h` dosyasını düzenleyin:

```cpp
#define WIFI_SSID "WIFI_ADINIZ"
#define WIFI_PASSWORD "WIFI_SIFRENIZ"
#define SERVER_ENDPOINT "http://192.168.1.34:8000/api/samples/batch"
```

Gerçek IP adresini kendi bilgisayarınıza göre değiştirin. `secrets.h` GitHub'a gönderilmez.

## 4. ESP32 Kodunu Yükleme

Arduino IDE içinde klasörü açın:

```text
arduino/akilli_baret_canli_wifi/
```

Kart olarak ESP32 Dev Module seçin. Gerekli ESP32 kart desteği ve kütüphaneler kurulu olmalıdır. Bu repo Arduino CLI derleme doğrulaması içermez; derleme ortamı yerel Arduino kurulumuna bağlıdır.

## 5. FastAPI Sunucusu

Proje kökünde:

```powershell
uvicorn live_server:app --host 0.0.0.0 --port 8000
```

Sağlık kontrolü:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
```

ESP32'nin erişebilmesi için Windows güvenlik duvarı `8000` portunu engellememelidir.

## 6. Canlı Dashboard

Yeni bir PowerShell penceresinde:

```powershell
.\.venv\Scripts\Activate.ps1
streamlit run live_dashboard.py --server.port 8502
```

Tarayıcıda genellikle şu adres açılır:

```text
http://localhost:8502
```

## 7. Donanım Olmadan Test

Sunucu çalışırken:

```powershell
python simulate_live_csv.py --delay 1
```

Hızlı test için:

```powershell
python simulate_live_csv.py --delay 0.01
```

Simülasyon, `data/demo_input/demo_unlabeled_scenario_01.csv` dosyasını 20 satırlık paketler halinde gönderir.

## 8. Sorun Giderme

| Belirti | Olası neden | Kontrol |
| --- | --- | --- |
| ESP32 Wi-Fi bağlanmıyor | Yanlış SSID/şifre veya 5 GHz ağ | 2.4 GHz ağ ve `secrets.h` değerlerini kontrol edin |
| HTTP gönderim hatası | IP veya port hatalı | `SERVER_ENDPOINT` IP adresini ve `uvicorn` komutunu kontrol edin |
| Dashboard veri görmüyor | Sunucu çalışmıyor veya farklı baret ID | `/health` ve `/api/status/Baret-01` endpointlerini kontrol edin |
| İlk tahmin gelmiyor | 40 satır pencere dolmadı | En az iki 20 satırlık paket gerekir |
| Arduino derlenmiyor | `secrets.h` yok | `secrets.example.h` dosyasını `secrets.h` olarak kopyalayın |
| Port erişilemiyor | Windows güvenlik duvarı | Yerel ağdan Python/8000 portuna izin verin |

## 9. Canlı Akış Özeti

- ESP32 20 Hz sensör okur.
- 20 satırlık paketi `/api/samples/batch` endpointine yollar.
- Sunucu 40 satırlık pencereyle V3 Final Model tahmini üretir.
- Dashboard `/api/status/Baret-01` üzerinden güncel durum ve geçmişi çeker.

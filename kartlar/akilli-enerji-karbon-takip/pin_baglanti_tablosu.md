# Pin Bağlantı Tablosu — Akıllı Enerji Yönetimi ve Karbon Takip Sistemi

Faz 1 çıktısı. Her satır: **pin → nereye bağlanacağı → neden (datasheet referanslı)**.
Onay bekleyen madde: son bölümdeki "Kullanıcı Onayı Gereken Ek Öneriler".

---

## 0. BOM / Tedarik Notu (Faz 1 adım 2)

| Bileşen | Kullanıcının belirttiği | Önerilen / kullanılacak | Neden |
|---|---|---|---|
| MCU modülü | ESP32-WROOM-32 | **ESP32-WROOM-32E** (veya 32UE) | Espressif'in resmi datasheet'i ESP32-WROOM-32 (32D/32U olmayan taban varyant) için **NRND (Not Recommended for New Designs)** ibaresi taşıyor. 32E/32UE pin-uyumlu, aktif üretimde ve tedarik riski düşük. Kaynak: `esp32-wroom-32_datasheet_en.pdf` başlık/revizyon notu. |
| Akım sensörü | ACS712 (5A veya 20A) | **ACS712 5A hazır breakout modül kartı** (VCC/OUT/GND pin header'lı, IP+/IP− tarafı vida terminalli, üzerinde ACS712ELCTR-05B-T + CBYP + CF entegre) | Faz 0'da kullanıcı 5A'yı onayladı (185 mV/A, Allegro datasheet Rev.22). **Bare SOIC-8 yerine hazır modül seçildi:** Röle modülü zaten aynı mantıkla (hazır kart) kullanılıyor (§3); SOIC-8'i doğrudan elle lehimlemek yerine yaygın, ucuz, hazır bypass/filter kapasitörlü modül kartı kullanmak elle-lehim pratiğine çok daha uygun ve daha güvenli (izolasyon mesafeleri modül PCB'sinde zaten üretici tarafından sağlanmış). |
| Röle | SRD-05VDC-SL-C (opto izolasyonlu "röle modülü") | Hazır **5V röle modülü kartı** (SRD-05VDC-SL-C bobinli, üzerinde PC817 optokuplör + sürücü transistör + flyback diyot + durum LED'i barındıran, yaygın 3+3 pinli "Arduino tipi" modül) | Kullanıcı açıkça "built-in optocoupler isolation" diyor — bu, çıplak röleyi değil hazır modül kartını tanımlıyor. Modülün iç sürücü devresi (transistör/diyot tam MPN'i) üretici bazında değişir ve pin haritası olmadan iç şema çizmek uydurma pin demektir (bkz. şematik kural #12) → modül **kara kutu** olarak, yalnız dış pinleriyle (VCC, GND, IN | COM, NO, NC) çizilecek. |
| Güç kaynağı | HLK-PM01 | HLK-PM01 (değişmedi) | 100–240VAC girişli, 5V/600mA (tepe 1A) çıkışlı, evrensel şebeke gerilimiyle uyumlu — kullanıcının "220V" varsayımını da kapsıyor. |
| LDO | AMS1117-3.3 | **AMS1117-3.3 hazır breakout modül kartı** (IN/OUT/GND pin header'lı, üzerinde giriş+çıkış kapasitörleri entegre) | Aynı gerekçe: bare SOT-223'ü doğrudan elle lehimlemek yerine, üzerinde CIN/COUT zaten bulunan yaygın hazır modül kartı kullanılıyor — hand-solder pratiğine uygun, 1A çıkış kapasitesi ESP32'nin ~500mA tepe akımını rahat karşılıyor. |

---

## 1. HLK-PM01 (220V AC → 5V DC izoleli güç kaynağı)

| Pin | Fonksiyon | Bağlantı | Neden |
|---|---|---|---|
| 1 | AC (L girişi) | AC Giriş Terminali → Röle COM (bkz. §4, yük yolu Röle üzerinden geçiyor — **HLK-PM01'in kendi beslemesi doğrudan AC Giriş L/N'den alınır, röle üzerinden DEĞİL**, aksi halde röle açıkken ESP32/HLK-PM01 de güçsüz kalır ve kendi kendini kesen bir sistem asla geri açılamaz) | HLK-PM01 datasheet: Pin1=AC canlı hat. Kendi beslemesi kesilirse MCU çalışamaz → röle kontrolünü kaybeder. Bu yüzden HLK-PM01 girişi **AC Giriş terminaline doğrudan**, kesilen yükün L hattından **bağımsız** bağlanmalı. |
| 2 | AC (N girişi) | AC Giriş Terminali (N) | HLK-PM01 datasheet Pin2. |
| 3 | −Vo (DC GND) | Sistem ortak GND (düşük gerilim tarafı) | HLK-PM01 çıkışı zaten AC tarafından galvanik izoleli (modül içi trafo/izolasyon); −Vo düşük-gerilim referans olarak kullanılabilir. |
| 4 | +Vo (5V DC çıkış) | 5V dağıtım rayı → Röle modülü VCC, ACS712 VCC, AMS1117 VIN | Çıkış 600mA sürekli / 1A tepe — röle bobini (~71mA) + ACS712 (~10mA) + AMS1117 girişi (ESP32 ~500mA tepe / 3.3V tarafında, 5V tarafında karşılığı daha düşük ama LDO kaybıyla birlikte) toplamı bütçe içinde. |

**Kritik mimari düzeltme:** Kullanıcının orijinal netlist taslağında "AC Giriş (L) → Röle COM" var; bu doğru — **kesilen yol yalnızca yük tarafı (AC Çıkış'a giden L)**. HLK-PM01'in kendi AC beslemesi ayrıca ve doğrudan AC Giriş L/N'den alınır (paralel), röle zincirinin İÇİNDE değildir. Aksi taktirde röle NO açık kalırsa (örn. arıza/kapanma) sistem kendini besleyemez ve bir daha asla açılamaz — kilitlenme (deadlock) riski.

---

## 2. ACS712 5A Breakout Modül (ACS712ELCTR-05B-T tabanlı hazır kart)

Modül dış arayüzü (kartın kendi iç CBYP/CF'i üreticide zaten var — ayrıca eklenmeyecek):

| Pin | Fonksiyon | Bağlantı | Neden |
|---|---|---|---|
| VCC | Modül beslemesi | 5V rayı (HLK-PM01 +Vo) | Datasheet: 5.0V tek beslemeli çalışma. |
| GND | Ortak GND | Sistem ortak GND (düşük gerilim) | — |
| OUT | Analog çıkış (VIOUT) | R3 (1kΩ seri) üzerinden ESP32 GPIO_36 (SENSOR_VP / ADC1_CH0) + D1 (Schottky) ile +3V3'e clamp | Kullanıcının belirttiği ADC pini — ADC1 kanalı olduğu için Wi-Fi aktifken de kullanılabilir (ADC2, Wi-Fi ile çakışır; ADC1 çakışmaz). **Faz 3.2 bulgusu (2026-09-25):** ACS712 5V beslemeli, çıkışı VCC/2±duyarlılık (~0.5–4.5V tam ölçek); ESP32 GPIO/ADC mutlak maksimum giriş gerilimi ise VDD3P3+0.3V (~3.6V). Normal bekleme-modu ölçümünde risk yok ama arıza/aşırı yük anında (F1 sigortasının izin verdiği tepe akıma kadar) çıkış 3.6V'a yaklaşabilirdi. Kullanıcı onayıyla R3 (seri akım sınırlama) + D1 (anot→ACS_OUT_PROT, katot→+3V3, aşırı gerilimi klempler) eklendi — bağımsız `kicad-cli` netlist'iyle doğrulandı. |
| IP+ (vida terminal) | Akım yolu girişi | Röle NO çıkışı | Allegro ACS712 datasheet: akım pinleri IP+ → IP− yönünde, pozitif eğim. |
| IP− (vida terminal) | Akım yolu çıkışı | AC Çıkış Terminali (L) | Akım yolu: Röle NO → IP+/IP− → AC Çıkış L. |

**Not — izolasyon mesafesi:** Allegro datasheet, çıplak IC üzerinde pin 1-4 (IP grubu) / pin 5-8 (sinyal grubu) arası clearance/creepage = 4mm belirtiyor. Modül kartı bu mesafeyi üretici PCB'sinde zaten sağlıyor; bizim karta düşen kısım yalnızca modülün VCC/GND/OUT (düşük gerilim) tarafı ile IP+/IP− (yüksek gerilim) vida-terminal tarafı arasında ana kart üzerinde de **ayrıca 4mm+ boşluk** bırakmaktır — genel 2.5-3mm kuralımızdan daha sıkı, bu daha sıkı değer geçerli olacak.

---

## 3. Röle Modülü (SRD-05VDC-SL-C tabanlı, opto-izoleli hazır modül)

| Pin | Fonksiyon | Bağlantı | Neden |
|---|---|---|---|
| VCC | Modül lojik/bobin beslemesi | 5V rayı | Modülün onboard transistör sürücüsü ve röle bobini 5V ile besleniyor. |
| GND | Ortak GND | Sistem ortak GND | — |
| IN | Kontrol girişi (genelde LOW-tetikleme, optokuplör üzerinden) | ESP32 **GPIO4** | Kullanıcı "GPIO_12 (veya boş bir GPIO)" önerdi; **GPIO12, ESP32'de MTDI strapping pinidir** — reset anında VDD_SDIO (flash gerilimi: 3.3V/1.8V) seçimini belirler. Reset sırasında dışarıdan (röle modülünün optokuplör girişi/pull-up'ı üzerinden) yüksek çekilirse flash gerilimi yanlış seçilip **boot arızasına** yol açabilir (Espressif datasheet, strapping pin uyarısı). Bu riski önlemek için **GPIO4** kullanılacak — strapping/SPI-flash/UART0 işlevi olmayan, serbest bir GPIO. |
| COM | Ortak kontak | AC Giriş Terminali (L) | Kullanıcı netlist'i: AC Giriş L → Röle COM. |
| NO | Normalde-açık kontak | ACS712 IP+ (pin 1-2) | Röle aktifken (yük açık) L hattı buradan akar. |
| NC | Normalde-kapalı kontak | **Kullanılmıyor (NC — no connect)** | Kart normal-açık (aktif rölede yük geçsin) mantığıyla çalışıyor; NC ucu bağlanmaz, boşta bırakılır. |

**Modülün iç şeması çizilmeyecek** — yalnızca dış pin arayüzü kullanılacak (yukarıdaki BOM notundaki gerekçe).

---

## 4. AMS1117-3.3 Breakout Modül (5V → 3.3V, IN/OUT/GND hazır kart)

| Pin | Fonksiyon | Bağlantı | Neden |
|---|---|---|---|
| IN | Giriş | 5V rayı (HLK-PM01 +Vo) | — |
| GND | Ortak | Sistem ortak GND | — |
| OUT | Çıkış | 3.3V rayı → ESP32 3V3 pini | Modülün kendi CIN/COUT'u üreticide zaten var (datasheet min. 10µF çıkış önerisi modül PCB'sinde karşılanıyor); ana kartta ayrıca eklenmeyecek. |

---

## 5. ESP32-WROOM-32E — kullanılan pinler

| Fiziksel Pin | İsim | Bağlantı | Neden |
|---|---|---|---|
| 2 | 3V3 | AMS1117 VOUT (3.3V rayı) + **C2 (100nF, yerel decoupling)** | Şematik kural #2 (Faz 2) — **Faz 5 EMI incelemesinde eksik bulundu:** AMS1117 modülünün COUT'u header/tel üzerinden fiziksel uzak, WiFi TX akım darbelerinde yerel kapasitör olmadan VDD sarkması/reset riski. C2 eklendi — elle lehimde U1 pin2/GND'ye fiziksel olarak mümkün olduğunca yakın lehimlenmeli. |
| 1, 15, 38, 39 | GND | Sistem ortak GND | 4 ayrı GND pini, hepsi bağlanmalı (datasheet). |
| 3 | EN | 3.3V rayına 10kΩ pull-up + 1µF GND'ye (RC gecikme) + manuel reset butonu (buton → GND) | Datasheet Typical Application: "R=10kΩ, C=1µF önerilen RC gecikme devresi." Buton, elle reset için standart bring-up pratiği. |
| 4 | SENSOR_VP / GPIO36 / ADC1_CH0 | ACS712 VIOUT → R3(1kΩ)+D1(Schottky clamp→+3V3) üzerinden | Kullanıcının netlist'te belirttiği pin, ADC1 kanalı (Wi-Fi ile çakışmaz). Faz 3.2 gerilim-marjı bulgusu için R3/D1 korumalı — bkz. §2. |
| 14 | IO12 / GPIO12 (MTDI, strapping) | **NC — kullanılmıyor** | Strapping pin riski nedeniyle röle kontrolü GPIO4'e taşındı (bkz. §3); bu pin bilinçli olarak boş bırakılıyor, kazayla bir şeye bağlanmayacak. |
| 26 | IO4 / GPIO4 | Röle modülü IN | Strapping/SPI-flash/UART0 işlevi olmayan serbest GPIO — bkz. §3 gerekçesi. |
| 34 | RXD0 / GPIO3 (U0RXD) | Programlama header'ı (6 pin: 3V3, EN, GPIO0, TXD0, RXD0, GND) → harici USB-seri (FTDI/CP2102) modülüne | ESP32-WROOM-32 çıplak modül, kendi USB'si yok — **programlanabilmesi için** UART header zorunlu (Faz 1 kuralı: "Typical Application incelemesi", bare-module bring-up standart pratiği). Kullanıcının orijinal spesifikasyonunda bu header yoktu; **kart programlanamazsa çalışmaz**, bu nedenle ekleniyor. |
| 35 | TXD0 / GPIO1 (U0TXD) | Programlama header'ı | Aynı gerekçe. |
| 25 | IO0 / GPIO0 (strapping) | Programlama header'ına + 3.3V'a 10kΩ pull-up + manuel BOOT butonu (buton → GND) | Boot-mode strapping pini; varsayılan yüksek = SPI boot (normal çalışma). Buton, elle flash moduna almak için. |
| 17–22 | IO9, IO10, IO11, IO6, IO7, IO8 (SPI flash) | **Dokunulmaz, kullanılmaz** | Datasheet: "not recommended for other uses" — modül içi flash'a bağlı. |
| Kalan tüm GPIO'lar (IO34, IO35, IO39/SVN, IO32, IO33, IO25, IO27, IO13, IO15, IO2, IO16, IO17, IO5, IO18, IO19, IO21, IO22, IO23) | — | **NC** | Kullanılmayan pinler boşta bırakılıyor (kural: output/GPIO pinleri asla GND/VCC'ye bağlanmaz). |

---

## 6. Terminal Bloklar

| Terminal | Pin | Bağlantı |
|---|---|---|
| AC Giriş (2 pin, 5.08mm) | L | HLK-PM01 Pin1 (AC) **ve** Röle COM (iki ayrı dal, §1'deki kritik not) |
| AC Giriş (2 pin, 5.08mm) | N | HLK-PM01 Pin2 (AC) **ve** AC Çıkış N (doğrudan, anahtarlanmıyor) |
| AC Çıkış/Yük (2 pin, 5.08mm) | L | ACS712 IP− (pin 3-4) |
| AC Çıkış/Yük (2 pin, 5.08mm) | N | AC Giriş N (doğrudan) |

---

## Ek Öneriler — Kabul Edilip Uygulanıyor (kullanıcı "devam" onayı, 2026-09-24)

Bunlar kullanıcının orijinal spesifikasyonunda yoktu; datasheet/güvenlik gerekçesiyle eklendi ve Faz 2'de uygulanacak:

1. **AC Giriş L hattına seri fuse (3.15A hızlı atan, 250V değerli).**
2. **HLK-PM01 girişinin AC Giriş'ten doğrudan (röle zincirinin dışından) alınması** (§1 deadlock düzeltmesi).
3. **ESP32 programlama header'ı + EN/BOOT butonları** (§5).
4. **ACS712 modülü için ana kart üzerinde 4mm izolasyon mesafesi.**
5. **ACS712 ve AMS1117 bare IC yerine hazır breakout modül kartı olarak kullanılıyor** (§2, §4) — röle modülüyle tutarlı, elle-lehim pratiğine uygun.

Faz 2 (şematik çizimi) başlıyor.
